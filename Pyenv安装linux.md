# Pyenv安装linux

> [!NOTE]
>
> github：[pyenv/pyenv: Simple Python version management](https://github.com/pyenv/pyenv?tab=readme-ov-file#1-automatic-installer-recommended)
>
> 需要先安装c++,c相关依赖

##### 1. 自动安装 (Recommended)

```
curl -fsSL https://pyenv.run | bash
```

##### 2.编辑 ~/.bashrc

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

##### 3.验证

```
pyenv --version
```

##### 4.更换国内镜像源

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

