## 执行Ansible ad-hoc命令
一个ad hoc命令是一种快速执行单一Ansible任务的方法。通常，不需要重复的运行的任务我们会用ad hoc命令。他们简单可直接操作，而不需要写playbook。
ad hoc命令对于快速测试和变化是非常有用的。例如，你可以用一个ad hoc命令去保证所有机器上某行都确定存在于某一个文件中。你可以使用其他的ad hoc命令去在许多不同的机器上重启服务，或者一个软件包是最新的。

ad hoc是使用有它独有的限制，如你不可能使用Ansible Playbooks去使用Ansible所有功能。
我们可以使用以下命令来运行ad hoc命令
```
ansible host-pattern -m module [-a module_args] [-i inventory]
```
让我们以ping module为例来尝试ad hoc命令，如下
```
ansible mysql -m ping
```
Moduls是ad hoc命令用来完成任务的工具。Ansible提供了数百个模块去做不同的事情。你通常可以找到一个经过测试的满足一定用途的模块并且已经安装在控制节点上。
查看系统上已安装的所有模块，我们可以使用以下命令：
```
ansible-doc -l
```
我们可以使用以下命令去查看某个模块的说明文档
```
ansible-doc moudle_name
```

## 常用的moudle
|  模块类|模块   | 用途|
|---|---|---|
|File moudles| copy   | 拷贝文件  |  
|            | file   | 设置文件属性  |  
|            | lineinfile  | 确保某行在或者不再文件中 |  
|            | synchronize | 同步内容 |
|Software package modules| package | 通过系统安装的包管理包管理包 |
|                        | yum | 用YUM包管理管理包 |
|                        | apt | 用APT包管理管理包 |
|                        | dnf | 用DNF包管理管理包 |
|                        | gem | 管理Ruby gems |
|                        | pip | 用PyPI管理Python包 |
|System modules| firewalld | 使用firewalld管理任意端口和服务 |
|              | reboot    | 重新启动机器 |
|              | service   | 管理服务 |
|              | user      | 增加，删除以及管理用户帐号 |
|Net Tools modules| get_url | 通过HTTP，HTTPS或者FTP下载文件 |
|                 | nmcli   | 管理网络 |
|                 | uri     | 和web服务交互 |
绝大多数的模块需要参数。你可以通过查看模块的说明文档来查看可用参数。Ad hoc命令通过-a传递参数。当没有参数时，忽略-a参数。如果需要多个参数，则我们需要用引号包括。例如：
```
ansible -m user -a 'name=newbie uid=4000 state=present' 
```
## 在被管理机器上执行任意指令
我们可以使用command模块实现，如下
```
ansible mymanagedhosts -m command -a /usr/bin/hostname
```
默认范围时多行，如果想单行返回，则加上‘-o’参数
command模块允许管理员很快捷的在被管理机器上执行命令。但是这些命令并不是被管理器机器上的shell所处理的。所以，它不能获取shell的环境变量或者执行shell的操作，如果重定向和管道。
如果我们需要shell处理，我们需要使用**shell**模块，如下：
```
ansible localhost -m shell -a set
```
无论时command还是shell模块都需要在被管理机器上安装Python。‘raw’第三方模块，可以直接在被管理机器上执行shell，如果被管理机器上没有安装Python，我们可以使用raw模块。

通常，我们不建议使用command, shell和raw模块的，因为我们有许多其他的模块，执行起来更安全。

## 配置ad-oc命令的连接
我们在上一章中将到过，在ansible.cfg中扩展权限。我们在ad hoc时也可以实现该功能。通过查看‘ansible --help’可在增加权限处看到参数和选项。
