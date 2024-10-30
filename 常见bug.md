# 常见bug

## 1. 安装shapely报错，找不到geos_c.dll

有的小伙伴pip install shapely 安装完成后还报错

错误： Could not find module ‘C:\Users\Administrator\anaconda3\Library\bin\geos_c.dll’ (or one of its dependencies). Try using the full path with constructor syntax.

解决方案：在这个网址下载geos_c.dll，放到***\Anaconda3\Library\bin目录下面

https://www.dll-files.com/geos_c.dll.html
原文链接：https://blog.csdn.net/love1005lin/article/details/115739817

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/6ed8ba6b551209c8243986acd856069d.png)

下载完成解压文件夹

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/37591c879140680cca24ae45cb17f345.png)

将geos_c.dll 文件放到***\Anaconda3\Library\bin目录下面 然后重新导入shapely 包就好用了

## 2. 解决node.js-opensslErrorStack: [ ‘error:03000086:digital envelope routines::initialization error‘ ]错误

详细错误提示如下：

![img](https://i-blog.csdnimg.cn/blog_migrate/64b983f8165f54e306939811841b585f.png)

1-出现这个错误原因：因为我之前是node16更新到18后出现这个查了很多资料才知道node高版本加入了更严格的限制。
2-在项目的package.json文件下更改<scripts>加上这行代码SET NODE_OPTIONS=--openssl-legacy-provider &&
截图如下：
![img](https://i-blog.csdnimg.cn/blog_migrate/895521ba16dac8129e8f9d13621b2df4.png)

3-重新运行npm run dev命令行完美解决这个问题

## 3. 解决 Github port 443 : Timed out

### 一、问题描述

如下图所示，无法 git clone 来自 Github 上的仓库，报端口 443 错误

![img](https://pic1.zhimg.com/v2-3b264466dfeec3e70ff0534c35da18c2_b.jpg)

### 二、问题分析

Git 所设端口与系统代理不一致，需重新设置

### 三、解决方法

#### 3-1、打开代理页面

打开 设置 --> 网络与Internet --> 查找代理

![动图](https://pic2.zhimg.com/v2-baceb7a931e8f13ff5b0956a68955639_b.webp)

记录下当前系统代理的 IP 地址和端口号

如上图所示，地址与端口号为：127.0.0.1:7890

#### 3-2、修改 Git 的网络设置

```
# 注意修改成自己的IP和端口号
git config --global http.proxy http://127.0.0.1:7890 
git config --global https.proxy http://127.0.0.1:7890
```

![img](https://pic2.zhimg.com/v2-32f6f2e6e7ea50f9de6f9aa35bd56dcd_b.jpg)

#### 3-3、取消代理

```bash
# 取消代理
git config --global --unset http.proxy
git config --global --unset https.proxy

# 查看代理
git config --global --get http.proxy
git config --global --get https.proxy
```
