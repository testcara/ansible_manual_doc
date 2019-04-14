## 管理Secrets
Ansible需要权限获取敏感数据去配置被管理机，例如passwords或者API keys。
通常，这些信息可以以明文形式存储在inventory的变量中或者其他文件中，但是这种情况下，所有可获取该Ansible项目的人都可获取到这些敏感数据，这存在非常明显的安全问题。
### Ansible Vault
Ansible Vault工具可用来加密解密敏感数据。
* 创建加密文件的两种方式
    * 直接输入密码来创建加密文件 
        ```
        ansible-vault create secret.yml
        ```
        输入两遍密码后会默认打开vi让你编写secret.yml文件，让你输入敏感信息，完成后保存即可。
    * 指定密码文件来创建加密文件
        ```
        ansible-vault create --vault-password-file=vault-pass secret.yml
        ```
        我们已经事先将密码写入vault-pass文件，指定文件后，则直接默认打开vi让你编写secret.yml文件，让你输入敏感信息，完成后，保存即可。
* 查看和编辑加密文件
使用ansible-vault进行加密的文件，使用vi直接打开看到是一串串加密后的数据。要想进行有意义的查看还编辑仍旧需要使用ansible-vault工具。在查看和编辑指定密码时，仍然有两种方式，可直接通过命令行输入也可指定密码文件。
如以下两个例子：
    ```
    ansible-vault edit --vault-password-file=vault-pass secret.yml
    ansible-vault view secret.yml
    ```
* 加密现有的文件
当我们已经有了一个包含敏感数据的文件，我们想对其进行加密，则我们可以对其进行加密处理，如下：
    ```
    ansible-vault encrypt test_encrypt.yml
    ```
    则输入密码后，test_encrypt.yml会被直接加密，我们可以不直接加密该文件，而将加密后的输出使用‘--output=’定向到另一个文件中。
* 解密已加密的文件
当我们已经被加密的文件后，我们可以直接使用以下命令来进行解密，使其变成明文文件。如下：
    ```
    ansible-vault decrypt test_encrypt.yml
    ```
    则输入密码后，test_encrypt会被直接解密变成明文文件，我们可以不直接解密该文件，而将解密后的输出使用‘--output=’定向到另一个文件中。
* 更改加密的密码
当我们对一个文件进行加密后，我们想改变其加密密码，则我们可使用以下指令：
    ```
    ansible-vault rekey test_encrypt.yml
    ```
   直接输入新的密码或者如下指定新的密码文件：
   ```
   ansible-vault rekey --new-vault-password-file=vault-pass test_encrypt.yml
   ```
 * Playbook和Ansible vault
如果playbook中使用到了使用Ansible vault加密的文件，我们必须给ansible-playbook命令提供密码，或者指定密码文件，否则如下错误信息会返回：
    ```
    [student@demo ~]$ ansible-playbook site.yml
    ERROR: A vault password must be specified to decrypt vars/api_key.yml
    ```
    可通过终端提供密码使用如下指令：
    ```
    ansible-playbook --vault-id @prompt site.yml
    ```
    可通过指令指定密码文件
    ```
    ansible-playbook --vault-password-file=vault-pw-file site.yml
    ```
### 变量文件管理的最佳实践
为了管理的方便，在一个Ansible项目中，将敏感数据和其他变量数据同项目中的其他数据相分是非常有意义的。包含敏感信息的数据可用ansible-vault加密。
我们之前用过如下的目录结构来管理变量：
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
则基于上述结构，当敏感数据变量存在于文件中时，则需要对整个文件进行加密。
然而，实际应用中，我们可将不需要加密的变量文件都放入vars的目录，而将需要加密的变量统一管理为加密文件，放入vault文件夹，则会出现如下的目录结构：
```
project
├── ansible.cfg
├── group_vars
│   └── datacenters
│       └── vars
├── host_vars
│   └── demo.example.com
│       ├── vars
│       └── vault
├── inventory
└── playbook.yml
```
如上，敏感变量和非敏感变量实现了分离。管理员则可以使用ansible-vault对vault-file进行加密。
