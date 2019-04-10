## Ansible配置
这一章节中，我们学习如何配置Ansible持久化用户配置
Ansible的配置文件通常命名为ansible.cfg，在该文件中我们可以：
* 配置默认的清单
    ```
    [defaults]
    inventory=./inventory
    ```
* 配置切换root权限
通过以下配置，Ansible在管理机器时可以自动的使用sudo命令从student切换到root。Ansible执行sudo指令时会请求你输入Password
    ```
    [privilege_escalation]
    become = true
    become_user = root
    become_method = sudo
    become_ask_pass = true
    ```
## 检验配置
执行完这些配置后，我们可以使用上一节我们用过的指令来检验有无问题：
    ```
    ansible myself --list-hosts
    ```
如果没有warning, hosts正常返回，则配置没有问题。
