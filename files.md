## Files模块
Ansible有大量的模块。Files模块提供了绝大多数操作文件系统的功能，例如创建，复制，编辑，编辑权限和文件的其他特定等。常见的Files的模块有：
|模块|作用|
|--|--|
|blockinfile|插入，更新，删除符合标准的多行|
|copy|从本地或者远程机拷贝文件到被管理机，并设定文件的属性和安全权限|
|fetch|和copy功能相似，但是从远端拷到本地|
|file|配置文件的权限，属性，所有者等一列信息|
|lineinfile|操作单一文件中的单行，确保文件中含有某行，或者对某行进行内容进行替换|
|stat|有点类似于Linux中的stat，用于获取文件信息|
|synchronize|类似于rsync但权限没有rsync多|
## Files模块常见的应用场景
- 被管理机器上创建文件
    ```
    ---
    - name: Touch one file and set permission
      file:
        path: /path/to/file
        owner: user1
        group: group1
        mode: 0640
        state: touch
    ```
- 编辑文件属性
  ```
  ---
  - name: SELinux type is set to samba_share_t
    file:
      path: /path/to/file
      setype: samba_share_t
  ```
- 复制和编辑被管理机器上的文件
  ```
  ---
  - name: Copy a file to a file
    copy:
      src: file
      dest: /path/to/file
  ```
  该task会将文件从管理结点拷到被管理结点。被管理结点的文件默认会被覆盖， 因为，file模块会使用默认选项‘force: yes’, 如果不希望默认覆盖文件，则将其矫正为'force: no'。
- 确保被管理机文件中有某一行
  ```
  - name: Add a line of Text of a file
    lineinfile:
      path: /path/to/file
      line: "Add the line to one file"
      state: present
  ```
- 确保被管理机文件中有某一段
  ```
  - name: Add a block to a file
     blockinfile:
       path: /path/to/file
       block: |
         First line in the additional file
         Second line in the additional file
  ```
- 从被管理机上删除文件
  ```
  - name: Make sure a file does not exist on managed host
    file:
      path: /path/to/file
      state: absent
  ```
- 获取被管理机上文件的状态
  ```
  - name: Verify the checksum of a file
    stat:
      path: /path/to/file
      checksum_algorithm: md5
    register: result
    
    debug:
      msg: "The checksum of the file is {{ result.stat.checksum }}"
  ```
- 同步管理结点和被管理结点上的文件
  ```
  - name: Synchronize the files to remote host
    synchronzie:
      src: file
      dest: /path/to/file
  ```
