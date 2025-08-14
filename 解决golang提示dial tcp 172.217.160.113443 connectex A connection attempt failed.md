# 解决golang提示dial tcp 172.217.160.113:443: connectex: A connection attempt failed

[go get](https://zhida.zhihu.com/search?content_id=178832281&content_type=Article&match_order=1&q=go+get&zhida_source=entity): module [github.com/rogpeppe/godef:](https://link.zhihu.com/?target=http%3A//github.com/rogpeppe/godef%3A) Get "[https://proxy.golang.org/github.com/rogpeppe/godef/@v/list](https://link.zhihu.com/?target=https%3A//proxy.golang.org/github.com/rogpeppe/godef/@v/list)": dial tcp 172.217.160.113:443: connectex: A connection attempt failed because the connected party did not properly respond after a period of time, or established connection failed because connected host has failed to respond.

解决方法：改成我们国内可用的代理地址

在命令提示符输入： go env -w GOPROXY=https://goproxy.cn

然后再做各种操作就可以成功了

![img](https://picx.zhimg.com/v2-2991a1f2d95af8b8464023ac383f53b1_1440w.jpg)