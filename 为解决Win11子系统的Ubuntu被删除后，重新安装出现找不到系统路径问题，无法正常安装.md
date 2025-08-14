# 为解决Win11子系统的Ubuntu被删除后，重新安装出现找不到系统路径问题，无法正常安装

出现如下问题如下：

{{C:\Users\HP>wsl 无法将磁盘“C:\Users\HP\AppData\Local\Packages\CanonicalGroupLimited.Ubuntu_79rhkp1fndgsc\LocalState\ext4.vhdx”附加到 WSL2： 系统找不到指定的文件。 Error code: Wsl/Service/CreateInstance/MountVhd/HCS/ERROR_FILE_NOT_FOUND}}(style=red|mode=revision|title=|advice=)

解决方法如下：

一、先使用如下命令查看全部环境;

> wsl -l

二、找到对应Ubuntu版本如下：（这里我设置了默认直接输入：Ubuntu就好）

使用如下命令：

> wsl.exe --unregister (版本号)

这里已经成功注销！

以管理员身份打开linux子系统