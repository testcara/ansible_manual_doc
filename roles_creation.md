## 创建Role
### 创建role目录结构
默认，Ansible会在Playbook中查找roles或者叫roles的子目录。
如果Ansible找不到role,则Ansible会在配置文件中查看roles_path指定的目录。默认，roles_path的路径为
~/.ansible/roles:/user/share/ansible/roles:/etc/ansible/roles
这就允许你在系统上安装roles，共享给多个项目。例如，你可以安装自己的roles在～/.ansible/roles下，可以将所有用户共享的roles放在/usr/share/ansible/roles目录。
每一个role都有符合标准目录的目录结构，如下：
```
[user@host ~]$ tree roles/
roles/
└── motd
    ├── defaults
    │   └── main.yml
    ├── files
    ├── handlers
    ├── meta
    │   └── main.yml
    ├── README.md
    ├── tasks
    │   └── main.yml
    └── templates
        └── motd.j2
```
这就定义了motd role。
README.md提供了基本的role的语言描述，文档以及如何使用它的例子等。其他的目录在上节中已讲解，这里不再重复。
如果子目录为空，例如本例子中的hanlder目录，则该目录就会被忽略。我们也看到本里中没有vars目录，这是因为该role可能用不到，所以没有提供。
### 创建一个Role的架构
你使用linux基本命令可以创建一个role所需要的所有的子目录和文件，但是，现有的命令行工具已经自动化了创建一个role的过程。
这个工具就是ansible-galaxy命令行工具。你可以执行ansible-galaxy去初始化一个新role的目录结构。
则过程如下
```
mkdir roles
cd roles
ansible-galaxy init my_new_role
ls my_new_role
```
### 定义角色内容
一旦创建了这个目录结构，我们就可以编写这个role的内容。最好是从ROLENAME/tasks/main.yml着手，因为这个文件记录了这个角色要做的主要任务。
假设我们有以下mail.yml：
```
[user@host ~]$ cat roles/motd/tasks/main.yml
---
# tasks file for motd

- name: deliver motd file
  template:
    src: motd.j2
    dest: /etc/motd
    owner: root
    group: root
    mode: 0444
```
我们看到这个role是为了管理被管理机器上的/etc/motd文件。我们看到其使用了motd.j2文件，则该文件为：
```
[user@host ~]$ cat roles/motd/templates/motd.j2
This is the system {{ ansible_facts['hostname'] }}.

Today's date is: {{ ansible_facts['date_time']['date'] }}.

Only use this system with permission.
You can ask {{ system_owner }} for access.
```
这个角色定义了system_owner变量一个默认的值，该值定义在defaults/main.yml中，如下：
```
[user@host ~]$ cat roles/defaults/main.yml
---
system_owner: user@host.example.com
```
### 推荐的Role的开发实践

Roles允许playbooks以模块的方式编写。为了最高效的开发新的role，我们有以下建议：

- 每一个role独享版本控制管理repo。
- 敏感信息，例如密码或者SSH keys,不同存储在role的repo里面。敏感值应该参数化为  变量，且默认值为非敏感的。Playbook可以使用Ansible Vault变量文件去存储敏感变量，环境变量后者其他Ansible-playbook的选项。
- 用ansible-galaxy去初始化role,然后删除你不需要的目录和文件。
- 创建并维护READM.md和meta/main.yml文件去记录这个role的作用，谁写了它，以及如何使用它。
- 保证你的role的目的性或者功能单一。如果你的role要做很多事情，则考虑创建多个role
- 经常复用或者重构role。避免为边缘配置创建新的role。如果现有的role满足了大多数的配置需求，重构你的role去集成新的配置场景。用集成和回归测试去保证这个role提供了新的功能和且老的功能没有回退。
### 定义role的依赖
Role依赖允许一个role去包含其他的role作为依赖。例如，一个定义文档服务器的role可以依赖于其他安装和配置web服务器的role。依赖定义在meta/mail.yml文件中的role的层级结构中。例如：
```
---
dependencies:
  - role: apache
    port: 8080
  - role: postgres
    dbname: serverlist
    admin_user: felix
```
默认，roles一次仅会向playbook中添加一个依赖。如果其他role也依赖于该依赖，则该依赖默认不会被重复执行，但是这种行为可以通过设置meta/main.yml文件中‘allow_duplicates’为yes来控制行为。
虽然我们可以定义role依赖，但是不建议定义和使用太多role依赖，因为这会使得role很难维护。
### 在playbook中使用role
我们在‘定义角色内容’章节中，定义个motd role要做的任务，则我们在以下playbook中引用这个role。
```
[user@host ~]$ cat use-motd-role.yml
---
- name: use motd role playbook
  hosts: remote.example.com
  user: devops
  become: true
  roles:
    - motd
```
当这个playbook被执行时，motd role的任务会被执行。因为motd role会被这个命令前缀识别到。当任务被执行时，我们可以看到motd这个前缀：
```
TASK [motd : deliver motd file] ************************************************
changed: [remote.example.com]
```
我们也可以给role的变量赋值，如下：
```
[user@host ~]$ cat use-motd-role.yml
---
- name: use motd role playbook
  hosts: remote.example.com
  user: devops
  become: true
  roles:
    - role: motd
      system_owner: someone@host.example.com
```
