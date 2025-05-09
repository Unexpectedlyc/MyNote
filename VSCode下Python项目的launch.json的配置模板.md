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

# VS Code中如何调试pytorch分布式训练脚本torch.distributed

## 一、问题描述

最近跑一些pytorch代码的时候遇到很多都是采用pytorch的分布式[torch](https://so.csdn.net/so/search?q=torch&spm=1001.2101.3001.7020).distributed来训练的，相比于传统的nn.DataParallel，使用分布式的训练方式可以显著提升GPU使用率， 从而加快训练速度。

一般常见的pytorch分布式训练命令如下：

```bash
$ export CUDA_VISIBLE_DEVICES=0,1



$ python -m torch.distributed.launch --nproc_per_node=2 tools/train.py --model bisenetv2
```

上述第一条命令用来设置当前环境变量中GPU数量。第二条命令为正式的启动脚本，该启动脚本与一般的使用类似python demo.py这种形式不同，整个python -m torch.[distributed](https://so.csdn.net/so/search?q=distributed&spm=1001.2101.3001.7020).launch为启动命令，实际的启动文件为：/home/dl/anaconda3/lib/python3.7/site-packages/torch/distributed/launch.py。也就是说该命令会自动的去调用torch包中的分布式启动文件launch.py来执行。后面的--nproc_per_node=2 tools/train.py --model bisenetv2全部为执行参数。

一般情况下，正式训练时会使用上述命令直接在终端中输入进行训练，但是，我们经常需要调试代码，一边运行一边查看中间变量，从而了解整个代码的运行机理。我个人喜欢用开源的VS Code，但是在VS Code中如何调试呢？谷歌了一下，有这种想法的人很多，但大多数都踩到了不同的坑里面，即使有解决方案也是用于pycharm的，为此本文专门进行了多次尝试，终于解决了这个问题，在此记录，以方便之后遇到这个问题的读者。

## 二、解决方案

在VS Code中想要调试Python脚本很简单，只需要创建一个launch.json文件即可。如果没有launch.json文件，只需要单机下图中“python：当前文件”旁的齿轮按钮即可创建一个launch.json文件。

![img](https://i-blog.csdnimg.cn/blog_migrate/8c5260af69182fcfd561118ebb37ddba.png)

下面是最关键的地方，用于为debug设置配置参数，具体如下：

```bash
{
	// Use IntelliSense to learn about possible attributes.
    // Hover to view descriptions of existing attributes.
    // For more information, visit: https://go.microsoft.com/fwlink/?linkid=830387
    "version": "0.2.0",
    "configurations": [ 
        {
            "name": "Python: 当前文件",
            "type": "python",
            "request": "launch",
            "program": "/home/dl/anaconda3/lib/python3.7/site-packages/torch/distributed/launch.py",//可执行文件路径
            "console": "integratedTerminal",
            "args": [
                "--nproc_per_node=1",
                "tools/train.py",
                "--model",
                "bisenetv1",
            ],
            "env": {"CUDA_VISIBLE_DEVICES":"0"},
        }
    ]
}
```

```bash
// 或者下面这种写法
{
	// Use IntelliSense to learn about possible attributes.
    // Hover to view descriptions of existing attributes.
    // For more information, visit: https://go.microsoft.com/fwlink/?linkid=830387
    "version": "0.2.0",
    "configurations": [ 
        {
            "name": "Python: 当前文件",
            "type": "python",
            "request": "launch",
            // "program": "/root/ModelZoo-PyTorch/PyTorch/built-in/cv/detection/YOLOV9_for_PyTorch/train_dual.py", //注意 这里不要设置program
      		"module": "torch.distributed.run", // 用torchrun 重点是这一行
            "console": "integratedTerminal",
            "args": [
                "--nproc_per_node=1",
                "tools/train.py",
                "--model",
                "bisenetv1",
            ],
            "env": {"CUDA_VISIBLE_DEVICES":"0"},
        }
    ]
}
```

其中，program参数用于设置使用torch分布式包中的launch.py文件来作为启动脚本，具体路径请参照具体的torch安装路径来修改。args用于设置每个参数。env用于设置环境变量。具体debug时，建议只用1个GPU来进行调试，所以nproc_per_node设置为1，CUDA_VISIBLE_DEVICES设置为0。

到这里如果直接按F5调式运行，可以勉强运行起来，如下所示：

![img](https://i-blog.csdnimg.cn/blog_migrate/2ecb19e06d420b93c4caa2814cf3bb86.png)

但是会发现一直卡在这里，然后在终端中输入命令nvidia-smi,效果如下：

![img](https://i-blog.csdnimg.cn/blog_migrate/b5c5eaf2a4ea7f9f218566bd9edc1c6e.png)

我的GPU没有一个正常运行起来的，每一个使用率全部是0%，但是这个时候程序也没有挂，也可以进行单步调试，通过单步调试发现问题出在这个地方：

```python
## train loop
for it, (im, lb) in enumerate(dl):
    im = im.cuda()
    lb = lb.cuda()
```

也就是在循环取数据的时候卡死在这一步了。然后在google继续查原因，终于发现问题了，在设置dataloader的时候，一般都会这么写：

```python
dl = DataLoader(
            ds,
            batch_size=batchsize,
            shuffle=shuffle,
            drop_last=drop_last,
            num_workers=4,
            pin_memory=True,
        )
    return dl
```

这里注意，有一个num_workers参数，如果是命令行运行，这样设置没有任何问题，但是在VS Code调试模式下这样设置就有问题了，google有人给出的解释是说“我们想通过这个参数来启动多个进程来加载数据，但是VS Code调试模型并不能成功的开启新进程”，到这里问题就变得显然了。只需要将num_workers设置为0即可。如下所示：

```python
dl = DataLoader(
            ds,
            batch_size=batchsize,
            shuffle=shuffle,
            drop_last=drop_last,
            num_workers=0,
            pin_memory=True,
        )
```

## 三、测试

按照步骤二进行相关修改，然后按F5启动调试，此时在终端中输入命令

```bash
nvidia-smi
```

查看GPU使用情况，如下图所示：

![img](https://i-blog.csdnimg.cn/blog_migrate/ea186b6f458af015679b5872df65b7da.png)

这个时候可以看到我的4块GPU中的第1块已经高负荷运行起来了，而且我也可以在VS Code中顺利的进行单步debug了。

