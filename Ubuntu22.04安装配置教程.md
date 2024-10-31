# Ubuntu22.04安装配置教程

## 1.安装VMware Tools

当之前安装过低版本的Ubuntu系统，或者是再交安装Ubuntu时，由于之前安装过VMware Tools，因此，虚拟机的菜单栏处是无法点击的，如下图所示：

![img](https://pic2.zhimg.com/v2-a79146ea7bda51b42cb644b1fcde17cf_b.jpg)

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



## 