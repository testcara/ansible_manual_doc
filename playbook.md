## Ansible Playbooks
上章我们学习了adhoc命令，一条指令定义一个简单的任务在所有的目标机器上执行。只有学习了怎么用playbooks在一系列机器上以很简单可重复的方式运行多个，复杂的任务，你才会知道Ansible的强大。

一个play是一组定义好顺序的执行于一组机器的任务集合。一个playbook是包含一个或者多个play的纯文本。

Plays可以将长而复杂的手动执行的管理员任务编程简单的可重复的工作列表。在一个playbook中，你可以顺序的将这些工作用可描述的语言写成可执行的任务。
## Ansible Playbook的格式
先看我们熟悉的ad hoc命令
```
ansible servera.lab.example.com -m user -a "name=newbie uid=4000 state=present“
```
则写成playbook则为：
```
---
- name: Configure important user consistently
  hosts: servera.lab.example.com
  tasks:
  - name: newbie exists with UID 4000
    user:
      name: newbie
      uid: 4000
      state: present
```
保存为.yml文件。

我们来分析这个playbook，可理解为：
```
开头总是三个dash
- name: 一个play的名字
  hosts: 目标被管理机器
  tasks:
  - name: 任务1
    module_name:
      parameter1_key: parameter1_value
      parameter2_key: parameter2_value
      parameter3_key: parameter3_value
```
## 执行playbook
假设我们保存上例中playbook为site.yml,则我们可执行如下
```
ansible-playbook site.yml --become
```
则我们看到play和task被执行, 被管理机器有改变则为changed,否则不变。

## 增量输出
Ansible默认的输出不会提供特别具体的任务执行信息。我们可以使用-v来让其提供更多的信息。v的个数提供了信息输出的等级。
具体信息如下：
| v | 显示信息 |
|----|----|
| -v | 显示结果 |
|-vv| 显示任务配置和结果 |
|-vvv| 还显示被管理机器连接的信息 |
| -vvvv | 还显示连接的插件，包括在被管理机器上的执行用户及执行的脚本信息等|
## 语法校验
在执行Playbook之前，执行语法校验保证语法正确是非常好的。ansible-playbook命令提供了一个‘--syntax-check’选项我们可以用来验证playbook的语法。用法如下
```
ansible-playbook --syntax-check webserver.yml
```
如果语法校验失败，则语法错误会返回。
## 空运行
除了语法校验外，我们还可以空运行我们的playbook。通过空运行我们可以看到我们playbook执行的过程，以及会导致哪些变化。空运行并不会在实际的被管理机器上。命令如下：
```
ansible-playbook -c webserver.yml
```
