## 机器清单
一个inventory定义了Ansible需要管理的机器的集合。这些机器会被分组以便于集中管理。
组还以包含子组，机器可以同时是多个组的成员。这个inventory也可以设置变量，应用于其定义的hosts和groups。
我们有两种不同的方法定义host inventories。静态的host inventory可以被定义在文本文件中。动态的host inventory可以通过脚本或者其他的程序基于静态脚本生成。

## 静态清单
我们可以用INI-style来解inventory的YAML文件。
INI-style（每行一个hostname/ip）如下：
```
web1.example.com
web2.example.com
db2.example.com
192.0.2.42
```
通常，我们将我们的管理的机器分组，每组名字需要用‘[]’来包裹。示例如下：
```
[web-server]
web1.example.com
web2.example.com
[db-server]
db2.example.com
192.0.2.42
```
同一个host可以存在在多个分组中，只要组织方式合理。
譬如我们可以根据web-server或者db-server进行分组，也可以根据target host进行分组，譬如针对Production hosts进行分组等。
```
[webservers]
web1.example.com
web2.example.com
192.0.2.42
[db-servers]
db1.example.com
db2.example.com
[east-datacenter]
web1.example.com
db1.example.com
[west-datacenter]
web2.example.com
db2.example.com
[production]
web1.example.com
web2.example.com
db1.example.com
db2.example.com
```
如上述案例中，我们显示定义了一些分组。实际上，有两个分组我们不需要显示定义也存在。这就是'all'和‘ungrouped’。
## 定义嵌套分组
Ansible host inventories还可以包括分组的分组，这可以通过':chinldren'后缀实现。如下所示：
```
[usa]
washington1.example.com
washington2.example.com
[canada]
ontario01.example.com
ontario02.example.com
[north-america:children]
canada
usa
```
则我们看到north-america在这里可以理解为all
一个分组可以含有被管理的hosts和子group。 例如
```
[north-america:children]
canada
usa
10.72.8.136
````
## 通过范围分组
你可以在host names中设定范围或者IP地址去简化Ansible host inventories，这里的范围可以是数字范围也可以字母范围。格式如下：
```
[START:END]
```
示例如下：
* 2.168.[4:7].[0:255] 会匹配到所有在192.168.4.0和192.168.7.255的机器
* server[01:20].example.com会匹配到所有名字为server01.example.com到server20.example.com的机器
#验证清单
我们定义完清单，可通过以下指令进行简单的验证
```
$ ansible washington1.example.com --list-hosts
 hosts (1):
    washington1.example.com
```
如上所示，hosts(1)表示存在，否则为0表示不存在。
你可以通过以下指令，列出一个组中所有的机器
```
$ ansible canada --list-hosts
  hosts (2):
    ontario01.example.com
    ontario02.example.com
```
如果一个清单中，有名字相同的host和host group，则ansible命令行则会输出警告并且host group则会被忽略，所以我们建议不重复命名。
## 重写清单
/etc/ansible/hosts是默认的静态清单。但是，在实际应用中，我们通常不会用它，而是定义我们自己的清单，然后通过‘--inventory PATHNAME’或者‘-i PATHNAME’参数指定inventory的地址。

## 在清单中定义变量
在playbooks中使用的变量的值可以在清单中指定。这些变量仅应用于特殊的hosts和groups。
通常，不直接在inventory文件中定义变量而在其他特定的目录中定义变量是更好方法。我们会在后续章节中在讲解。

## 动态清单
Ansible清单信息可以动态生成。在Ansible的开源社区，有许多生成动态清单的脚本。如果这些脚本不符合我们的要求，则我们可以自己的动手写。
例如，一个动态的inventory progeam可以和你的server建立链接，获取到存储的信息，然后生成相应的inventory。
这个用法在后面章节也会在进行讲解。
