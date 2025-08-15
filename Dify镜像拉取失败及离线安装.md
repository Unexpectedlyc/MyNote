# Dify镜像拉取失败及离线安装

- **Dify**的Docker**镜像拉取**失败
- Docker**离线部署**Dify详细教程

## 一、Dify的Docker镜像拉取失败

### 1. Dify拉取失败原因

Dify镜像拉取命令，通过这个命令Docker会拉取Dify需要的所有镜像，具体和配置有关，比如数据库的选择等等，大概要拉取10个左右的镜像。

```
docker-compose up -d
```

当我们直接执行这个命令的时候，可能会报下面的错误：

![img](https://p3-sign.toutiaoimg.com/tos-cn-i-6w9my0ksvp/6640e0d43dd845daa56a0ae30b0fdd2b~tplv-tt-origin-web:gif.jpeg?_iz=58558&from=article.pc_detail&lk3s=953192f4&x-expires=1755833708&x-signature=5TIfAqrzaRRHSNQxPWcWmShg48E%3D)

报错信息：

```
 ✘ worker Error Get "https://registry-1.docker.io/v2/": net/http: request canceled while...              18.4s 
 ✘ api Error    context canceled                                                                         18.4s 
 ✘ web Error    Get "https://registry-1.docker.io/v2/": net/http: request canceled while wa...           18.4s 
Error response from daemon: Get "https://registry-1.docker.io/v2/": net/http: request canceled while waiting for connection (Client.Timeout exceeded while awaiting headers)
```

这说明我们的无法访问
https://registry-1.docker.io/v2/这个镜像的源，这时候我们需要修改Docker的镜像源地址，使用**国内的镜像源来替代**。

### 2. Docker镜像拉取失败解决方法

因为有的源也不一定行，所以我们可以使用多个源，这些是**亲测目前可以使用**的。

```
"registry-mirrors": [
    "https://docker.1panel.live",
    "https://docker.nju.edu.cn",
    "https://docker.m.daocloud.io",
    "https://dockerproxy.com",
    "https://hub-mirror.c.163.com",
    "https://docker.mirrors.ustc.edu.cn",
    "https://registry.docker-cn.com",
    "https://registry.cn-hangzhou.aliyuncs.com"
  ]
```

添加方法，不同系统的添加方法不一样。

#### 2.1 Linux系统(如Ubuntu)的解决步骤

##### 2.1.1 创建一个daemon.json文件

这个文件是docker的一个附加的配置文件，不是系统配置文件，一般临时修改都是修改这个文件即可。

Docker初次安装后是没有daemon.json的，需要自己创建。

```
配置文件的默认路径：
gedit /etc/docker/daemon.json
```

##### 2.1.2 配置源地址

就是我们上面的那些地址，直接把下面的内容复制到文件里面，保存即可。

```
{
 #镜像源管理
"registry-mirrors": [
    "https://docker.1panel.live",
    "https://docker.nju.edu.cn",
    "https://docker.m.daocloud.io",
    "https://dockerproxy.com",
    "https://hub-mirror.c.163.com",
    "https://docker.mirrors.ustc.edu.cn",
    "https://registry.docker-cn.com",
    "https://registry.cn-hangzhou.aliyuncs.com"
  ]
}
```

##### 2.1.3 重启生效

创建并修改完daemon.json文件后，需要reload让这个文件生效，并且重启docker服务

```
#systemctl daemon-reload
#systemctl restart docker
```

#### 2.2 Windows的解决步骤

windows系统的修改和Ubuntu的修改类似，只是安装路径不同，找到对应的daemon.json，创建修改即可。

#### 2.3 Mac的解决步骤

Mac的配置也非常简单，打开DockerAPP，点击配置按钮。

![img](https://p3-sign.toutiaoimg.com/tos-cn-i-6w9my0ksvp/68041e00821848eba3f6a8bfb045761a~tplv-tt-origin-web:gif.jpeg?_iz=58558&from=article.pc_detail&lk3s=953192f4&x-expires=1755833708&x-signature=Eek4WKes17Bv6L1nS49XKC5PPhA%3D)

按照图示，把上面的地址贴到里面就可以了。

![img](https://p3-sign.toutiaoimg.com/tos-cn-i-6w9my0ksvp/ef0da0c21fb94d95a06439854ee48ac6~tplv-tt-origin-web:gif.jpeg?_iz=58558&from=article.pc_detail&lk3s=953192f4&x-expires=1755833708&x-signature=oVZPbfy0yZ1kSS5S%2BY6pQl0g%2F%2BU%3D)

这时Docker会重启，我们再拉取一下，效果：

![img](https://p3-sign.toutiaoimg.com/tos-cn-i-6w9my0ksvp/df57e7496d274687a080c5c5b960d788~tplv-tt-origin-web:gif.jpeg?_iz=58558&from=article.pc_detail&lk3s=953192f4&x-expires=1755833708&x-signature=5czZsN%2F4SHDBDZUTK0b4nzGeZ2E%3D)

这样就成功拉取了Dify的镜像。

## 二、Docker离线部署Dify详细教程

### 1. 准备工作

- **在线环境**：一台可以访问互联网的机器，安装好Docker和Docker Compose。
- **离线环境**：目标部署机器，安装好Docker和Docker Compose。
- **存储硬盘**：一个足够大的移动硬盘或U盘，用于传输镜像和文件。

### 2. 在线环境操作

#### 2.1 拉取Dify镜像

在在线环境中，使用以下命令拉取Dify镜像：

```
docker-compose up -d
```

#### 2.2 保存镜像

将拉取的镜像保存为tar文件：

```
docker save -o myimage.tar myimage:tag
```

这里，myimage.tar是你想要创建的tar文件的名字，myimage:tag是你想要保存的镜像的名称和标签。

![img](https://p3-sign.toutiaoimg.com/tos-cn-i-6w9my0ksvp/b336ea11d8754c018f71b63a5d32a3a6~tplv-tt-origin-web:gif.jpeg?_iz=58558&from=article.pc_detail&lk3s=953192f4&x-expires=1755833708&x-signature=NjvsVjgjtTjAFhwJh6SLvI9e0oA%3D)

例如，如果你想保存一个名为langgenius/dify-web、标签为0.15.2的镜像，

```
docker save -o dify_web.tar langgenius/dify-web:0.15.2
```

当然也可以同时**打包多个镜像**包。

例如，如果你想保存一个名为langgenius/dify-web、标签为0.15.2的镜像和名为langgenius/dify-sandbox、标签为0.2.10的镜像

```
docker save -o dify_all.tar langgenius/dify-web:0.15.2 langgenius/dify-sandbox:0.2.10
```

这样可以把所有Dify依赖的镜像打到一个**dify_all.tar**的包里面。

#### 2.3 下载Dify的Docker Compose文件

从Dify的官方GitHub仓库下载docker-compose.yml文件：

```
git clone https://github.com/langgenius/dify.git
在docker目录下
```

#### 2.4 复制离线安装文件

将保存的镜像文件dify_all.tar和docker-compose.yml文件复制到移动硬盘中。

### 3. 离线环境操作

#### 3.1 导入镜像

将dify_all.tar文件复制到离线环境的机器上，然后使用以下命令导入镜像：

```
docker load -i dify_all.tar
```

#### 3.2 部署Dify

将docker-compose.yml文件复制到离线环境的机器上，然后使用以下命令启动Dify：

```
docker-compose up -d
```

这个命令会根据docker-compose.yml文件中的配置启动Dify相关的容器。

#### 3.3 验证部署

使用以下命令检查Dify容器是否正常运行：

```
#查看Docker 镜像
docker images

#查看容器运行情况
docker ps
```