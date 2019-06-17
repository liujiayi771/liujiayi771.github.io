---
title: ansible教程之file模块的使用
date: 2018-11-13 09:30:07
tags: Linux
---
ansible是一个自动化运维非常重要的工具，对于做大数据开发的人来说，ansible是一个管理集群的利器，我们可以用它来在集群中批量执行一些指令，而不需要对服务器一台一台进行操作。ansible的功能非常多，但是对于我来说一般只用来在集群中批量执行一些命令，例如批量安装软件、批量删除文件、批量新建文件夹等，具体详细的功能可以参考这个中文文档 [ansible中文权威指南](http://www.ansible.com.cn/docs/)。

### 用ansible对集群中文件进行管理
有时候我们可能需要在集群中批量删除某些文件，例如有时候spark slave节点上的work文件夹中的无用文件太多，又没有被自动清除，我们可以使用如下ansible指令来对文件夹中的文件进行批量删除：

```linux
ansible spark -m shell -a "rm -rf /usr/local/spark/work/*"
```
这样做确实能正确将work文件夹中的内容都删除掉，但是会收到一个警告信息：

```linux
[WARNING]: Consider using the file module with state=absent rather than running rm.  If you need to use command because file is insufficient you can add warn=False to this command task or set command_warnings=False in ansible.cfg to get
rid of this message.
```
其实，这里`-m`后面接的参数就是使用的module，这里用了ansible的shell模块在每台机器上运行shell command进行删除操作，warning中也告诉了我们，ansible中有一个file模块可以用来做文件处理的操作。

### file模块
file模块的功能非常多，官方文档是这样描述的：

```linux
Sets attributes of files, symlinks, and directories, or removes files/symlinks/directories. Many other modules support the same options as the `file' module - including [copy], [template], and [assemble].
```
我们可以使用`ansible-doc file`来查看file模块的详细使用文档（精简版可以使用`ansible-doc -s file`），这里举几个常用的例子来介绍file模块的使用。

#### file模块删除文件或目录
正如前面所说的删除文件的方法，用file模块的使用方法是：

```linux
$ ansible spark -m file -a "path=/usr/local/spark/work/* state=absent"
dell-r730-2 | SUCCESS => {
    "changed": false,
    "path": "/usr/local/spark/work/*",
    "state": "absent"
}
dell-r730-4 | SUCCESS => {
    "changed": false,
    "path": "/usr/local/spark/work/*",
    "state": "absent"
}
dell-r730-3 | SUCCESS => {
    "changed": false,
    "path": "/usr/local/spark/work/*",
    "state": "absent"
}
dell-r730-1 | SUCCESS => {
    "changed": false,
    "path": "/usr/local/spark/work/*",
    "state": "absent"
}
```
这样就能达到将`/usr/local/spark/work`目录中的`app-xxx`目录删除的目的，这里`-m`参数后面指定了使用file模块，`-a`参数后面为模块的参数，指定了文件或目录的路径`path`，以及状态`state=absent`，表示将指定的path进行删除处理。

#### file模块新建一个文件或目录

```linux
ansible spark -m file -a "path=/usr/local/spark/conf/slaves state=touch"
```
这里`state=touch`表示新建文件，而`state=directory`则表示新建目录。这里也可以是用`mode`来指定创建的文件或目录的权限`mode='u=rw,g=r,o=r'`或者`mode=0755`

#### file模块递归设置文件的属主或属组

```linux
ansible spark -m file -a "path=/usr/local/spark owner=spark group=spark recurse=yes"
```
`owner`设置属主，`group`设置属组，`recurse`设置是否对目录进行递归进行设置。

