# 【解决方法】libGL.so.1: cannot open shared object file: No such file or directory

```
File "/usr/local/lib/python3.6/dist-packages/cv2/__init__.py", line 8, in <module>
    from .cv2 import *
ImportError: libGL.so.1: cannot open shared object file: No such file or directory
```

**解决方法
安装这个库即可**

```
pip install opencv-python-headless
```

亲测有效

在Stack Overflow上有其他回答，当我试了无效
这边也提供给大家

1、在docker中出错

将以下行添加到您的 Dockerfile：

```
RUN apt-get update
RUN apt-get install ffmpeg libsm6 libxext6  -y
```

这些命令安装通常存在于本地计算机上的 cv2 依赖项，但可能会在您的 Docker 容器中丢失，从而导致问题。

2、libGL.so.1由 package 提供libgl1，所以添加一下代码

```
apt-get update && apt-get install libgl1
```

3、如果您使用的是 CentOS、[RHEL](https://so.csdn.net/so/search?q=RHEL&spm=1001.2101.3001.7020)、Fedora 或其他使用 的 linux 发行版yum，您将需要：

```
sudo yum install mesa-libGL -y

```

