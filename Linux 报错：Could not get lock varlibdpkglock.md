# Linux 报错：Could not get lock /var/lib/dpkg/lock 

## Could not get lock /var/lib/dpkg/lock

  有朋友在使用[Ubuntu](https://so.csdn.net/so/search?q=Ubuntu&spm=1001.2101.3001.7020) Linux的apt 包管理器更新或安装软件时，可能会遇到过诸如以下：

```
E: Could not get lock /var/lib/dpkg/lock-frontend - open (11: Resource temporarily unavailable)
E: Unable to acquire the dpkg frontend lock (/var/lib/dpkg/lock-frontend), is another process using it?
```

或者：

```
E: Could not get lock /var/lib/dpkg/lock-frontend - open (11: Resource temporarily unavailable)
E: Unable to acquire the dpkg frontend lock (/var/lib/dpkg/lock-frontend), is another process using it?
```

再或者：

```
E: Could not get lock /var/lib/apt/lists/lock – open (11: Resource temporarily unavailable)
E: Unable to lock directory /var/lib/apt/lists/
```

等这样的报错信息。

  这种时候做为小白的你或许会有些不知所措，请冷静不必惊慌，这些错误提示大多是因为某些程序在系统后台进行着某些 apt 操作，因此锁定了 apt 数据库，所以暂时不能进行 apt 操作。就像windows上某程序或文件被另一进程所占有时，其他进程也无法访问一样，这是符合设计逻辑的。

#### 解决方案

  那遇到这种情况，一般我们只需要安静地等待几分钟，或者先去做其他的事情，比如打把落地成盒或者王者农药，直到当前的更新、安装或卸载任务完成后，锁就会自动释放，然后就可以进行 apt 操作了。

  当然了，上面说的是正常情况下的对应，那非正常情况下，比方说你等了好多个几分钟锁都还没有被释放，你就要看看是不是该进程由于某些原因而卡住了并且一直占用着锁。如果是的话，那你只能干掉这个进程，然后删除该锁定了。

  首先，我们 ***先找出是哪个进程占用了锁文件 /var/lib/dpkg/lock***

```
$ sudo lsof /var/lib/dpkg/lock
```

其他锁文件对应的命令

```
$ sudo lsof /var/lib/dpkg/lock-frontend
$ sudo lsof /var/lib/apt/lists/lock
```

然后得到输出结果：

```
COMMAND PID USER FD TYPE DEVICE SIZE/OFF NODE NAME
unattende 1548 root 6uW REG 8,2 0 1181062 /var/lib/dpkg/lock
```

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/0e17ba9e3e5b6f34414fbde1e5480f22.png#pic_center)

 我们可以从结果中看到，该进程的 PID 为 32321

  接着，kill 掉这个进程

```
$ sudo kill -9 32321
```

然后你就可以放心地删除锁文件

```
$ sudo rm /var/lib/dpkg/lock
```

或者

```
$ sudo rm /var/lib/dpkg/lock-frontend
$ sudo rm /var/lib/apt/lists/lock
```

如果需要，还可以删除缓存目录下的锁文件

```
$ sudo rm /var/cache/apt/archives/lock
```

做完上面的步骤后，记得要运行以下命令

```
$ sudo dpkg --configure -a

```

###### 这样问题一般是被解决了，如果你还没能解决的话，请查阅更多资料。