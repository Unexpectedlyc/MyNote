# Windows 10 离线安装 WSL2 Ubuntu 22.04.5 LTS 简明教程 - 踩坑及解决方案

### **背景**

近期开始探索在WIndows PC上通过[LMDeploy](https://zhida.zhihu.com/search?content_id=253789255&content_type=Article&match_order=1&q=LMDeploy&zhida_source=entity)运行本地部署的[Deepseek-R1](https://zhida.zhihu.com/search?content_id=253789255&content_type=Article&match_order=1&q=Deepseek-R1&zhida_source=entity)模型。前置步骤需要用[WSL2](https://zhida.zhihu.com/search?content_id=253789255&content_type=Article&match_order=1&q=WSL2&zhida_source=entity) Ubuntu。把流程教程和踩坑及解决方案发出来供后人参考。

到在 Windows 10 专业版（版本 19044.3086）环境中，因网络问题叠加本地WSL安装错误（错误代码 `0xc8000641`）无法通过在线命令 `wsl --install` 完成安装。通过手动下载 WSL 镜像并离线导入，成功部署 Ubuntu 22.04.5 LTS。以下是完整操作流程：

本教程适用于当你无法使用以下命令成功安装Ubuntu .

```text
wsl --install -d Ubuntu-22.04
```

### **一、准备工作**

### 1. **系统要求**

- **操作系统**：Windows 10 2004+（Build 19041+）或 Windows 11
- **硬件虚拟化**：在 BIOS/UEFI 中启用 **Intel VT-x/AMD-V**
- **存储空间**：至少 5GB 空闲空间（建议 SSD）

### 2. **启用 WSL 功能**

```text
powershell
Copy

# 以管理员身份运行 PowerShell 
dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart 
dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart 
wsl --set-default-version 2 

Restart-Computer
```

### 3. 手动启用必需的 Windows 功能

1. **启用“虚拟机平台”与“适用于 Linux 的 Windows 子系统”**

- 打开【控制面板】→【程序】→【启用或关闭 Windows 功能】。
- 勾选**虚拟机平台**、**适用于 Linux 的 Windows 子系统**，（如有 Hyper-V，建议也勾选）。
- 点击确定后，重启计算机。

> [!IMPORTANT]
>
> 如何查看虚拟化是否开启
>
> ![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/dc5c7d26ef35470fbe6371f78c08f911.png)

![img](https://pic4.zhimg.com/v2-8af2b8587b51dfb80ab8a9132e2f4413_1440w.jpg)

![img](https://pica.zhimg.com/v2-0947d2483b3bd36c523f1326592d296a_1440w.jpg)

**确认 BIOS 中已启用虚拟化支持**
进入 BIOS 设置，确保**虚拟化技术**（VT-x/AMD-V）已启用。

### **二、手动下载 Ubuntu WSL 镜像**

### 1. **获取官方镜像**

- **下载地址**：
  [Ubuntu 22.04 WSL ](https://learn.microsoft.com/en-us/windows/wsl/)- Windows Subsystem for Linux Documentation
  - 在此页面中找到[Manual install steps](https://learn.microsoft.com/en-us/windows/wsl/install-manual) （有趣的是，如果你直接点击这个链接，你很可能会得到“无访问权限”的返回）之后有两个方法，我尝试了第二个才成功。
  - [Ubuntu 22.04.5 LTS - Free download and install on Windows | Microsoft Store](https://link.zhihu.com/?target=https%3A//apps.microsoft.com/detail/9pn20msr04dw%3Fhl%3Den-us%26gl%3DNZ)
  - [Ubuntu 22.04 LTS](https://link.zhihu.com/?target=https%3A//aka.ms/wslubuntu2204)（我所安装版本的文件直接下载路径）

### 2. **保存镜像文件**

将下载的 `Ubuntu2204-221101.AppxBundle` 保存至本地路径（如 `C:\wsl\`）。

### **三、离线安装 Ubuntu 22.04**

### 1. 打开 PowerShell

按 `**Win + X**`，然后选择：

- **Windows PowerShell（管理员）** 或
- **Windows Terminal（管理员）**

### 2. 导航到下载文件夹

```text
cd C:\wsl\
```

然后按 **Enter**，这样就进入了你的 **下载文件夹 （以“**C:\wsl\” 为例**）**。

### 3. 导航到下载文件夹

输入以下命令并回车：

```text
Add-AppxPackage .\Ubuntu2204-221101.AppxBundle
 
```

这将安装 `Ubuntu 22.04`。

**⚠️ 注意事项**

1. **必须用管理员权限** 打开 PowerShell，否则可能会遇到权限问题。
2. **确保文件名正确**，可以用 `ls` 或 `dir` 命令列出下载目录中的文件：powershell
   CopyEdit
   `dir`
   如果文件名有错误，可以用 `Tab` 键自动补全。
3. **如果失败，可以尝试旁加载**：

- **打开 Windows 设置** → **Apps** → **Apps & Features**
- **找到 “Sideload apps”** 选项并启用



安装成功后，你可以在**开始菜单**搜索 `Ubuntu 22.04`，然后运行它！



**三. 如何判断是否安装成功？**

### **检查“开始”菜单**

1. 按 **`Win + S`**（打开 Windows 搜索）。
2. 输入 **`Ubuntu 22.04`**。
3. 如果出现 **"Ubuntu 22.04"** 应用图标，并且能正常打开，则表示安装成功。

### **⚠️ 如果安装失败**

如果 `Add-AppxPackage` 命令失败或找不到应用，可以尝试：

```text
powershell

Add-AppxPackage -Register C:\Users\zhtom\Downloads\Ubuntu2204-221101.AppxBundle
```

如果仍然无法安装，可以重新下载 `.AppxBundle` 文件，并确保 `Sideload Apps` 选项已启用。



**至此Ubuntu 22.04 已成功安装！** 如果你现在正在进行 **WSL（Windows Subsystem for Linux）** 的**首次初始化**。目前，它要求你**创建一个默认的 UNIX 用户账户**，这个账户将用于 WSL 中的日常操作。

### **你需要做的操作**

**在 PowerShell 终端或 Ubuntu 窗口中，按照以下步骤输入信息：**

### **1️⃣ 输入你的 UNIX 用户名**

在 `Enter new UNIX username:` 后输入你想要的 Linux 账户名（无需与 Windows 用户名相同），例如：
**tom**

然后按 **Enter**。
✅ **注意：**

- 不能使用大写字母。
- 不能有空格。
- 这个用户名将是默认用户，具有 `sudo` 权限（类似 Windows 的管理员权限）。

**2️⃣ 创建一个密码**

它会提示：

```
Enter new UNIX password:
```

输入你的密码（⚠️ **输入时不会显示任何字符，这是 Linux 的安全特性**），然后按 **Enter**。

✅ **密码要求：**

- **必须记住这个密码！** 因为以后使用 `sudo`（管理员权限）时需要输入它。
- **至少6个字符**，尽量使用大小写字母、数字、特殊字符的组合，以增强安全性。



### **3️⃣ 确认密码**

```text
Retype new UNIX password: 
```

再次输入 **相同的密码**，然后按 **Enter**。

如果输入正确，你会看到类似：

```text
passwd: password updated successfully
Installation successful! 
```

说明你的 WSL Ubuntu 用户已创建，并且安装完成 ！



### **四. 接下来可以做什么？**

1️⃣ **测试 Ubuntu 是否成功安装** 在 Ubuntu 终端中输入：
`whoami`

如果返回的是你的用户名（例如 `tracy`），说明账户创建成功。

2️⃣ **检查 WSL 版本** 在 PowerShell 运行：

```text
wsl --list --verbose 
```

如果返回：

```text
NAME            STATE           VERSION
Ubuntu-22.04    Running         2 
```

说明 WSL 2 已成功启用。

3️⃣ **更新 Ubuntu** **首次使用建议更新系统软件**，在 Ubuntu 终端中输入：&`lt;br/>sudo apt update && sudo apt upgrade -y&`lt;br/>

然后输入刚才创建的 **密码** 进行更新。



### **你现在可以开始使用 Ubuntu 22.04 了！**

你可以：

- **在 PowerShell 或 CMD 运行 `wsl`** 进入 Linux 终端：powershell
  `wsl`
- **使用 `exit` 退出 WSL**sh
  `exit`
- **直接在 Windows 终端中打开 Ubuntu** 按 `Win + S` 搜索 **Ubuntu 22.04**，打开即可。



### **五、其他注意事项**

1. **路径规范**：安装目录避免空格和中文（如 `C:\wsl\`）。
2. **权限问题**：始终以管理员身份运行 PowerShell。
3. **镜像完整性**：下载后校验 SHA256（官网提供校验值）。

### **总结**

通过离线安装方式，可有效规避网络或其他问题引起的错误（如 `0xc8000641`）。此方案适用于企业内网隔离、网络不稳定或微软商店访问受限的场景。未来若需安装其他分发版（如 Debian、Kali），只需替换镜像文件路径即可。