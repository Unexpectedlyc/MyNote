---

---

#  配置深度学习环境: Docker + Conda + Pytorch + SSH + VS Code



> [!NOTE]
>
> 面向多用户的深度学习环境主要满足以下需求：
>
> 1. **宿主机隔离**：用户处于虚拟环境中，拥有root权限，但无法访问宿主机。
> 2. **用户隔离**：多个用户共同使用宿主机的计算资源，互不影响。
> 3. **开发环境隔离**：同一个用户可能需要多个开发环境（例如不同的深度学习框架、同一深度学习框架的不同版本），开发环境之间互不影响。
> 4. **远程访问**：用户可以通过多种客户端远程连接服务器。
> 5. **远程开发**：用户通过本地IDE进行编程，而代码和编译器位于远程服务器。
>
> 对以上需求具体化，就是每个用户以非root身份通过SSH远程访问服务器上的Docker容器。Docker容器提前安装好基础环境，包括特定版本的操作系统、[CUDA](https://zhida.zhihu.com/search?content_id=236521344&content_type=Article&match_order=1&q=CUDA&zhida_source=entity)、[CuDNN](https://zhida.zhihu.com/search?content_id=236521344&content_type=Article&match_order=1&q=CuDNN&zhida_source=entity)、常用软件。用户可以自定义容器内操作系统的用户名和密码（在运行Docker容器时设置好）。以上步骤由管理员操作，Docker容器运行起来后，用户根据个人需要设置具体的环境细节和以root身份安装所需的软件，并使用[VS Code](https://zhida.zhihu.com/search?content_id=236521344&content_type=Article&match_order=1&q=VS+Code&zhida_source=entity)连接服务器进行开发。
>
> 根据上述需求，本节内容包括创建Docker镜像、启动Docker容器、配置Conda环境、安装PyTorch和Python package、配置VS Code，其中前两部分内容需要用到Dockerfile、build_image.sh、init_container.sh、run_container.sh四个文件（四个文件需要放到同一个目录下），SSH也包含在前两部分内容之中。



## **一、创建Docker镜像**

### **1. 建立Dockerfile文件**

Docker镜像的获取有三种方式，第一种方式是在第三方网站上（例如Docker Hub）下载现成的Docker镜像，但是这样的Docker镜像可能不符合用户的具体要求。第二种方式是在第一种方式的基础上运行Docker容器，然后对容器内的软件按需求进行安装、卸载和对环境做相应的配置，再用docker commit命令保存成新的Docker镜像。这种方式的缺点是镜像较大，不利于传播，且轻微改动就需要另存一个镜像版本。第三种方式是编译Dockerfile文件得到Docker镜像，好处是可以从头高度定制Docker镜像，具有自动化和可重复性，可维护性强，功能一目了然，易于共享。缺点是需要掌握Dockerfile的语法，构建镜像的时间较长（其实比下载镜像慢不了多少）。

我采用第三种方式，Dockerfile文件的内容如下：

```basemake
FROM nvidia/cuda:11.8.0-cudnn8-devel-ubuntu22.04

MAINTAINER xxx xxx <xxx@xxx.edu.cn>

USER root

# Install some essential tools
RUN apt update && apt install -y \
  sudo \
  wget \
  curl \
  vim \
  git \
  build-essential \
  cmake \
  checkinstall \
  pkg-config \
  openssh-server \
  openssh-client \
  iputils-ping \
  zip \
  unzip \
  p7zip \
  tmux \
  htop \
  iftop \
  iotop \
  libxmu6 \
  libsm6 \
  libxext-dev \
  libxrender1 \
  libgl1-mesa-glx

# Install Miniconda
RUN wget https://repo.continuum.io/miniconda/Miniconda3-latest-Linux-x86_64.sh -O Miniconda.sh && \
  /bin/bash Miniconda.sh -b -p /opt/conda && \
  rm Miniconda.sh
ENV PATH /opt/conda/bin:$PATH

# Create the directories that will be used in the docker run command to mount the volumes on the host machine
RUN mkdir /home/data /home/share

# SSH settings
RUN mkdir /var/run/sshd
# Replace the "session required pam_loginuid.so" in /etc/pam.d/sshd with "session optional pam_loginuid.so"
# But it seems that this is not needed in the new versions of docker. See https://gitlab.com/gitlab-org/gitlab-foss/-/issues/3027
# SSH login fix. Otherwise user is kicked off after login
RUN sed 's@session\s*required\s*pam_loginuid.so@session optional pam_loginuid.so@g' -i /etc/pam.d/sshd
ENV NOTVISIBLE "in users profile"
RUN echo "export VISIBLE=now" >> /etc/profile
EXPOSE 22

# Add cuda and conda paths
# RUN echo "export PATH=/usr/local/cuda/bin:/usr/local/nvidia/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin" >> /etc/profile
RUN echo "export PATH=/usr/local/cuda/bin:/usr/local/nvidia/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/opt/conda/bin" >> /etc/profile
RUN echo "export LD_LIBRARY_PATH=/usr/local/cuda/lib64:/usr/local/nvidia/lib64:/usr/lib64:/usr/local/lib:/usr/lib:/usr/lib/x86_64-linux-gnu" >> /etc/profile

# Clean up all temp files
RUN apt clean && rm -rf /tmp/* /var/tmp/* /var/lib/apt/lists/* /var/cache/apt/*

# Set the entrypoint
COPY ./init_container.sh /usr/local/bin/init_container.sh
RUN chmod +x /usr/local/bin/init_container.sh
ENTRYPOINT ["/usr/local/bin/init_container.sh"]
```

这个Dockerfile主要做七件事。第一是选择基础镜像，即首行的FROM语句。基础镜像可以在[hub.docker.com](https://link.zhihu.com/?target=https%3A//hub.docker.com/r/nvidia/cuda/tags)上搜索得到，如下图所示，这里需要考虑Ubuntu、CUDA、CuDNN的版本号和镜像类别。其中Ubuntu版本号影响不大，最近几年的（1804/2004/2204）都可以；CUDA、CuDNN的版本号跟GPU型号、深度学习框架版本有关，例如RTX 4090只支持11.8及以上的CUDA，高版本的PyTorch也只支持高版本的CUDA；镜像类别则分为base、runtime、devel，简单来说，它们三个预装的库依次增多，但镜像文件的体积也依次变大。我选择的基础镜像是nvidia/cuda:11.8.0-cudnn8-devel-ubuntu22.04，各种版本号和镜像类型都包含在镜像名称和标签中。

![img](https://pic4.zhimg.com/v2-0bf36f0922be5a3f3b809ec6abcef08b_1440w.jpg)

第二是安装各种基础性工具软件以及软件包管理器Miniconda（即line 7-38）。

第三是提前建立几个目录（即line 40-41），方便统一管理各个容器内部与外部宿主机的交互情况，这在后面的docker run命令的--mount参数会有进一步的解释。

第四是配置SSH，即line 43-51。

第五是把CUDA和Conda的路径写入配置文件中（即line 53-56），这样非root用户可以直接正常使用，否则需要在容器运行之后每个用户自己再另行配置。

第六是删除各种中间文件和缓存文件（即line 58-59），节省镜像体积。

第七是把Docker容器的初始化文件init_container.sh拷贝到容器内，并在容器启动时运行（即line 61-64）。这个初始化文件的功能是根据容器启动时输入的用户名和密码来新建用户，并开启SSH服务。这里使用ENTRYPOINT而不是CMD，是因为后者会被Docker容器启动时的其它命令所覆盖，只能以后台方式启动容器。

### **2. 建立init_container.sh文件**

init_container.sh文件的内容如下：

```bash
#!/bin/bash

# Fail if NEW_USER or NEW_PWD is not given
if [ -z "$NEW_USER" ]; then
    echo 'ERROR: Missing -e NEW_USER="username" in docker run command.'
    exit 1
fi
if [ -z "$NEW_PWD" ]; then
    echo 'ERROR: Missing -e NEW_PWD="password" in docker run command.'
    exit 1
fi

# Create a sudo user account with a password and a /bin/bash shell
useradd -m -G sudo -s /bin/bash $NEW_USER &>/dev/null
echo "$NEW_USER:$NEW_PWD" | chpasswd

# Start SSH service
service ssh start

# Switch to the new user
su - $NEW_USER
```

值得注意的是，如果init_container.sh文件是在Windows系统上编辑的，一定要把回车换行符转换成Linux的格式（即CRLF → LF），否则在启动Docker容器时会出现如下错误：

exec /usr/local/bin/container_init.sh: no such file or directory

这需要提前执行如下命令转换回车换行符的格式：

```text
sed -i 's/\r//' container_init.sh
```

### **3. 生成Docker镜像**

新建一个脚本文件build_image.sh，文件内容如下：

```text
sudo docker build --no-cache -t nvdocker:v1 -f ./Dockerfile .
```

这里只有一条命令却仍然采用脚本文件的原因是易于溯源和重用。参数--no-cache是不使用缓存，这可以安装最新的软件，避免缓存导致的数据不安全或数据不一致问题，但是构建速度会慢一些。

执行命令bash build_image.sh，输出结果如下：

![img](https://picx.zhimg.com/v2-497f237a37c9256a82dd97916f0ac463_1440w.jpg)

### **4. 查看Docker镜像**

执行下面的命令以检测Docker镜像是否成功生成：

```text
sudo docker images
```

![img](https://pica.zhimg.com/v2-bf310064bbdff860841fcdd3becd2370_1440w.jpg)

## **二、启动Docker容器**

### **1. 准备Docker容器的启动参数**

新建一个脚本文件run_container.sh，文件内容如下：

```text
#!/bin/bash
c=1;
new_user=liming
new_pwd=cs123

mkdir -p /data/cont_space/"$new_user"_"$c"
sudo docker run -it \
    -h srv1 \
    -p "$((50401+c)):80" \
    -p "$((49448+10*c)):443" \
    -p "$((10000+c)):6006" \
    -p "$((20000+c)):6007"  \
    -p "$((32868+c)):22" \
    -p "$((40100+10*c)):8000" \
    -p "$((41200+10*c)):8080" \
    -p "$((42300+10*c)):8081" \
    -p "$((43400+10*c)):8082" \
    -p "$((2201+c)):$((2201+c))" \
    --gpus all \
    --name "$new_user"_"$c" \
    --ipc=host \
    --mount type=bind,source=/data/public_data,target=/home/data,readonly \
    --mount type=bind,source=/data/shared_data,target=/home/share \
    --mount type=bind,source=/data/cont_space/"$new_user"_"$c",target=/home/$new_user \
    -e NEW_USER=$new_user \
    -e NEW_PWD=$new_pwd \
    nvdocker:v1 /bin/bash
```

各个参数的介绍如下：

-it：交互式启动Docker容器，方便在第一时刻进入Docker容器查看运行情况，这个参数往往和最后的/bin/bash结合使用，即以bash命令行窗口进行交互。这也是Dockerfile的最后一行命令使用ENTRYPOINT而非CMD的原因。

-h：Docker容器内部的主机名

-p：端口映射。使用场景是用户通过SSH远程访问Docker容器，而容器内部的程序无法直接被主机上的其他应用访问，只有先将容器内部的端口与主机的端口进行映射才能实现这个目的。Ubuntu系统的端口号范围是0~65535，其中最重要的是专门用于SSH服务的22端口，其它的端口号可以选择性添加。由于端口号0~1023大都有一些固定用途，所以最好选择容器内部的较大的端口号与主机端口进行映射。此外，由于是多用户共同访问同一个服务器，为了使各个用户的容器端口不冲突，这里使用一个变量c作为区分每个用户的ID，每个端口在映射时都包含了c的运算，使得每个用户的每个端口都互不相同。

--gpus：指定使用的GPU设备。其中--gpus all表示该容器可以使用所有的GPU，其它参数如--gpus '"device=0,1"'表示该容器只使用前两个GPU（注意是单引号里面有双引号，否则出现错误）。这个参数可以为不同的用户分配不同的GPU，防止出现某个用户占用所有GPU的情况（赶ddl时经常发生）。

--name：Docker容器名称

--ipc：指定容器的IPC（Inter-Process Communication）命名空间，--ipc=host参数可以使容器与宿主机共享IPC命名空间。

--mount：挂载主机目录。由于Docker采用了文件系统隔离机制，Docker容器无法直接访问宿主机上的目录和文件，但是可以通过挂载主机目录的方式来实现。这里挂载了三个主机目录，其中/home/data专门用于访问宿主机上的公开数据集，因为数据集是所有用户共用的，不必每个用户都在自己的容器内部存放一份，另外设为只读访问； /home/share用于用户分享自己的数据到宿主机给其它用户访问；/home/$new_user是每个用户的工作目录，存放所有的个人数据，这是因为Docker容器中的文件的物理地址默认是系统盘，系统盘空间有限，这里把工作目录挂载到数据盘的指定目录下。注： -v参数也可实现挂载主机目录，--mount是新版本Docker增加的参数，功能更加强大。

-e：设置Docker容器内部的环境变量。由于容器启动时新建用户需要用到用户名和密码，这里把用户名和密码以环境变量的方式传递给容器的初始化文件init_container.sh。

### **2. 执行Docker容器的启动脚本**

```text
bash run_container.sh
```

此时已进入容器内部（见命令行左端的用户名和主机名），如下图所示：

![img](https://pic1.zhimg.com/v2-a18cf185a7e8c0b17956abecf1a05b08_1440w.jpg)

### **3. 检查Docker容器**

在Docker容器内部查看CUDA版本和SSH运行情况

```text
nvidia-smi
nvcc -V
ps aux | grep sshd
```

![img](https://pic2.zhimg.com/v2-06fa27078c1cacfecd34ddf31d025021_1440w.jpg)

先后按Ctrl+p和Ctrl+q，退出Docker容器。执行如下命令，查看容器状态

```text
sudo docker ps -a
```

![img](https://pic1.zhimg.com/v2-26ddabe2b8928fd5fb21b887f660fc12_1440w.jpg)

### **4. SSH连接Docker容器**

以上步骤都是通过SSH连接服务器的宿主机完成的，从这一步骤开始将通过SSH连接服务器上的Docker容器（查看命令行顶端用户名和主机名的变化）。首先用MobaXterm建立一个SSH Session，然后输入Docker容器IP（在这里也是服务器IP）、Docker容器内的用户名、端口号，发起连接，输入SSH密码（在这里也是Docker容器内的用户密码，这种方式比较方便，但是不如公钥登录SSH安全）

![img](https://picx.zhimg.com/v2-965277025533f78136f4d5179c0926c9_1440w.jpg)

## **三、配置Conda环境**

### **1. 检查当前Conda版本及环境**

```text
conda --version
conda info -e
```

![img](https://pica.zhimg.com/v2-2486420020226c9c0f9381f95524fe22_1440w.jpg)

### **2. 安装新的Conda环境**

确定要安装的PyTorch版本和Python版本，注意版本之间的兼容性。执行下面的命令安装带有Python 3.10的Conda环境（PyTorch的安装在后面介绍）：

```text
conda create -n py310pt113 python=3.10
```

![img](https://pic2.zhimg.com/v2-b443cea4718e064194ad8c4356cee4d7_1440w.jpg)

再次执行命令conda info -e，可以看到环境py310pt113已经安装好，如下图所示：

![img](https://pica.zhimg.com/v2-08887bc041d8c75848a886efbcc4ea2a_1440w.jpg)

### **3. 进入Conda环境**

执行下面的命令进入刚才新安装的Conda环境:

```text
conda activate py310pt113
```

这时会发现上述命令输出错误CommandNotFoundError，这是Conda环境还没有激活所致。解决方法是先执行下面的命令来激活Conda环境：

```text
source activate
```

这时会发现命令行窗口的liming@srv1变成了(base) liming@srv1，(base)表示这是Conda的基础环境或根环境。Conda的基础环境也可以进行程序开发，但不建议这样做，因为安装各种包会污染基础环境，导致版本冲突。再次执行命令conda activate py310pt113，可以发现命令行窗口的(base) liming@srv1又变成了(py310pt113) liming@srv1，这才是刚才安装新的Conda环境。整个过程如下图所示：

![img](https://pica.zhimg.com/v2-7765c630feea3bf53acaf53a7d9a25fe_1440w.jpg)

## **四、安装PyTorch和Python package**

### **1. 安装思路**

在Conda环境下安装PyTorch和其它的Python package本来是一件很简单的事情，但现实情况却有些复杂，因此在这里多花点笔墨来介绍。

目前常见的安装PyTorch和Python package的方式有两种：pip install和conda install。pip和Conda都是Python package的管理工具，用以解决环境内的package依赖，只不过Conda比pip进行更加严格的Python package依赖检查和管理。Conda还有一个pip不具备的优势是能够创建独立的Python环境，实现多个环境之间相互隔离（这也是在Docker镜像中预装Conda的原因）。注：pip既可以在Conda环境外部使用，也可以在Conda环境内部使用，下面涉及到的的pip install都是在Conda环境内部进行安装。

理想情况下，可以在Conda环境中使用conda install命令来安装PyTorch和所有的Python package。但是conda install也有一些缺点：package种类没有pip上的齐全，package更新速度没有pip上的及时，package下载速度和安装速度也没有pip上的快。尤其是package下载速度这一点，在国内尤其慢，这是在国内使用官方镜像源所致。许多教程都提到通过更换镜像源来解决conda install的这一问题，例如清华源、中科大源、阿里云源等。但是近年来清华、中科大的镜像源由于没有得到Annconda的授权而停止了服务，打开它们的网页可以看到有些镜像都很久没有更新；阿里云源似乎比较新，但跟清华源和中科大源一样，安装过程中要么输出错误CondaHTTPError，要么出现版本兼容问题。总之，目前国内常用的Conda镜像源很多都已失效，官方镜像源仍然可用但是下载速度较慢。

上述问题导致无法在Conda环境中完全使用conda install命令来安装PyTorch和所有的Python package，所以conda install和pip install混用是不可避免的。但是二者混用又容易引起 package发生冲突，其依赖关系不易解决。折中的解决方法是在Conda环境中尽量只使用一种安装方法，即要么大部分用conda install，少部分用pip install；要么仅深度学习框架用conda install（也可用pip install），其余全部都用pip install。

由于国内镜像源的问题，只能采用后一种情况，有两种安装方法如下：

1）Conda env + conda install PyTorch + pip install all Python packages

2）Conda env + pip install PyTorch and all Python packages

其中第一种方法是除了深度学习框架PyTorch用conda install安装以外，其它的所有Python package均用pip install来安装。第二种方法是如果连PyTorch都下载不了，就只能全部都用pip install来安装了。

### **2. 安装过程**

进入PyTorch官网[Previous PyTorch Versions | PyTorch](https://link.zhihu.com/?target=https%3A//pytorch.org/get-started/previous-versions/)，复制PyTorch 1.13.0的Conda和Wheel在Linux平台的安装指令如下所示：

```text
conda install pytorch==1.13.0 torchvision==0.14.0 torchaudio==0.13.0 pytorch-cuda=11.7 -c pytorch -c nvidia
```

或

```text
pip install torch==1.13.0+cu117 torchvision==0.14.0+cu117 torchaudio==0.13.0 --extra-index-url https://download.pytorch.org/whl/cu117
```

注意Docker镜像内安装的CUDA版本是11.8（为了后续安装PyTorch 2.0版本），而PyTorch 1.13.0最高只支持CUDA 11.7。由于CUDA是向下兼容，所以安装带有CUDA 11.7的PyTorch 1.13.0无妨。

先执行第一个conda install命令安装PyTorch，如果安装不成功则执行第二个pip install命令。其余Python package都采用pip install命令来安装，例如安装OpenCV的命令是

```text
pip install opencv-python
```

**3. 检查PyTorch安装**

进入Python环境，输入如下指令查看PyTorch安装是否成功，如下图所示：

![img](https://pic3.zhimg.com/v2-7a04182700f302a7f12a0076a74e8dc0_1440w.jpg)

## **五、配置VS Code**

Python是深度学习最流行的编程语言，最常用的Python IDE是PyCharm。然而PyCharm比较笨重，硬件资源占用多；功能齐全的专业版要想免费使用，要么需要安装破解版本，要么需要edu邮箱注册并每年续约，比较麻烦。因此，这里采用开源免费的VS Code作为深度学习开发的Python IDE。下面的VS Code配置过程均在Windows 10系统上完成。

### **1. 下载和安装VS Code**

打开VS Code官网[Download Visual Studio Code - Mac, Linux, Windows](https://link.zhihu.com/?target=https%3A//code.visualstudio.com/Download)，根据操作系统下载对应的VS Code版本。我的操作系统是64位的Win10，下载的是User Installer的x64版本，如下图所示。User Installer是把VS Code安装在当前用户目录下，System Installer是安装在公共目录下可供所有用户使用，.zip相当于不需要安装的绿色版本，CLI是用于服务器的命令行版本。个人电脑一般选择User Installer即可。

安装VS Code很简单，一路点击“下一步”完成安装。

![img](https://picx.zhimg.com/v2-3c876688c1e39d84fdfddf23d9e90575_1440w.jpg)

### **2. 安装VS Code插件**

使用VS Code远程连接服务器上的Docker进行Python编程，需要安装的基础VS Code插件有4个：Python extension、Pylance、Remote SSH、Dev Containers。在VS Code左边栏点击Extensions图标，在弹出的搜索框内输入上述插件名称，点击安装即可。插件安装好之后如下图所示：

![img](https://pica.zhimg.com/v2-d83cf4c12a53af0cfe34144fdf09a55e_1440w.jpg)

### **3. 安装OpenSSH**

VS Code需要通过SSH连接远程服务器来实现远程开发。首先以管理员权限打开Windows PowerShell，执行如下指令查看是否已安装OpenSSH。

```text
Get-WindowsCapability -Online | ? Name -like 'OpenSSH*'
```

注意：要使用管理员权限打开PowerShell，否则会输出下面的错误：

Get-WindowsCapability : The requested operation requires elevation.

如果OpenSSH未安装，则执行如下指令安装OpenSSH：

```text
Add-WindowsCapability -Online -Name OpenSSH.Client~~~~0.0.1.0
```

整个过程如下图所示：

![img](https://pic3.zhimg.com/v2-d2c956ab5ce427bc3ae715b0e6440646_1440w.jpg)

### **4. VS Code免密连接**

为了使VS Code连接服务器省去输入密码的步骤，需要配置VS Code的SSH密钥。

1）首先在PowerShell执行如下指令生成SSH密钥。

```text
ssh-keygen -t rsa
```

上述指令在C:\Users\WIN10 22H2\.ssh\用户目录下生成id_rsa和id_rsa.pub两个私钥和公钥文件，过程如下图所示：

![img](https://pica.zhimg.com/v2-19155cab860a6e50e1a40196d76d20dc_1440w.jpg)

2）使用如下命令复制公钥到远程服务器。

```text
scp -P 32869 ~/.ssh/id_rsa.pub liming@10.106.112.149:~/
```

3）在远程服务器上使用如下命令将公钥添加到authorized_keys（如果目录~/.ssh/ 不存在要先使用命令mkdir ~/.ssh/生成该目录）。

```text
cat ~/id_rsa.pub >> ~/.ssh/authorized_keys
```

### **5. 添加VS Code配置文件**

调试和运行Python程序需要先指定程序的入口文件、程序参数、调试方式等，在VS Code中需要手动添加配置文件。首先点击菜单栏Run中的Add Configuration…，会打开一个配置文件launch.json。一般情况下采用如下图所示的配置内容即可，如果有程序参数需要从外部输入，则添加args字段，另外VS Code官方文档[Launch.json attributes](https://link.zhihu.com/?target=https%3A//code.visualstudio.com/docs/editor/debugging%23_launchjson-attributes)详细介绍了launch.json中每个字段的含义。

![img](https://pica.zhimg.com/v2-5ad7ea88483a478d3ec016a41450060a_1440w.jpg)

### **6. 连接远程服务器**

写好配置文件后，在VS Code左边栏点击Remote Explorer图标，然后点击Refresh按钮来刷新连接，在SSH一栏会出现一个名为liming的新的连接如下图所示：

**![img](https://pic4.zhimg.com/v2-e6dd509b53791edfaad166d833f417b7_1440w.jpg)**

点击该新连接右边的箭头符号，即可在当前窗口打开远程服务器的窗口。在新的窗口上，左下角显示远程服务器已连通的标志，同时自动切换到左上角的文件图标按钮，并显示服务器上用户目录下的所有子目录和文件。整个过程如下两图所示：

![img](https://picx.zhimg.com/v2-a1e6d27777df42eb6a9266915643e52b_1440w.jpg)

![img](https://pic2.zhimg.com/v2-be9cd6508759aa66c3dc176ba5ff1069_1440w.jpg)

### **7. 调试Python代码**

首先在左侧的文件目录窗口找到要运行的Python代码文件并单击打开，然后在界面右下角选择服务器上的编译器（这个编译器对应着前面配置Conda环境安装的Python版本，SSH连接成功后会自动选择一个默认的编译器），接着单击代码行号左侧设置断点，最后按F5开始调试并运行到断点处。整个过程如下两图所示：

![img](https://picx.zhimg.com/v2-19dcf81c2db7264606dcf12e8573c14f_1440w.jpg)

![img](https://pic1.zhimg.com/v2-02c59c1a5696f04c20a0a31f5db7d4ae_1440w.jpg)

### **8. 调试代码注意事项**

1）对于左侧的文件目录窗口，单击文件会用新的代码窗口覆盖原有的代码窗口，双击文件则新的代码窗口不覆盖原有的代码窗口。

2）对于调试的启动按钮，如果选择菜单栏Run中的Start Debugging (F5)，会加载配置文件launch.json；而如果选择代码窗口右上方的调试按钮Debug Python File，则不会加载launch.json。如下图所示：

![img](https://pic3.zhimg.com/v2-4e58093dd75f83cfab937c84d1f70b92_1440w.jpg)

3）VS Code的launch.json是个全局配置文件，位于用户主目录的.vscode之下，而似乎没有为单个Python工程而设的局部配置文件。但是可以在launch.json中为多个Python工程添加对应的多个配置，即configurations数组中的每一项配置对应一个Python工程，仍然通过点击菜单栏Run中的Add Configuration…来实现。在VS Code左边栏点击Run and Debug图标，然后在上方的下拉列表中选择配置名称（由launch.json中的name字段设置），即可加载相应的配置，如下图所示：

![img](https://pic1.zhimg.com/v2-61802bc0af5bfb9302161f665858d318_1440w.jpg)