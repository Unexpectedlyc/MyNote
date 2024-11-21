# Windows 安装 PostgreSQL  vector 扩展

vector 扩展

下载地址：

> vector: Open-source vector similarity search for Postgres / PostgreSQL Extension Network
> Supports L2 distance, inner product, and cosine distance
> https://pgxn.org/dist/vector/

![img](https://i-blog.csdnimg.cn/direct/83df6667614f4437abc19ae45e169a98.png)

下载后解压

解压后的根目录为 C:\Users\xxx\Downloads\vector-0.7.3（编译时在命令行会使用cd进入到这个路径，进行编译安装）

![img](https://i-blog.csdnimg.cn/direct/1cb0fd51d869402789fe574a980fbdcd.png)

在 Windows 上编译需要先下载 Visual Studio

> Visual Studio: 面向软件开发人员和 Teams 的 IDE 和代码编辑器
> Visual Studio 开发工具和服务让任何开发人员在任何平台和语言的应用开发都更加轻松。 随时随地免费使用代码编辑器或 IDE 进行开发。
> https://visualstudio.microsoft.com/zh-hans/

![img](https://i-blog.csdnimg.cn/direct/3c41bdc061fd4e0d9a587088ce70da31.png)

安装时勾选C++

![img](https://i-blog.csdnimg.cn/direct/894dd6f6d2f24f3384193e094d24b171.png)

安装完成后，使用**管理员模式**打开cmd，依次执行以下命令便能够完成安装

```bash
call "C:\Program Files\Microsoft Visual Studio\2022\Community\VC\Auxiliary\Build\vcvars64.bat"
cd C:\Users\xxx\Downloads\vector-0.7.3
set "PGROOT=C:\Program Files\PostgreSQL\16"
nmake /F Makefile.win
nmake /F Makefile.win install
```
最后在数据库连接工具中，选中具体的数据库实例，执行以下命令，就能扩展 vector 类型了

```sql
CREATE EXTENSION IF NOT EXISTS vector;
```

![img](https://i-blog.csdnimg.cn/direct/6b4c0b5fa9c94ba3bf33554e10401976.png)
