# VSCode下Python项目的launch.json的配置模板

示例为langchain-chatchat项目

```json
{
    // 配置文件的版本号，用于指定 launch.json 文件遵循的格式版本
    "version": "0.2.0",
    // 调试配置数组，可包含多个不同的调试配置
    "configurations": [
        {
            // 调试配置的名称，会在调试器的启动配置下拉菜单中显示
            "name": "Python Debugger: Current File",
            // 调试器的类型，这里使用 debugpy 作为 Python 调试器
            "type": "debugpy",
            // 请求类型，launch 表示启动一个新的进程进行调试
            "request": "launch",
            // 要调试的 Python 程序文件路径，${file} 表示当前打开的文件
            "program": "${file}",
            // 调试时使用的控制台类型，integratedTerminal 表示使用 IDE 集成的终端
            "console": "integratedTerminal",
            // 调试程序的工作目录，这里指定为工作区下的特定目录
            "cwd": "${workspaceFolder}/libs/chatchat-server/chatchat",
            // 传递给被调试程序的命令行参数
            "args": ["start", "-a"],
            // 调试程序时设置的环境变量
            "env": {
                // 定义 CHATCHAT_ROOT 环境变量，指向工作区下的特定目录
                "CHATCHAT_ROOT": "${workspaceFolder}/libs/chatchat-server/chatchat",
                // 定义 PYTHONPATH 环境变量，用于指定 Python 模块的搜索路径
                "PYTHONPATH": "${workspaceFolder}/libs/chatchat-server/"
            }
        }
    ]
}

```

