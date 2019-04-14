## 管理变量
### Ansible变量简介
Ansible可使用变量，用于可在整个Ansible项目中重复使用的一些值。这样可以用来简化项目的创建和维护成本。
变量提供了一种便捷的方式去管理Ansible项目中动态值。我们常用变量来表示的值有：
* 需要创建的用户
* 需要安装的包
* 需要重启的服务
* 需要删除的文件
* 收集网上下载的文件

命名变量必须以字母开头，可包含或者仅可包含字母，数字和下划线。
### 定义变量
变量可被定义在项目中许多位置，但变量的作用域值有三种：
* 全局变量： 执行时从命令行指定的变量或者Ansible配置文件中的变量
* Play变量： 在Play中定义的变量
* Host变量：在Inventory中定义的变量
如果在不同作用域出现重复命名的变量，但作用域会进行叠加选更小的作用域。例如在Play变量会覆盖Host变量，而命令行指定的变量会覆盖所有作用域的变量。
### 在Playbook中定义和使用变量
当我们编写Playbook时，我们可以定义变量然后在tasks中进行调用。编写和调用的方式有两种：
* 在Playbook中开头定义变量块，后续使用，如下：
    ```
    - hosts: all
      vars:
        user: joe
        home: /home/joe
    ```
* 将变量定义在一个外部文件中，然后在Playbooks声明使用变量文件，如下：
    ```
    - hosts: all
      vars_files:
        - vars/users.yml
    ```
    变量文件也为yaml格式，使用如下：
    ```
    user: joe
    home: /home/joe
    ```
变量定义完成后， 我们就可以在Playbook中引用变量，引用变量使用{{}}，则Ansible会在任务执行时，替换该变量。如下：
```
vars:
  user: joe

tasks:
  # This line will read: Creates the user joe
  - name: Creates the user {{ user }}
    user:
      # This line will create the user named Joe
      name: "{{ user }}"
```
注意，我们建议引用变量时，一律使用双引号包括以防放声解析错误。
### 在Inventory中定义和使用变量
Inventory中的变量分为两类：
* 单一Host变量
  应用于单一host
* Groups变量
  应用于一组hosts

定义上面这两种变量有两种方法，如下：
* 老方式，直接定义在Inventory文件中。如下：
    ```
    [servers]
    demo.example.com  ansible_user=joe
    Defing the user group variable for the servers host group.

    [servers]
    demo1.example.com
    demo2.example.com

    [servers:vars]
    user=joe
    ```
    这种方法的缺点是它将变量和hosts信息混合在一起，让Inventory文件不太易读。
* 新方法： 使用group_vars和host_vars目录文件去定义变量。
    例如为一个servers group去定义group变量，则可以创建一个group_vars/servers的yaml文件，同理，如果为一个host定义host变量，则可以创建一个host_vars/host的yaml文件。示例如下：
   我们的Inventory文件如下：
    ```
    [admin@station project]$ cat ~/project/Inventory
    [datacenter1]
    demo1.example.com
    demo2.example.com
    
    [datacenter2]
    demo3.example.com
    demo4.example.com
    
    [datacenters:children]
    datacenter1
    datacenter2
    ```
    让我们添加group变量如下：
    ```
    [admin@station project]$ cat ~/project/group_vars/datacenters
    package: firewalld
    [admin@station project]$ cat ~/project/group_vars/datacenter1
    package: httpd
    [admin@station project]$ cat ~/project/group_vars/datacenter2
    package: apache
    ```
    如果我们只想设置host变量，则可设置如下：
    ```
    [admin@station project]$ cat ~/project/host_vars/demo1.example.com
    package: httpd
    [admin@station project]$ cat ~/project/host_vars/demo2.example.com
    package: apache
    [admin@station project]$ cat ~/project/host_vars/demo3.example.com
    package: mariadb-server
    [admin@station project]$ cat ~/project/host_vars/demo4.example.com
    package: mysql-server
    ```
    则此时的目录结构为：
    ```
    project
    ├── ansible.cfg
    ├── group_vars
    │   ├── datacenters
    │   ├── datacenters1
    │   └── datacenters2
    ├── host_vars
    │   ├── demo1.example.com
    │   ├── demo2.example.com
    │   ├── demo3.example.com
    │   └── demo4.example.com
    ├── Inventory
    └── Playbook.yml
    ```

### 通过命令行覆盖变量
像给定义在Inventory或者Playbook中的变量直接赋值，则可以通过在命令行指定参数：
```
[user@demo ~]$ ansible-Playbook main.yml -e "package=apache"
```

### 用数组表述变量
我们提高我们可以用yaml来表示变量，如下：
```
user1_first_name: Bob
user1_last_name: Jones
user1_home_dir: /users/bjones
user2_first_name: Anne
user2_last_name: Cook
user3_home_dir: /users/acook
```
则我们还可以用array来表示上述信息：
```
users:
  bjones:
    first_name: Bob
    last_name: Jones
    home_dir: /users/bjones
  acook:
    first_name: Anne
    last_name: Cook
    home_dir: /users/acook
```
然后，我们可以通过以下格式去访问其中变量：
users.bjones.first_name或者users['bjones']['first_name']
### 注册变量
管理员可以注册状态去捕捉一个命令的输出，这样输出就可以被保存以备于定位或者获取信息等，例如特定的配置等。示例如下：
```
---
- name: Installs a package and prints the result
  hosts: all
  tasks:
    - name: Install the package
      yum:
        name: httpd
        state: installed
      register: install_result

    - debug: var=install_result
```
当你运行该Playbook时，debug模块会用于转存install_result到终端并输出。如下：
```
TASK [debug] ***************************************************************
ok: [demo.example.com] => {
    "install_result": {
        "changed": false,
        "msg": "",
        "rc": 0,
        "results": [
            "httpd-2.4.6-40.el7.x86_64 providing httpd is already installed"
        ]
    }
}
```
