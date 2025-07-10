# 安装Docker容器

## 一、[Ubuntu 22.04](https://zhida.zhihu.com/search?content_id=236517538&content_type=Article&match_order=1&q=Ubuntu+22.04&zhida_source=entity)安装Docker

这里的Docker安装方法主要参考Docker官网的安装指南[Install Docker Engine on Ubuntu | Docker Docs](https://link.zhihu.com/?target=https%3A//docs.docker.com/engine/install/ubuntu/)。也有网上教程说只需要sudo apt install docker.io一条命令就可以安装，但是docker.io属于比较旧的Docker版本，如果要安装新版本的Docker，还是需要下面多个步骤。

### **1. 删除旧的Docker版本**

```bash
sudo apt remove docker docker-engine docker.io containerd runc
```

新安装的Ubuntu系统可以不执行这一步骤

### **2. 更新apt软件包列表**

```text
sudo apt update
```

### **3. 安装必要的依赖**

```text
sudo apt install apt-transport-https ca-certificates curl software-properties-common gnupg lsb-release
```

### **4. 添加Docker的官方GPG密钥，以验证 Docker 软件包的真实性**

```text
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
```

### **5. 把Docker软件源添加到系统的APT软件源列表中**

```text
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

如果是在大陆，这里的软件源可以换成阿里云或163的软件源来提高下载速度。但是我个人感觉使用官方的软件源也很快，几乎没有什么影响。

### **6. 再次更新apt软件包列表**

```text
sudo apt update
```

### **7. 安装最新的社区版本Docker CE**

```text
sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

上面是官网推荐的安装命令，其实最后两个Docker插件不安装也行。另外，也可以执行命令apt-cache madison docker-ce查询可安装的Docker版本，然后选择所需的版本进行安装（只要把docker-ce改成docker-ce=xxx即可，其中xxx是查到的Docker版本号，docker-ce-cli也是同样处理）。

### **8. 查看Docker是否已在运行**

```text
sudo systemctl status docker
```

![img](https://pic3.zhimg.com/v2-9cad8d9dd2caca76def26cd609b12176_1440w.jpg)

一般Docker安装之后就已经在运行了，如果没有，就执行下面命令来运行Docker：

```text
sudo systemctl start docker
```

### **9. 查看Docker版本号以进一步验证Docker是否安装成功**

```text
sudo docker --version
```

![img](https://pic1.zhimg.com/v2-1d86a7c8c78b2415af4d7ba401bc7644_1440w.jpg)

### **10. 再次更新apt软件包列表**

```text
sudo apt update
```

### 11.运行hello-world镜像来继续验证Docker是否安装成功

```text
sudo docker run hello-world
```

![img](https://pic3.zhimg.com/v2-c56b58a2331a1b2f5fa3cb79b556eac8_1440w.jpg)

上图中红框内的文字表示Docker已安装成功。

### **12. 将Docker服务设置为开机自启动**

```text
sudo systemctl enable docker
```

![img](https://pic2.zhimg.com/v2-06404490ef1ffe46a5662c08f446d1bf_1440w.jpg)

### **13. 添加当前用户到Docker用户组**

```text
sudo usermod -aG docker ${USER}
```

添加到Docker用户组可以避免每次使用Docker命令都要加sudo（仅添加受信任的用户，不建议使用）。

> [!IMPORTANT]
>
> 1.docker配置镜像Docker pull时报错：https://registry-1.docker.io/v2/
>
> ```text
> docker run hello-world时报错
> Unable to find image 'hello-world:latest' locally
> docker: Error response from daemon: Get "https://registry-1.docker.io/v2/": read tcp 192.168.30.91:51536->44.208.254.194:443: read: connection reset by peer.
> ```
>
> 这个错误表明Docker客户端尝试访问[Docker Hub](https://zhida.zhihu.com/search?content_id=253867805&content_type=Article&match_order=1&q=Docker+Hub&zhida_source=entity)或其他[Docker注册中心](https://zhida.zhihu.com/search?content_id=253867805&content_type=Article&match_order=1&q=Docker注册中心&zhida_source=entity)时出现了问题。具体来说，是在尝试获取注册中心API的响应时遇到了错误。可能的原因包括网络问题、认证问题、注册中心URL不正确或者注册中心服务本身不可用。
>
> 
>
> 2、解决方法
>
> ```text
> systemctl status docker
> sudo mkdir -p /etc/docker
> vim /etc/docker/daemon.json  
> ```
>
> 添加：
>
> ```text
> {
>   "registry-mirrors" : ["https://docker.registry.cyou",
> "https://docker-cf.registry.cyou",
> "https://dockercf.jsdelivr.fyi",
> "https://docker.jsdelivr.fyi",
> "https://dockertest.jsdelivr.fyi",
> "https://mirror.aliyuncs.com",
> "https://dockerproxy.com",
> "https://mirror.baidubce.com",
> "https://docker.m.daocloud.io",
> "https://docker.nju.edu.cn",
> "https://docker.mirrors.sjtug.sjtu.edu.cn",
> "https://docker.mirrors.ustc.edu.cn",
> "https://mirror.iscas.ac.cn",
> "https://docker.rainbond.cc",
> "https://do.nark.eu.org",
> "https://dc.j8.work",
> "https://dockerproxy.com",
> "https://gst6rzl9.mirror.aliyuncs.com",
> "https://registry.docker-cn.com",
> "http://hub-mirror.c.163.com",
> "http://mirrors.ustc.edu.cn/",
> "https://mirrors.tuna.tsinghua.edu.cn/",
> "http://mirrors.sohu.com/" 
> ],
>  "insecure-registries" : [
>     "registry.docker-cn.com",
>     "docker.mirrors.ustc.edu.cn"
>     ],
> "debug": true,
> "experimental": false
> }
> ```
>
> 重载和重启dockers服务
>
> ```text
> sudo systemctl daemon-reload
> sudo systemctl restart docker
> docker info
> ```
>
> 重新执行
>
> ```text
> docker run hello-world
> ```

![img](https://pic1.zhimg.com/v2-91e2843593bdf43fbf4b29c6878baa80_1440w.jpg)

## **二、Ubuntu 22.04安装NVIDIA Container Toolkit**

早期的Docker版本不支持GPU，需要额外安装NVIDIA-docker。在Docker 19.03之后，Docker开始支持GPU，已不再需要安装NVIDIA-docker，但还是需要安装NVIDIA Container Toolkit来更好地使用和管理GPU资源。

这里的安装方法主要参考NVIDIA官网的安装指南[Installing the NVIDIA Container Toolkit — container-toolkit 1.14.1 documentation](https://link.zhihu.com/?target=https%3A//docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/install-guide.html)。

### **1. 添加NVIDIA Container Toolkit的官方GPG密钥**

```text
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
```

### **2. 添加NVIDIA Container Toolkit的APT软件源**

```text
curl -s -L https://nvidia.github.io/libnvidia-container/stable/deb/nvidia-container-toolkit.list | sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' | sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list
```

如果是在大陆，这里的软件源可以换成阿里云或163的软件源来提高下载速度。

### **3. 更新apt软件包列表**

```text
sudo apt update
```

### **4. 安装最新的NVIDIA Container Toolkit**

```text
sudo apt install nvidia-container-toolkit
```

![img](https://pic2.zhimg.com/v2-48914ff077e333e4de1f9b5ee6a41d79_1440w.jpg)

### **5. 设置Docker默认使用NVIDIA runtime**

```text
sudo nvidia-ctk runtime configure --runtime=docker
```

![img](https://picx.zhimg.com/v2-d73d2b6eb53f9f5ec33d06c0931cec99_1440w.jpg)

### **6. 重启Docker**

```text
sudo systemctl restart docker
```

### **7. 验证NVIDIA Container Toolkit是否安装成功**

在NVIDIA的Docker Hub页面[cuda · GitLab](https://link.zhihu.com/?target=https%3A//gitlab.com/nvidia/container-images/cuda/blob/master/doc/supported-tags.md)挑一个CUDA镜像来运行，命令如下：

```text
sudo docker run --rm --runtime=nvidia --gpus all nvidia/cuda:11.8.0-cudnn8-devel-ubuntu22.04 nvidia-smi
```

上述命令中，nvidia/cuda:11.8.0-cudnn8-devel-ubuntu22.04是一个包含了CUDA、cuDNN及其所有依赖的镜像，将近10GB（下载时间大致10分钟，也可以选择一个小一点的镜像）。如果没有提前pull下来的话，会先下载再生成容器。

![img](https://pic2.zhimg.com/v2-2252ea1786be5ca9c37b7f27e0cce365_1440w.jpg)