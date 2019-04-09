## Ansible架构
在Ansible架构中，有两种节点（机器）： 控制节点和被管理节点
* 控制节点
    Ansible被安装在控制节点，我们在控制节点上复制我们的Ansible项目文件，然后运行ansible任务。控制节点应该是管理员的机器，或者被许多管理员共享的系统，或者运行Red Hat Ansible Tower的一个服务器。
* 被管理节点
    在机器清单（inventory）中列出的被管理的结点，通常会被编辑成组以便于集中管理。这些机器清单可以在静态文件中定义，可以通过脚本动态从外部资源中获取。

## Ansible执行过程
Ansible用户创建高级别的plays去保证一个host或这一组host处于特定的状态。一个play在hosts顺序执行一系列的任务。这些plays都以YAML格式写在纯文本文件中。这种包含一个或这多个plays的文件就是一个playbook。
每个task运行一个module，或者具有一些特殊参数的一小段用python, PowerShell或者其他语言的代码块。每个module都是你工具箱至关重要的工具。Ansible发布了数百个有用的模块，可以用来执行各种自动化工作。他们可以操作系统文件，安装软件或者触发API calls。

当module在task中被使用时通常时用来保证这个机器处于默写特定状态。例如，一个task用一个模块去保证这个文件存在，且具有特定的权限和内容，同时，另一个task用另一个module去确定一个文件系统被挂载。如果系统已经处于某种状态，这个task就什么都不会做，否则，task则会执行，让系统处于这种预期状态下。默认，如果task在执行过程中失败，则会任务终止。

Tasks, plays和playbooks被设计为幂等的，也就是说你可在同一个host上多次运行tasks，如上段所示，而不会发生什么非预期的状况。Ansible还允许你执行任意命令，但是使用时，要多加小心，确保仍是幂等的。

Ansible还使用插件。插件是指可以加入ansible用以扩展新用法和新平台。

## Red Hat Ansible Tower
Red Hat Ansible Tower是一个企业级框架去帮助你批量控制，加密和管理你的Ansible自动化。
你可以用它控制谁有权限在那个host执行哪些playbooks，分享ssh证书而不需要允许他们看到内容，记录你的ansible任务的过程，管理你的机器清单（inventory）等许多事情。

它提供了一个web用户接口（web UI）和一个RESful API。 
Ansible Tower并不是Ansible的核心部分，而是一个独立的产品去帮助你更高效更有规模的使用ansible。
