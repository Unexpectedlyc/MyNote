# Ubuntu22.04安装配置教程

## 1.安装VMware Tools

当之前安装过低版本的Ubuntu系统，或者是再交安装Ubuntu时，由于之前安装过VMware Tools，因此，虚拟机的菜单栏处是无法点击的，如下图所示：

![ubuntu安装图](https://pic2.zhimg.com/v2-a79146ea7bda51b42cb644b1fcde17cf_b.jpg)

此时，需要先卸载ubuntu中的vmware tools工具后，才能重新安装。在Ubuntu中点击右键，打开终端，输入卸载指令即可，卸载执行代码如下：

```python
sudo apt purge open-vm-tools-desktop
```

输入上面指令后，回车即可卸载。注意，如果设定了用户密码，需要输入用户密码。完成后，先关闭虚拟机，然后再重新启动。再次进入系统后，就可以看到虚拟机上的菜单栏可以点击了。如果再次进入时，菜单仍无法点击，可以再次重复执行一次上面的指令。

（2）重新安装VMware Tools的注意事项

点击虚拟机菜单栏的“重新安装VMware Tools(T)"，Ubuntu桌面上会出现一个文件，打开后，将名字为“VMwareTools-10.3.22-15902021.tar.gz"的文件直接拖到桌面（Ubuntu22支持此操作），然后，在文件上鼠标右键点击，在弹出菜单中点击”提取到此处“，将该压缩文件直接释放到桌面上。打开解压后的文件夹，点击右键->”在终端打开“，输入以下安装指令(此处是进入解压后文件夹，运行其中的vmware-install.pl文件)：

```text
./vmware-install.pl
```

然后就开始安装，为了确保安装成功，请仔细阅读其中的英文提示，特别是后面的提示要覆盖原来的文件（针对多次安装不成功的，或安装一半中断的情况），要输入YES，然后再回车，不能全部采用默认值。

安装完成后，直接关闭虚拟机，然后再启动虚拟机。（注意：不要直接点重启虚拟机，会出现错误。）

系统重启后，可以打开/mnt/hgfs/目录，查看安装时设定的共享文件夹。

（3）解决无法复制粘贴问题

重新安装后，很有可能还是无法在虚拟机与原主机之间进行复制粘贴，此时，可以再次打开终端，输入并执行以下指令：

```python3
sudo apt-get autoremove open-vm-tools
sudo apt-get install open-vm-tools
sudo apt-get install open-vm-tools-desktop
```

上面的指令是再次卸载，然后安装。指令执行完毕后，关闭虚拟机，再次重启，此时，多半会大功告成！如果还不行，请再次执行上面的指令。（再次提醒：不要直接点击重启，要先关闭虚拟机，然后再启动。）

## 2.安装ssh,net-tools

```
sudo apt-get install ssh openssh-server
sudo apt-get install net-tools
```

## 3. 安装gcc和g++

### 1.安装GCC和G++：

```
sudo apt update
sudo apt install build-essential
sudo apt-get install libz-dev
sudo apt-get install gettext
```

回车运行等待即可。

### 2.验证安装：

安装完成后，输入以下命令验证GCC和G++的安装是否成功：

```
gcc --version
g++ --version
```

#### 1 **检查环境变量配置**

- 在终端中输入：

```bash
echo $PATH
```

- 显示：![img](https://i-blog.csdnimg.cn/direct/c6f52da41d944b21a0a784bb5909feb5.png)
- 并没有找到gcc，说明未添加到环境变量，那咱添加一手环境变量：



#### 2 找到gcc的路径

- 输入：

```bash
which gcc
```

- 看看返回值：

![img](https://i-blog.csdnimg.cn/direct/1372a6e3fafa44b18d6a31a65343f698.png)

- 记下gcc的路径



#### 3 添加gcc路径到Linux环境变量

- 在终端中输入并执行**vim ~/.bashrc**命令，用**vim**工具进入**~/.bashrc**文件
- 光标选到文件最后一个字符，输入**o**命令在光标下插入新行，进入编辑模式，在最后一行加上以下环境变量↓

```bash
export PATH=$PATH:/usr/bin/gcc
```

- 单击**esc**键退出编辑模式，输入**:wq**保存修改后退出vim窗口
- 输入source ~/.bashrc命令更新bashrc文件，重启shell终端

### 3.编译和运行C程序：

代码如下

```
#include <stdio.h>

int main() {
    printf("Hello, GCC!\n");
    return 0;
}
```

使用下面编译命令编译

```
gcc main.c -o main
```

编译完成后使用下面的命令运行：

```
./main
```

终端将显示`"Hello, GCC!"`，这证明您的程序已成功编译和运行。

### 4.编译和运行C++程序：

```
#include <iostream>
using namespace std;

int main() {
    cout << "Hello, G++!" << endl;
    return 0;
}
```

编译：

```
g++ hello.cpp -o hello
```

运行：

```
./hello
```

终端将显示`"Hello, G++!"`，这证明您的C++程序已成功编译和运行。

###  5.gcc和g++的选项

GCC和G++提供了许多选项，可用于控制编译和链接过程。

1.**使用-c选项只进行编译，不进行链接：**

```
gcc -c source.c   // 编译source.c文件，生成目标文件source.o
```

**2.使用-o选项指定输出文件名：**

```
gcc source.c -o output   // 编译source.c文件，并将输出文件命名为output
```

**3.使用-Wall选项启用所有警告信息：**

```
gcc -Wall source.c   // 编译source.c文件，并启用所有警告信息
```

**4.使用-g选项启用调试信息：**

```
gcc -g source.c   // 编译source.c文件，并生成带调试信息的可执行文件
```

**5.使用-I选项指定包含文件的目录：**

```
gcc -I include source.c   // 编译source.c文件时，在include目录中查找包含文件
```

**6.使用-L选项指定库文件的目录：**

```
gcc -L /path/to/lib source.c -o output   // 编译source.c文件时，在指定目录中查找库文件
```

## 4.设置永久环境变量

当然，你可以添加多个环境变量。以下是几种方法来在 `.bashrc` 文件中添加多个环境变量，并确保它们都生效。

### 方法一：手动编辑文件

1. **打开终端**。

2. **编辑 .bashrc 文件**：
   
   - 使用你喜欢的文本编辑器打开 `~/.bashrc` 文件。例如，使用 `nano` 编辑器，可以输入以下命令：
     ```bash
     nano ~/.bashrc
     ```
   
3. **在文件末尾添加多个环境变量**：
   
   - 将光标移动到文件的末尾，然后添加以下行：
     ```bash
     export PATH=$PATH:/path/to/your/bin1
     export PATH=$PATH:/path/to/your/bin2
     export PATH=$PATH:/path/to/your/bin3
     ```
   
4. **保存并退出**：
   - 在 `nano` 中，按 `Ctrl+O` 保存文件，然后按 `Ctrl+X` 退出编辑器。

5. **使更改生效**：
   
   - 为了让新的环境变量立即生效，你需要重新加载 `.bashrc` 文件：
     ```bash
     source ~/.bashrc
     ```

### 方法二：使用命令行直接添加

如果你不想手动打开编辑器，也可以直接通过命令行添加多行：

1. **打开终端**。

2. **使用 echo 命令逐行追加到 .bashrc 文件**：
   - 运行以下命令：
     ```bash
     echo 'export PATH=$PATH:/path/to/your/bin1' >> ~/.bashrc
     echo 'export PATH=$PATH:/path/to/your/bin2' >> ~/.bashrc
     echo 'export PATH=$PATH:/path/to/your/bin3' >> ~/.bashrc
     ```

3. **使更改生效**：
   - 重新加载 `.bashrc` 文件以使新添加的环境变量生效：
     ```bash
     source ~/.bashrc
     ```

### 方法三：使用一个命令添加多个环境变量

你也可以在一个命令中添加多个环境变量：

1. **打开终端**。

2. **使用 echo 和 here-document (<<) 语法一次性添加多个环境变量**：
   - 运行以下命令：
     ```bash
     cat <<EOF >> ~/.bashrc
     export PATH=$PATH:/path/to/your/bin1
     export PATH=$PATH:/path/to/your/bin2
     export PATH=$PATH:/path/to/your/bin3
     EOF
     ```

3. **使更改生效**：
   - 重新加载 `.bashrc` 文件以使新添加的环境变量生效：
     ```bash
     source ~/.bashrc
     ```

### 验证

为了确保环境变量已经正确设置，你可以运行以下命令来检查 `PATH` 变量：

```bash
echo $PATH
```

你应该能看到 `/path/to/your/bin1`、`/path/to/your/bin2` 和 `/path/to/your/bin3` 都已经被添加到了 `PATH` 变量的末尾。

以上步骤完成后，你的环境变量就已经被永久设置了。每次打开新的终端窗口时，新的 `PATH` 设置都会自动生效。

在Linux系统中，设置永久环境变量通常涉及编辑特定的配置文件。这些文件可以分为两类：全局配置文件和用户特定配置文件。



### 全局环境变量

如果你希望为所有用户设置环境变量，你可以编辑以下文件之一：

- `/etc/environment`：这个文件是系统启动时加载的基本环境变量。它不是shell脚本，所以只能包含变量名=值的形式，不能有其他命令或逻辑。

- `/etc/profile`：这个文件适用于所有用户的登录shell。在这个文件中，你可以使用shell命令来设置环境变量。

- `/etc/profile.d/` 目录下的脚本：在这个目录下，你可以创建一个以 `.sh` 结尾的脚本文件来设置环境变量。这些脚本会在每个用户的登录shell启动时执行。

- `/etc/bashrc` 或者 `/etc/bash.bashrc`：这些文件适用于所有用户的非登录交互式bash shell。

### 用户特定环境变量

如果你只希望为特定用户设置环境变量，你可以编辑该用户的以下文件：

- `~/.profile`：对于登录shell，这个文件是一个很好的选择。它会被所有 Bourne-compatible shells（如 bash 和 dash）读取。

- `~/.bash_profile`：对于使用bash作为shell的用户，这个文件会优先于 `~/.profile` 被读取。

- `~/.bashrc`：对于非登录shell（例如，当你打开一个新的终端窗口时），这个文件会被读取。这是设置别名、函数和环境变量的好地方。

### 设置环境变量

要设置环境变量，你需要在上述文件中添加类似下面的行：

```bash
export VARIABLE_NAME=value
```

例如，如果你想为特定用户设置 `JAVA_HOME` 环境变量，可以在 `~/.bashrc` 文件中添加如下行：

```bash
export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
```

然后，为了使更改生效，你需要重新加载配置文件，可以通过运行以下命令实现：

```bash
source ~/.bashrc
```

或者，如果你修改的是全局配置文件，可能需要重新登录或重启计算机以使更改生效。

确保在设置环境变量时检查路径和值的正确性，避免因错误的环境变量导致程序无法正常运行。

## 5.把普通用户升级为root

在 CentOS 系统中，将普通用户提升至具有 `root` 权限的操作需要非常谨慎。通常有以下几种方式来实现这一目标：

### 1. 使用 `sudo` 命令

如果您的系统已经安装了 `sudo`，您可以将普通用户添加到 `/etc/sudoers` 文件中，使其能够执行特定的命令或所有 `root` 用户可以执行的命令。为了安全起见，建议使用 `visudo` 命令编辑此文件，因为它可以防止因语法错误而锁定系统。

- 打开终端。
- 以 `root` 用户身份运行 `visudo` 命令：
  ```bash
  sudo visudo
  ```
- 在打开的文件中找到下面这行（或者类似行）：
  ```bash
  ## Allow root to run any commands anywhere
  root    ALL=(ALL)       ALL
  ```
- 在该行下面添加一行，格式如下：
  ```bash
  username    ALL=(ALL)       ALL
  ```
  其中 `username` 是您希望赋予 `root` 权限的用户的用户名。
- 保存并退出编辑器（通常是按 `Esc` 键，然后输入 `:wq` 并按回车键）。
- 现在，`username` 可以通过 `sudo` 命令执行任何 `root` 权限下的命令，例如：
  ```bash
  sudo command
  ```

### 2. 将用户添加到 `wheel` 组

在某些 CentOS 安装中，默认情况下只有 `wheel` 组的成员可以通过 `sudo` 执行命令。如果您希望某个用户能够使用 `sudo`，可以将其添加到 `wheel` 组：

- 以 `root` 用户身份登录。
- 使用以下命令将用户添加到 `wheel` 组：
  ```bash
  usermod -aG wheel username
  ```
  这里的 `username` 是您想要添加到 `wheel` 组的用户名。
- 如果需要，确保 `/etc/sudoers` 文件中包含允许 `wheel` 组成员使用 `sudo` 的条目。通常这个设置默认是存在的，但如果没有，您可以通过 `visudo` 添加如下行：
  ```bash
  %wheel  ALL=(ALL)       ALL
  ```

### 3. 修改用户的 UID 为 0

直接将用户的 UID 设置为 0 会使该用户拥有与 `root` 相同的权限，但这通常不推荐，因为这样做会绕过所有安全措施，增加系统被滥用的风险。

- 以 `root` 用户身份登录。
- 使用 `usermod` 命令修改用户的 UID：
  ```bash
  usermod -u 0 username
  ```
  但是请注意，这种方法极其危险，不建议使用。

### 注意事项

- 在赋予用户 `root` 权限时，一定要考虑安全性和必要性。
- 操作 `/etc/sudoers` 文件时务必小心，错误的配置可能导致无法访问系统。
- 授予 `root` 权限后，应教育用户正确使用这些权限，避免对系统造成不必要的损害。

完成上述步骤后，您的普通用户应该就能够以 `root` 身份执行命令了。