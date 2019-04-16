## Ansible中错误处理
Ansible会根据每个任务的返回值判断任务成功或者失败。通常，如果一个任务失败，Ansible会立即中止play中剩余任务的执行，然而，有时候我们希望即使某一个任务失败，play可以继续执行，执行一些工作例如采取恢复措施等。在Ansible中有许多用于处理错误和失败的特性，可以提供这种场景支持。常见的处理方式有：

- 忽略任务失败
    默认，一个任务失败，则该play会被中止。我们可以使用‘ignore_errors’关键字去重写失败任务的行为，即忽略任务失败。如下：
    ```
    - name: Latest version of notapkg is installed
      yum:
        name: notapkg
        state: latest
      ignore_errors: yes
    ```
    即是notapkg不能安装最新版本成功，play的其他任务也会继续执行。

- 失败后强制唤醒handlers
    默认，当一个任务失败，play立即中止，包括之前被其他task唤醒的handlers。我们可以通过‘force_handlers: yes’关键词强制唤醒，无论play是否中止，被唤醒的hanlders总会被执行。如下：
    ```
    ---
    - hosts: all
      force_handlers: yes
      tasks:
        - name: a task which always notifies its handler
          command: /bin/true
          notify: restart the database
    
        - name: a task which fails because the package doesn't exist
          yum:
            name: notapkg
            state: latest
    
      handlers:
        - name: restart the database
          service:
            name: mariadb
            state: restarted
    ```
    第一个任务真正执行即返回‘changed’，handler被唤醒。这时，即使第二个任务失败，则handler还是会被执行。
- 指定任务失败的条件
  我们可以使用‘failed_when’关键字去指定哪种情况下任务是失败的。例如，我们可以执行一个脚本并输出错误信息，根据信息定义任务失败的状态。如下：
  ```
  tasks:
  - name: Run user creation script
    shell: /usr/local/bin/create_users.sh
    register: command_result
    failed_when: "'Password missing' in command_result.stdout"
  ```
  我们根据返回结果判断如果'Password missing'则任务执行失败。我们可以将该任务写成以下两个任务。我们可以用fail模块去提供一个具体和清晰的错误信息。
    ```
    tasks:
      - name: Run user creation script
        shell: /usr/local/bin/create_users.sh
        register: command_result
        ignore_errors: yes
    
      - name: Report script failure
        fail:
          msg: "The password is missing in the output"
        when: "'Password missing' in command_result.stdout"
    ```
- 指定任务'changed'行为
  当一个任务在被管理机器上执行，它通报状态为changed并唤醒handlers。当一个任务没有真实执行，则会通报ok并不唤醒handlers。
  ‘changed_when’关键字可以控制当changed发生时的行为。如下：我们可以执行指定任务的返回值
  ```
  - name: get Kerberos credentials as "admin"
    shell: echo "{{ krb_admin_pass }}" | kinit -f admin
    changed_when: false
  ```
  该任务总是返回changed，通过我们指定changed_when为false，则我们可以让该任务总是返回ok或者failed。
  我们可以根据任务执行的输出，指定返回状态为changed。如下
  ```
  tasks:
  - shell:
      cmd: /usr/local/bin/upgrade-database
    register: command_result
    changed_when: "'Success' in command_result.stdout"
    notify:
      - restart_database

    handlers:
      - name: restart_database
         service:
           name: mariadb
           state: restarted
  ```
  当command_result含有Success时，我们返回changed，然后唤醒handlers。
- Ansible块和错误处理
  在playbook中，块是逻辑上的一组任务。一个任务块可以用一个when来条件控制多个任务的执行。如下：
  ```
  - name: block example
  hosts: all
  tasks:
    - block:
      - name: package needed by yum
        yum:
          name: yum-plugin-versionlock
          state: present
      - name: lock version of tzdata
        lineinfile:
          dest: /etc/yum/pluginconf.d/versionlock.list
          line: tzdata-2016j-1
          state: present
      when: ansible_distribution == "RedHat"
  ```
  如上，两个任务为一个任务块，当条件满足时，该任务块会执行。
  我们已经讲了单个任务的错误处理，则对于块的错误处理我们需要结合‘rescue’和‘always’关键字来实现目的。则block, rescue和always的定义如下：
  * blocks: 定义需要执行的任务组
  * rescue: 定义如果block中的任务出现失败，则执行rescue中的任务
  * always: 定义总会执行的任务，而无论block的任务执行错误或者失败。
  则实际应用的例子如下：
    ```
    tasks:
      - block:
        - name: upgrade the database
            shell:
              cmd: /usr/local/lib/upgrade-database
        rescue:
        - name: revert the database upgrade
          shell:
            cmd: /usr/local/lib/revert-database
        always:
        - name: always restart the database
            service:
              name: mariadb
              state: restarted
    ```
  该play的主要任务是升级数据库，当升级数据库失败，则恢复数据库，但无论结果，最后，总是重启数据库。
