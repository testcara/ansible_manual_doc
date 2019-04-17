## 模版文件
### Jinja2简介
Ansible中许多模块可以用来编辑现有的文件，例如lineinfile, blockinfile等。但是，他们有时候并不总是高效的和准确的。
一个更强大的管理文件的方式就是去模版化他们。你可以写一个模版配置文件，当在被管理机上部署该文件时，会用Ansible变量和facts被自动的定制化。使用模版文件进行部署，会更好控制和减少错误。

Ansible使用Jinja2模版系统去模版化文件。我们之前接触过的在文件中用双括号去引用变量的语法就是Jinja2的语法。

### 使用Jinja2
一个Jinja2模版时有多个元素，例如数据, 变量和表达式组成。当模版被渲染时，这些变量和表达式会被值所代替。模版中使用的变量可在playbook中vars中定义，也可以引用被管理机的facts作为模版中的变量，如下
```
Welcome to {{ ansible_facts.hostname }}.
Today's date is: {{ ansible_facts.date_time.date }}
```
当以上模版所关联的playbook被执行时，template中会被值所代替。
在Jinja2中除了使用{{}}来于输出变量或者表达式的输出，而用{%%}来输出表达式和逻辑块。
另外，模版文件不需要特殊的文件后缀，例如.j2。但是，我们以这样的文件后缀命名可以让我们更清楚的认识到这是一个模版文件。
### 部署Jinja2模版
当我们已经创建了类似以上的模版配置文件，我们就可以使用template模块去将其部署到被管理机器上。如下：
```
tasks:
  - name: template render
    template:
      src:/tmp/j2-template.j2
      dest: /tmp/dest-config-file.txt
```
### 管理模版文件
为了表明模版生成的配置文件是Ansible自动生成的，而非管理员手动编辑，我们会在template文件的开始添加一条声明。
常见的做法是，我们会在ansible.cfg中定义变量“ansible_managed = Ansible managed”， 然后在template中使用“{{ ansible_managed }}”。
### 控制结构
你可以在template文件中使用Jinja2控制结构去减少重复输入。
Jinja2使用for表述去提供循环功能，如下
```
{% for user in users %}
    {{ user }}
{% endfor %}
```
让我们来举一个具体的例子，如下：
```
- name: /etc/hosts is up to date
  hosts: all
  gather_facts: yes
  tasks:
    - name: Deploy /etc/hosts
      template:
        src: templates/hosts.j2
        dest: /etc/hosts
```
我们定义hosts.j2如下：
```
{% for host in groups['all'] %}
{{ hostvars[host]['ansible_facts']['default_ipv4']['address'] }} {{ hostvars[host]['ansible_facts']['fqdn'] }} {{ hostvars[host]['ansible_facts']['hostname'] }}
{% endfor %}
```
则该play会用该模版生成文件并部署到被管理机器上。
Jinja2除了会用for控制语句外，还会用if去提供控制支持。如下
```
{% if finished %}
{{ result }}
{% endif %}
```
### 变量过滤
Jinja2提供了过滤去改变模版中表达式输出格式，例如指定输出to_json或者to_yaml，也可以从输入解析不同格式的文件，通过执行输入格式例如‘from_json’或者‘from_yaml’来完成。
