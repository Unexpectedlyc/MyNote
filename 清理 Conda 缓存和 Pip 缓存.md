# 清理 Conda 缓存和 Pip 缓存

## 清理 Conda 缓存

查看 Conda 缓存的使用情况：

```none
conda clean --dry-run --all
```

删除不再使用的包和缓存：

```none
conda clean --all
```

## 清理 Pip 缓存

在使用 pip 安装 Python 库时，如果之前已经下载过该库，pip 会默认使用缓存来安装库，而不是重新从网络上下载。缓存文件通常存储在用户目录下的缓存文件夹中，具体位置因操作系统和Python版本而异。以下是一些常见的Python版本和操作系统下缓存文件的默认位置：

```none
Windows 10：C:\Users\username\AppData\Local\pip\Cache
macOS：/Users/username/Library/Caches/pip
Linux：~/.cache/pip
```

查看缓存信息:

```none
pip cache info
```

查看 cache 列表和路径：

```none
pip cache list
pip cache dir
```

清除缓存:

```none
pip cache purge   # 清除所有缓存，包括已下载但未安装的软件包和已安装但未被使用的缓存
pip cache remove package-name   # 只清除特定软件包的缓存。package-name 是你要清除缓存的软件包的名称。
```

