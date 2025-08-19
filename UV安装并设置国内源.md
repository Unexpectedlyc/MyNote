# UV安装并设置国内源

## 一、UV下载

### 1.官方一键安装

```
# On Windows.
powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 | iex"
# On macOS and Linux.
curl -LsSf https://astral.sh/uv/install.sh | sh
```

### 2.github下载安装

国内网络问题无法下载解决方案

来到github下载自己对应系统的包：
https://github.com/astral-sh/uv/releases

![img](https://i-blog.csdnimg.cn/direct/f35ac3e06c9448d38fa3eb735254c454.png)

我这里以linux为例选择`x86_64`下载地址为：
https://github.com/astral-sh/uv/releases/download/0.8.3/uv-x86_64-unknown-linux-gnu.tar.gz

下载到本地后上传到服务器并解压缩

### Linux/macOS 安装

将uv 、 uvx 放到 /usr/local/bin下即可

```
# 解压到指定目录
tar -xzf uv-x86_64-unknown-linux-gnu.tar.gz -C /usr/local/
mv /usr/local/uv-x86_64-unknown-linux-gnu /usr/local/uv

# 添加到环境变量（临时）
export PATH="/usr/local/uv:$PATH"

# 永久生效（添加到 ~/.bashrc 或 ~/.zshrc）
echo 'export PATH="/usr/local/uv:$PATH"' >> ~/.bashrc
source ~/.bashrc
```

### Windows 安装

```powershell
# 解压到 C:\uv
# 添加 C:\uv\bin 到系统环境变量 PATH
```

### 2.1 验证安装

打开终端，输入以下命令验证安装：

```bash
uv --version
# 输出示例：uv 0.7.11

uv --help
# 查看可用命令
```

### 2.2 准备 Python 环境

UV 默认使用 [python-build-standalone](https://link.zhihu.com/?target=https%3A//github.com/astral-sh/python-build-standalone) 项目编译的 Python。在内网环境中，我们需要手动下载这些预编译的 Python 版本。

### 2.2.1 选择 Python 版本

访问 [python-build-standalone releases](https://link.zhihu.com/?target=https%3A//github.com/astral-sh/python-build-standalone/releases) 页面选择合适的版本。

**示例：**

- Linux x86_64: `cpython-3.11.13+20250612-x86_64-unknown-linux-gnu-install_only_stripped.tar.gz`
- Windows x86_64: `cpython-3.11.13+20250612-x86_64-pc-windows-msvc-install_only_stripped.tar.gz`

### 2.2.2 安装 Python

```bash
# 创建 Python 安装目录
sudo mkdir -p /usr/local/python-build-standalone

# 解压 Python
tar -xzf cpython-3.11.13+20250612-x86_64-unknown-linux-gnu-install_only_stripped.tar.gz -C /usr/local/python-build-standalone

# 验证安装
/usr/local/python-build-standalone/bin/python3 --version
```

### 2.3 配置 Python 环境

由于内网环境无法在线下载 Python，我们需要配置本地 Python 环境：

```bash
# 将 Python 添加到 PATH（优先级最高）
export PATH="/usr/local/python-build-standalone/bin:$PATH"

# 验证 UV 能够识别 Python
uv python list --no-managed-python
# 应该显示我们安装的 Python 路径

# 测试创建项目
mkdir test-project && cd test-project
uv init --no-managed-python
```

### 项目管理

### 3 创建新项目

```bash
# 创建新项目
uv init my-project --no-managed-python
cd my-project
```

### 总结

UV 作为新一代 Python 包管理工具，在企业内网环境中展现出了强大的优势：

### ✅ 主要优势

1. **极致性能**：比传统工具快 10-100 倍
2. **统一工具链**：一个工具替代多个传统工具
3. **现代化设计**：支持锁文件、工作空间等特性
4. **内网友好**：支持离线安装和本地缓存
5. **跨平台兼容**：完美支持各种操作系统

## 二、更换国内镜像源（加速下载）

### 方法1：临时环境变量（单次生效）

```
# 使用阿里云镜像源
export UV_INDEX_URL=https://mirrors.aliyun.com/pypi/simple/
uv pip install [包名]

# 或清华大学镜像源
export UV_INDEX_URL=https://pypi.tuna.tsinghua.edu.cn/simple/
```

### 方法2：永久配置（推荐）

创建或修改配置文件

在用户目录下创建 uv.toml 文件（路径参考）：

- Linux/macOS:` ~/.config/uv/uv.toml`
- Windows: `%APPDATA%\uv\uv.toml`

添加国内镜像源

编辑文件内容如下：

```
[[index]]
url = "https://mirrors.aliyun.com/pypi/simple/"
default = true
# 或使用清华源
# url = "https://pypi.tuna.tsinghua.edu.cn/simple/"
```

### 方法3：命令行直接指定源

```
uv pip install -i https://pypi.tuna.tsinghua.edu.cn/simple/ [包名]
```

## 三、验证镜像源是否生效

```
# 查看当前配置
uv config get index.url

# 安装测试包（观察下载速度）
uv pip install numpy
```

常见镜像源地址

| 镜像名称 | URL                                                  |
| -------- | ---------------------------------------------------- |
| 阿里云   | https://mirrors.aliyun.com/pypi/simple/              |
| 清华大学 | https://pypi.tuna.tsinghua.edu.cn/simple/            |
| 豆瓣     | https://pypi.doubanio.com/simple/                    |
| 华为云   | https://repo.huaweicloud.com/repository/pypi/simple/ |

注意事项

- 若同时使用 pip 和 uv，镜像源需分别配置（uv 不读取 pip 的配置）。
- 更换源后如遇 SSL 错误，尝试将 http:// 替换为 https://。
- 清除缓存命令：`uv clean`。