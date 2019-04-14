## Ansible Facts
### Ansible Facts简介
Ansible facts是Ansible从被管理机上自动收集来的变量。Facts包含了一些宿主机特有的信息，而这些信息可和其它常规变量一样在plays中使用于各种场合。
从一个被管理机搜集到的信息通常包含：
* The hostname
* The kernel version
* The network interfaces
* The IP addresses
* The version of the operating system
* Various environment variables
* The number of CPUs
* The available of free memory
* The available disk space

Facts提供了一种非常便捷的方式让我们从被管理机器上收集状态信息，我们可以根据不同的状态信息采取不同的措施。例如：
* 基于获取到的kernel version信息，来决定是否重启机器
* 基于获取到的可使用的内存来配置Mysql
* 基于获取到的IPv4 address来指定配置文件的值 

正常的，每一个play在执行第一个task之前都会自动执行setup模块去自动收集facts。我们通常无需在play中再创建tasks去收集facts。
如果你想查看Ansible自动都收集了哪些facts，则可使用我们以前章节中用过的‘debug’模块。如下：
```
- name: Fact dump
  hosts: all
  tasks:
    - name: Print all facts
      debug:
        var: ansible_facts
```
则当你运行playbook时，这些facts就会在job中输出如下：
```
[user@demo ~]$ ansible-playbook facts.yml

PLAY [Fact dump] ***************************************************************

TASK [Gathering Facts] *********************************************************
ok: [demo1.example.com]

TASK [Print all facts] *********************************************************
ok: [demo1.example.com] => {
    "ansible_facts": {
        "all_ipv4_addresses": [
            "172.25.250.10"
        ],
        "all_ipv6_addresses": [
           "fe80::5054:ff:fe00:fa0a"
        ],
        "ansible_local": {},
        "apparmor": {
            "status": "disabled"
        },
```
我们可以看到ansible_facts输出为JSON格式的hash/directory变量以及其包含的信息。我们可以在play中使用ansible_facts.default_ipv4.address/ansible_facts['default_ipv4']['address']等fact，play被执行时，这些facts变量会被直接替换成对应的值。
### 关掉自动收集Facts
有时，我们并不想收集facts以提高我们play的执行速度或者减少被管理机上的负载等。
我们可通过以下指令让play停止自动收集facts:
```
---
- name: This play gathers no facts automatically
  hosts: large_farm
  gather_facts: no
```
### 手动收集Facts
在某些场景下，自动收集facts被关闭，则我们可以通过以下指令手动收集facts:
```
  tasks:
    - name: Manually gather facts
      setup:
```
## 定制化Facts
除了setup模块自动收集的facts，如果我们还需要收集其他的facts，则我们需要定制我们的facts。定制化的facts可被定义在被管理结点的一个INI或者JSON的静态文件中，对的，不支持YAML格式。默认的，setup模块会从被管理结点的/etc/ansible/facts.d目录下加载以.fact后缀命令的文件或者脚本
假设我们现在有如下定制化的Fasts，格式为INI，保存为/etc/ansible/facts.d/custom.fact
```
[packages]
web_package = httpd
db_package = mariadb-server

[users]
user1 = joe
user2 = jane
```
则我们运行setup模块时，可看到以下输出：
```
    "ansible_facts": {
...output omitted...
        "ansible_local": {
            "custom": {
                "packages": {
                    "db_package": "mariadb-server",
                    "web_package": "httpd"
                },
                "users": {
                    "user1": "joe",
                    "user2": "jane"
                }
            }
        },
```
则我们可以使用ansible_local.custom.packages.db_package或者ansible_local['custom']['packages']['db_package']来引用定制化的变量。
### 魔法变量
魔法变量也是Ansible自动配置的但并不是facts或者由setup模块完成的。常见的魔法变量有：
* hostvars
  包含了被管理结点的变量
* group_names
  列出该被管理结点所处的所有groups
* groups
  列出inventory中的所有groups和hosts
* inventory_hostname
  列出inventory中当前被管理结点的hostname。该值可能和facts中获取的hostname不一致。

自然，我们还有许多没有列出的魔法变量，可查看[Ansible variables相关的官方文档](https://docs.ansible.com/ansible/2.7/user_guide/playbooks_variables.html#variable-precedence-where-should-i-put-a-variable)
我们可以通过debug模块直接查看到这些变量的值，如下：
```
 ansible localhost -m debug -a 'var=hostvars["localhost"]'
```
