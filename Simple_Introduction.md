## 什么是Ansible
Ansible是一个开源的自动化平台，是一种可以在Ansible playbook中用于描述IT应用架构的简单语言， 还是一个用来运行Ansible Playbook的自动化引擎。
Ansible可以管理强大的自动化任务，适用于不同的工作流程和环境。同时，即使是新人也可以很快的且很高产的掌握和应用Ansible。
## 简单的Ansible
Ansible Playbook提供了人类可读的自动化，也就是说playbooks是人类易读，易理解和易改变的自动化工具。编写playbooks并不需要什么特殊的编程技能。Playbooks顺序执行任务。总的来说，playbook的简单设计使得每个IT人都可以很方便的很高效的使用它。
## 强大的Ansible
你可以使用Ansible去部署应用，配置管理，流程自动化，网络自动化等。Ansible可以用来编排整个应用的生命周期。
## 无客户端的Ansible
Ansible是基于无客户端的结构进行编译的。通常，Ansible通过OpenSSH后者WinRM链接它管理的host，然后运行任务，一般会推送一些小程序（Ansible modules）到host，这些程序会将系统变成我们需要的特定的状态， 当任务完成后，这些被推送过去的Ansible modules会被删除。
由于Ansible无客户端特性，你可以随时使用Ansible去管理你的hosts而不用去考虑客观端和服务端安全架构等其他信息。

## 总结Ansible关键优势
* 支持跨平台 - agentless支持Linux/windows/UNIX/network，各种物理的，云的，容器的环境
* 易读的自动化- Ansible Playbooks是YAML格式，非常容易读。
* 对应用的完美表述 - Ansible playbooks可以做所有环境改变，应用环境的每一个方面都可以被很好描述。
* 很便捷的版本控制 - Ansible Playbooks和项目都是纯文本，所以我们可以就想像管理源代码一样管理它，将其放入你的版本控制系统中
* 支持动态的host清单 - Ansible管理的机器列表可以根据外部输入动态更新，以获取所有正确的可管理的服务器列表。
* 和谐的编排 - Ansible可以很好将外部系统，例如：HP SA， Puppet, Jenkins, Redhat Satellite等其他在你环境中存在的系统集成在Ansible workflow中。
沟通是DevOps的关键。Ansible可以是第一个自动化你需要读和写的，其也是唯一的自动化引擎，可以自动化应用的生命周期以及继续交付(continuous delivery pipeline)。
