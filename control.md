## 任务控制
### 用Loops遍历任务
有一些module仅接受一个参数，则执行相似的任务，不使用loop我们需要创建多个非常相似的任务。Loop语法控制则可以更好的组织和编排。
- 遍历列表
如下，我们需要的启动两个服务，则不用loop。我们需要这样写：
    ```
    - name: Postfix is running
      service:
        name: postfix
        state: started
    
    - name: Dovecot is running
      service:
        name: dovecot
        state: started
    ```
    使用loop后，我们则可以如下写
    ```
    ---
    - name: Postfix and Dovecot are running
      service:
        name: "{{ item }}"
        state: started
      loop:
       - postfix
       - dovecot
    ```
    或者使用变量，其需要遍历的值写到一个列表变量里。
    ```
    vars:
      mail_services:
        - postfix
        - dovecot
    
    tasks:
      - name: Postfix and Dovecot are running
        service:
          name: "{{ item }}"
          state: started
        loop: "{{ mail_services }}"
    ```
    注意：item默认为loop的元素引用，这里不可以随意起名。
- 遍历字典或者哈希
    除了遍历列表，我们还以可以遍历字典或者哈希，如下
    ```
    - name: Users exist and are in the correct groups
      user:
        name: "{{ item.name }}"
        state: present
        groups: "{{ item.groups }}"
      loop:
        - name: jane
          groups: wheel
        - name: joe
          groups: root
    ```
- Loop中的注册变量
    ```
    ---
    - name: Loop Register Test
      gather_facts: no
      hosts: localhost
      tasks:
        - name: Looping Echo Task
          shell: "echo This is my item: {{ item }}"
          loop:
            - one
            - two
          register: echo_results
    
        - name: Show echo_results variable
          debug:
            var: echo_results
    ```
    则我们看到echo_results记录了循环的所有输出。
### 基于条件控制执行任务
- 条件控制的应用场景：
    Ansible提供了为是否执行任务及如何执行任务提供了条件控制。常见的条件场景有：
    - 硬控制可以定义为变量，与被管理机上可用的内存进行比较
    - 命令的输出可以被Ansible捕捉和分析去决定是否执行下一步任务。例如，A程序失败了，则B程序不再执行
    - 基于Ansible Facts确定被管理机上的网络配置，决定上传那个网络模版
    - 基于CPUs的数目决定怎样配化配置一个网络服务器
    - 比较实时获取的facts和我们事先定义好的变量去判断服务是否发生变化。例如，比较一个服务配置文件的MD5的checksum来判断一个服务是否发生变化
- 条件控制语法和示例
    ```
    tasks:
      - name: example_task
        ...
        when: condition
    ```
    当条件满足时，任务执行，否则不执行。条件可以是一个布尔值，也可以是一个判断语句。
    如下：
    ```
    ---
    - name: Simple Boolean Task Demo
      hosts: all
      vars:
        run_my_task: true
    
      tasks:
        - name: httpd package is installed
          yum:
            name: httpd
          when: run_my_task
        - name: "{{ my_service }} package is installed"
          yum:
            name: "{{ my_service }}"
          when: my_service is defined
- 条件控制运算符
    大于，小于，等于等都可作为条件来进行控制。常用的运算符有：
    | 运算符 | 示例 |
    | ---    | ---  |
    | == string | ansible_machine == "x86_64" |
    |== numberic | max_memory = 512 |
    | < numberic | max_memory < 512 |
    | > numberic | max_memory > 512 |
    | >= numberic | max_memory <= 512 |
    | <= numberic | max_memory >= 512 |
    | != numberic | min_memory != 512 |
    | variable exists | min_memory is defined |
    | variable does not exist | min_memory is not defined |
    | boolean variable | memory_available(1/true/True/yes/0/false/False/no) |
    | nnot boolean variable | not memory_available |
    | variable in list | ansible_distribution in supported_distros |
    则我们用表格最后一个条件控制来举例：
    ```
    ---
    - name: Demonstrate the "in" keyword
      hosts: all
      vars:
        supported_distros:
          - RedHat
          - Fedora
      tasks:
        - name: Install httpd using yum, where supported
          yum:
            name: http
            state: present
          when: ansible_distribution in supported_distros
    ```
    当被管理机的操作系统为centos或者fedora时，则我们使用yum去安装所需包。
- 多重条件判断
  有时候情况比较复杂，我们需要使用多重条件去进行条件判断。例如：
  ```
  when: ansible_distribution == "RedHat" or ansible_distribution == "Fedora"
  ```
  或者以如下格式进行展示
  ```
  when:
  - ansible_distribution_version == "7.5"
  - ansible_kernel == "3.10.0-327.el7.x86_64"
  ```
  或者
  ```
  when: >
    ( ansible_distribution == "RedHat" and
      ansible_distribution_major_version == "7" )
    or
    ( ansible_distribution == "Fedora" and
    ansible_distribution_major_version == "28" )
  ```
### 条件和语句控制执行任务
在接下来的例子中，我们会判断如果还有某个文件系统还有300M的空间，我们则会安装mariadb-server。 为了达到这个目的，则我们需要遍历所有挂载的文件系统然后判断其是否满足条件，满足条件则执行操作，否则不执行。
则该play如下：
```
- name: install mariadb-server if enough space on root
  yum:
    name: mariadb-server
    state: latest
  loop: "{{ ansible_mounts }}"
  when: item.mount == "/" and item.size_available > 300000000
```
ansible_mounts是Ansible facts中的一项
我们再举一个例子，如果postfix执行，则重启httpd服务，否则不重启。
```
---
- name: Restart HTTPD if Postfix is Running
  hosts: all
  tasks:
    - name: Get Postfix server status
      command: /usr/bin/systemctl is-active postfix
      ignore_errors: yes
      register: result

    - name: Restart Apache HTTPD based on Postfix status
      service:
        name: httpd
        state: restarted
      when: result.rc == 0
```
