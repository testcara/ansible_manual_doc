## 用Roles来组织Playbooks
### Roles简介
当你开发了足够多的playbooks，你会发现你有很多机会去复用你已经写好的playbooks中的代码。例如一个配置mysql的play, 通过指定不同的主机名，密码和用户，就可以变成不同应用的数据库。
但是，如果一个play中，我们包含和倒入了许多的tasks或者playbooks，则这个play很有可能很长很复杂。
Ansible roles提供了另外一种方式让我们更容易去复用Ansible代码。你可以以一种标准的目录结构去打包，所有的工作，变量，文件，模版和其他用于创建基础架构和构建应用的所有相关资源。你可以通过复制目录在不同项目之间来回复制角色。你可以简单的调用play中的角色去执行这个角色负责的所有任务。

写的非常好的角色允许你实时传递变量来调整行为，来配置所有hostname, IP, usernames, secrets等。
Ansible roles有以下长处：
- Roles分组内容，让我们更容易分享代码
- Roles可以用来定义一个系统的所有关键元素，例如，web服务器，database服务器，Git版本管理等
- Roles让大型项目更好管理
- Roles支持不同的系统管理员并行开发
除了写，用，重用和分享你的roles,你还可以从其他地方获取到roles。
### Role文件结构
一个Ansible role拥有通用的子目录和文件结构。第一层目录定义了Role本身，子目录根据不同的目的来存放文件，例如tasks用来存放任务和handlers用来存放事物处理器，模版子目录用来存放模版等。如下：
```
[user@host roles]$ tree user.example
user.example/
├── defaults
│   └── main.yml
├── files
├── handlers
│   └── main.yml
├── meta
│   └── main.yml
├── README.md
├── tasks
│   └── main.yml
├── templates
├── tests
│   ├── inventory
│   └── test.yml
└── vars
    └── main.yml
```
这些不同目录的作用如下：
|子目录|作用|
|---|---|
|defaults|main.yml为恶年包含了这个role的变量的默认值，这些值可在role被执行时替换|
|files|这里存放被role tasks需要引用的的静态文件|
|meta|main.yml包含了这个role的信息，例如作者，证书，平台，角色依赖等|
|tasks|mail.yml定义这个role的任务|
|template|定义了Jinja2模版，这些模版会被role tasks引用|
|tests|包含了一个inventory和test.yml用来测试这个role|
|vars|main.yml定义这个role变量的值，通常这些变量是内部使用。在我们执行playbook时，这些变量值不变|
注意，并不是每个role都需要有所有这些目录。
### 定义vars和defaults
我们刚介绍过了vars和defaults目录的作用，我们知道这两个目录的的都是用来配置变量的。这main.yml定义了一组组键值对，main.yml中定义的变量可被通过{{ VAR_NAME }}来被引用。defaults中定义的默认值。
我们可以在vars/main.yml或者defaults/main.yml中定义变量，但是不能在两处都定义。如果我们预期该变量的值会被重写，则建议定义在defaults/main.yml中。
Roles不应该包含敏感数据，例如密码或者私有钥匙。因为roles预期是通用的，可复用的，可共享的，其中不应该有任何硬编码。
敏感数据应该通过其他方式提供给role，例如变里的方式。
### 在playbook中使用role
在Playbook中使用role,是非常简单的，如下：
```
---
- hosts: remote.example.com
  roles:
    - role1
    - role2
```
如上，role1和role2，所有role的tasks, handlers, variables, dependencies都会被倒入这个playbook中。
role1会先执行，role2会后执行。
## 执行顺序的控制
Playbook中的tasks总是顺序执行的，等所有任务执行完，被唤醒的hanlder会顺序执行。
当role被倒入到Playbook时，role的handler也被倒入该文件中，且执行顺序会在原playbook的handler之前。
在特定的场景，在role之前有必要先执行以下其他的任务，则‘pre_tasks'可以帮助我们实现这种功能。在pre_tasks中的所有任务会在所有role执行前执行。如果pre_tasks中任一任务触发了handler，则handler也会在所有role之前执行。与之相似的，我们还有‘post_tasks’关键字，这些任务会在其他任务执行完成后执行。
让我们来看一下下面这个例子，来进一步了解pre_tasks, roles, tasks,post_tasks和handlers的执行顺序。
```
- name: Play to illustrate order of execution
  hosts: remote.example.com
  pre_tasks:
    - debug:
        msg: 'pre-task'
      notify: my handler
  roles:
    - role1
  tasks:
    - debug:
        msg: 'first task'
      notify: my handler
  post_tasks:
    - debug:
        msg: 'post-task'
      notify: my handler
  handlers:
    - name: my handler
      debug:
        msg: Running my handler
```
则执行的顺序为pre_tasks的debug,my hander,role1, tasks，post tasks的debug,最后为my handler
role也可以通过import_role和include_role来倒入到play中，如下
```
- name: Execute a role as a task
  hosts: remote.example.com
  tasks:
    - name: A normal task
      debug:
        msg: 'first task'
    - name: A task to include role2 here
      include_role: role2
```
