# centos7创建桌面快捷方式

以pycharm为例子

## 1.创建pycharm.desktop

```bash
cd /usr/share/applications/
gedit pycharm.desktop

```

内容如下：

```bash
#!/usr/bin/env xdg-open
[Desktop Entry]
Encoding=UTF-8
Name=Pycharm
Comment=pycharm-community-2019.2.3
Exec=/usr/local/pycharm-community-2019.2.3/bin/pycharm.sh
Icon=/usr/local/pycharm-community-2019.2.3/bin/pycharm.png
Terminal=false
StartupNotify=true
Type=Application
Categories=Application;

```

## 2.复制到用户目录下

```bash
chmod 777 pycharm.desktop
cp pycharm.desktop /home/marvin/Desktop/

cd /home/marvin/Desktop
chmod 777 pycharm.desktop
chown marvin:marvin pycharm.desktop

```

## 3.运行

右键点击图标，然后open，再点击信任和运行。

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/6d9160b641a80916f98ea8665abc61c4.png)