# ide使用技巧

## 1.vscode导出到extensions文本，一键安装

```
code --list-extensions > extensions.txt
Get-Content extensions.txt | ForEach-Object { code --install-extension $_ }
```

## 2.VSCode设置代理模式

在搜索框中输入【proxy】，选择【代理服务器】，点击【在settings.josn中编辑】

![img](https://pic4.zhimg.com/v2-161486eb15e83b2f2bf26707ca355197_1440w.jpg)

在打开的 settings.json文件中，添加以下代码：

```
"http.proxy": "http://your-proxy-server:port",
"https.proxy": "https://your-proxy-server:port",
"http.proxyStrictSSL": false
```

![img](https://pic4.zhimg.com/v2-41851c820e2a3e98f395876482e3d1af_1440w.jpg)

### 通过可视化配置设置

打开VS Code设置页面，在搜索框中输入【proxy】，选择【代理服务器】，在第一项中填入【你的代理IP】，有必要的取消勾选【Http: Proxy Strict SSL】选项

![img](https://pic3.zhimg.com/v2-4823b794928dbdf23b5043655f8d2d9a_1440w.jpg)

### 注意事项

- 确保代理服务器正常可用
- 确保代理服务器地址和端口号正确
- 如果代理需要身份验证，可能还需要在设置文件中添加 http.proxyAuthorization
