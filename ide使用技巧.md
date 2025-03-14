# ide使用技巧

## 1.vscode导出到extensions文本，一键安装

```
code --list-extensions > extensions.txt
Get-Content extensions.txt | ForEach-Object { code --install-extension $_ }
```

### 
