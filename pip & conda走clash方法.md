# pip & conda走clash方法

## pip

Clash for Windows -> Settings -> System Proxy -> [v] Specify Protocol

## conda

1. 生成.condarc文件

```powershell
conda config --set show_channel_urls yes
```

2. 打开.condarc (C:\Users\xxxx\.condarc)并添加

```text
proxy_servers:
  http: http://127.0.0.1:7890
  https: http://127.0.0.1:7890
```

其中127.0.0.1:7890是实际的代理端口

取消所有源，替换为默认

```text
pip config unset global.index-url
```