# Pyenv安装linux

> [!NOTE]
>
> github：[pyenv/pyenv: Simple Python version management](https://github.com/pyenv/pyenv?tab=readme-ov-file#1-automatic-installer-recommended)
>
> 需要先安装c++,c相关依赖

## 一 .在线安装

### 1. 自动安装 (Recommended)

```
curl -fsSL https://pyenv.run | bash
```

### 2.编辑 ~/.bashrc

```
sudo vim ~/.bashrc
```

将以下内容复制到末尾~/.bashrc

```
export PYENV_ROOT="$HOME/.pyenv"
[[ -d $PYENV_ROOT/bin ]] && export PATH="$PYENV_ROOT/bin:$PATH"
eval "$(pyenv init - bash)"
eval "$(pyenv virtualenv-init -)"
```

更新配置

```
source  ~/.bashrc
```

### 3.验证

```
pyenv --version
```

### 4.更换国内镜像源

linux/mac

```
 export PYTHON_BUILD_MIRROR_URL="https://registry.npmmirror.com/-/binary/python"
 export PYTHON_BUILD_MIRROR_URL_SKIP_CHECKSUM=1
```

windows

```
PYTHON_BUILD_MIRROR_URL=https://jedore.top/tools/python-mirrors/
```

使用 "pyenv update" 刷新缓存的 Python 版本及下载地址, 重新安装即可



## 二.离线安装

在Ubuntu上离线安装pyenv的步骤包括下载pyenv安装包和Python版本，然后配置环境变量以便使用。

### 步骤概述

1. **安装pyenv**:

- 将下载的pyenv安装包解压到您希望安装的目录中。然后，您可以使用以下命令安装pyenv：

```bash
cd /path/to/pyenv
./install.sh
```

安装完成后，您需要将pyenv的路径添加到您的bash配置文件中（例如`.bashrc`或`.bash_profile`）

   2.**配置环境变量**:

- 编辑`~/.bashrc`文件，添加以下行以便每次打开终端时加载pyenv：

```bash
export PATH="$HOME/.pyenv/bin:$PATH"
eval "$(pyenv init --path)"
eval "$(pyenv init -)"
```

- 保存文件后，运行以下命令使更改生效：

```bash
source ~/.bashrc
```

1. **下载Python安装包**:

- [您需要下载与您的操作系统和架构相匹配的Python安装包。可以在有网络的环境中下载Python的源码包，然后将其转移到目标Ubuntu系统上。 ](https://www.oryoy.com/news/gao-bie-wang-luo-yong-du-jiao-ni-qing-song-li-xian-an-zhuang-pyenv-yu-python-huan-jing.html)


2. **安装Python**:

- 使用以下命令安装Python（假设您已经将Python源码包放在pyenv的缓存目录中）:

```bash
pyenv install <python-version>
```

- 例如，要安装Python 3.8.10，可以运行：

```bash
pyenv install 3.8.10
```

   3.**使用pyenv管理Python版本**:

- 安装完成后，您可以使用pyenv添加和管理Python版本。可以通过以下命令查看已安装的Python版本：

```bash
pyenv versions
```

[通过以上步骤，您就可以在离线环境中成功安装pyenv和Python环境。这种方法特别适用于网络不稳定或内网环境的开发场景](https://www.oryoy.com/news/gao-bie-wang-luo-yong-du-jiao-ni-qing-song-li-xian-an-zhuang-pyenv-yu-python-huan-jing.html)
