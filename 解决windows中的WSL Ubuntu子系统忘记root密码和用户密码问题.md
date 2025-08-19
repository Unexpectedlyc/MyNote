# 解决windows中的WSL Ubuntu子系统忘记root密码和用户密码问题

### 1、在powershell中执行wsl.exe --user root

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/6c0bcd143509bf8793eb26352b93bac7.png)

**如果出现了上面的报错，则需要运行步骤3、4，然后在执行步骤5改密码，如果没有出错，请直接跳到第5步改密码操作！！！**
3、运行 wslconfig /l 命令查看子系统版本

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/6bd4801e1c7c2d1023f5815c2741db35.png)

这里看到我的子系统名称为ubuntu-22.04

4、运行如下命令wsl.exe -d Ubuntu-22.04 --user root，命令中ubuntu-22.04是子系统的版本名，来自上面查询的结果，这里需要根据你查寻的结果进行替换

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/7cdc46218bc6fb1712299b7334fcbccf.png)

5、运行passwd root 修改 root 密码

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/aaacd64c63a8bf8b15072f4a48dec7a6.png)

```
#设置默认启动的wsl2的系统
wslconfig /setdefault <分发版名称>
```

