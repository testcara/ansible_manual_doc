## Ansible事件处理器
当一个事件发生，另一个事件必须一同发生，才能完成特定任务。例如我们改变了一个服务的配置文件，则我们需要重新加载这个配置，才能时新的配置生效。
这种场景下，我们就可以等更新配置任务完成后通知事件处理器，重启服务。如下：
```
tasks:
  - name: copy demo.example.conf configuration template
    template:
      src: /var/lib/templates/demo.example.conf.template
      dest: /etc/httpd/conf.d/demo.example.conf
    notify:
      - restart apache

handlers:
  - name: restart apache
    service:
      name: httpd
      state: restarted
```
当事物处理器接收到任务发来的用来唤醒事物的通知，则就会重启服务。
这个例子中，任务只唤醒一个事物，实际应用中，也可以唤醒多个事物，如下：
```
tasks:
  - name: copy demo.example.conf configuration template
    template:
      src: /var/lib/templates/demo.example.conf.template
      dest: /etc/httpd/conf.d/demo.example.conf
    notify:
      - restart mysql
      - restart apache

handlers:
  - name: restart mysql
    service:
      name: mariadb
      state: restarted

  - name: restart apache
    service:
      name: httpd
      state: restarted
```
另外，需要注意：
- 事物处理器命名必须全局唯一，出现重名的情况，也仅有一个有效
- 事物处理器只有在任务真实执行的情况下才会被唤醒
- 事物被唤醒后，总是在该play所有任务都执行完成后按照在handlers处的定义顺序去顺序执行，而并不取决于其唤醒顺序
- 事物被多次唤醒，也仅执行一次

