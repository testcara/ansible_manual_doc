# 安装Ansible
Ansible非常容易安装且仅仅需要安装在Control node上。被管理机器不需要安装Ansible。
控制结点需要时Linux或者UNIX系统。Windows系统不支持控制结点，尽管Windoes系统也可以用来管理机器。
Python 2 (Version 2.7 or later)或者Python 3(Version 3.6 or later)需要在控制结点上安装。我们以centos为例，通过以下命令确认python已经安装
```
$ yum list installed python
Loaded plugins: changelog, fs-snapshot, priorities, refresh-packagekit, rhnplugin, rpm-warm-cache, verify
*Note* Spacewalk repositories are not listed below. You must run this command as root to access Spacewalk repositories.
Repository google-chrome is listed more than once in the configuration
Installed Packages
python.x86_64                                                     2.7.5-69.el7_5                                                      @production-rhel-x86_64-workstation-7.5
```
Then install the ansible
```
$ sudo yum install ansible
```
我们知道，Ansible控制结点需要和被控制结点通过网络进行通讯。默认会用到SSH， 但是如果被管理的结点时Miscrosoft Windows系统的化，则我们还需要用到其他的协议。在Centos上，如果被管理的结点是WIN时，则我们仍需要安装‘**python2-winrm**’的0.3.0或者更新版本的包。
被管理结点不需要安装Ansible, 但是根据控制结点如何和他们进行链接以及那些模块在其上运行，会有一些依赖需要解决。
Linux或者UNIX的被管理结点需要安装Python 2（version 2.6或者更新版本—）或者Python 3(version 3.5或者更新的版本)，这样绝大多数的moudle才能工作。
如果在被管理机器上安装**SELinux**并使能了，你还需要安装**libselinux** python包。
还有一些module有他们自己独有的依赖，例如d'**dnf**'模块则需要安装‘**python-dnf**’包
Ansible提供了许多专门服务于Microsoft Windows system的modules。绝大多数模块需要安装**PowerShell 3.0**或者更新版本，而不需要安装python。另外，在被管理的Win上，这个PowerShell需要配置。除了PowerShell, .NET Framework 4.0或者更新的版本也是需要安装在被管理的Win结点上。

除了**Linux**和**WIN**系统，我们还可以用Ansible来配置被管理的网络设备，例如路由器和交换器。同样的，Ansible提供了很多相应的模块，例如Cisco IOS, IOS XR和NX OS等。

我们可以用同样的**基础知识**去编写Ansible Playbooks。因为绝大多数的网络设备不支持Python，我们在控制结点运行network modules，而不是在被管理结点上，这一点和其他类型的被管理机不同。一些特殊的链接方法可以用来和网络设备进行交流，例如CLI over SSH, XML over SSH或者API over HTTP(s)。

**我们后面的例子均以Linux作为被管理机**。
