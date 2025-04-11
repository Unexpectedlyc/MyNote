### 远程服务器方案 (jetbrains gateway)

> 参见CLion官方文档：[使用gateway进行远程控制](https://www.jetbrains.com/help/clion/remote.html)

这种方案配置起来最简单。

> 本章开头提到：gateway 会在远程机安装一个对应的IDE后台程序，并在本地启动一个瘦客户端(Jetbrains Client)，瘦客户端和远程机的IDE后台建立连接实现远程开发。gateway既可作为一个独立的启动器安装，也默认捆绑在jetbrains的IDE中。

我们只需要打开Gateway，或者打开CLion的欢迎界面，又或点击CLion`菜单栏 -> File -> Remote Development...`。
选到`Remote Development -> New Connection`，然后配置SSH连接。
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/f248d77540ad1ca1a9399d9c783ebe13.png)
接着选择要安装的IDE后台程序，以及项目目录。gateway就会自动将IDE后台程序安装到远程机，然后打开项目。
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/a92e8188738be67e0ccb3a3d8377493b.png)

> [gateway](https://www.jetbrains.com/help/clion/remote-development-overview.html#e4caca2f)需要和远程服务器建立端口转发。如果提示`The SSH port forwarding check failed`，可能是因为远程机器禁止了端口转发。编辑`/etc/ssh/sshd_config`，将`AllowTcpForwarding no`改为`AllowTcpForwarding yes`。然后重启`sshd`服务再次尝试即可。

欢迎界面-`Remote Development / SSH`中会显示远程项目列表，此后可以通过这里打开项目。