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

## 4.端口未被占用，但是却提示端口无法使用

**问题记录**

**环境：Windows10**

### 问题如下：

> An attempt was made to access a [socket](https://so.csdn.net/so/search?q=socket&spm=1001.2101.3001.7020) in a way forbidden by its access permissions。

1.通过`netstat -aon|findstr “3306”`并没有看到端口占用

2.通过`netsh int ipv4 show dynamicport tcp`可以看到端口范围确实包含了3306端口

> 协议 tcp 动态端口范围
> 启动端口 : 1024
> 端口数 : 16383

### 原因

考虑是系统“TCP动态端口起始端口”配置问题。

### 解决步骤

1. 以管理员身份运行CMD

2. 关闭Hyper-V

   执行命令：`dism.exe /Online /Disable-Feature:Microsoft-Hyper-V`

   或采用传统方式，在控制面板的“程序与功能”中关闭。

   （这步完成后不要关机）

3. 修改动态端口范围

   `netsh int ipv4 set dynamicport tcp start=49152 num=16383`

   `netsh int ipv4 set dynamicport udp start=49152 num=16383`

4. 检查结果

   `netsh int ipv4 show dynamicport tcp`

   > 协议 tcp 动态端口范围
   > 启动端口 : 49152
   > 端口数 : 16383

5. 开启Hyper-V

   `dism.exe /Online /Enable-Feature:Microsoft-Hyper-V /All`

   输入`Y`确认重启计算机

   

## 5.“ERROR 1: PROJ: proj_create_from_database: Cannot find proj.db” 

这通常是由于GDAL在获取图像的投影信息时找不到proj.db文件引起的。

解决此问题的方法取决于您是使用conda还是pip安装的GDAL、PROJ和GEOS。下面提供了两种可能的解决方案：

解决方案1：如何您之前使用了conda安装的GDAL、PROJ和GEOS

打开命令提示符或Anaconda Prompt。
执行以下命令配置GDAL环境变量：

```
setx GDAL_DATA "C:\Anaconda3\envs\your_env_name\Library\share\gdal"
```

或者在代码中添加

```
os.environ['GDAL_DATA'] = r'C:\Anaconda3\envs\your_env_name\Library\share\gdal'
```

或者在系统环境变量path中添加：

```
'C:\Anaconda3\envs\your_env_name\Library\share\gdal'
```

将上面命令中的 your_env_name 替换为您的conda环境名称。
执行以下命令配置PROJ环境变量：

```
setx PROJ_LIB "C:\Anaconda3\envs\your_env_name\Library\share\proj"
```

或者在代码中添加

```
os.environ['PROJ_LIB'] = r“C:\Anaconda3\envs\network37\Library\share\proj"
```

或者在系统环境变量path中添加：

```
'C:\Anaconda3\envs\your_env_name\Library\share\proj'
```

同样，将上面命令中的 your_env_name 替换为您的conda环境名称。

## 6.“RuntimeError: main thread is not in main loop“的几种解决方案

#### 方法一（Tkinter）

最后写

```python
root.mainloop()
```

当然，如果不是root，则应使用Tk对象的名称代替root。

#### 方法二（多线程）

将线程设置为守护程序

```python
t = threading.Thread(target=your_func)
t.setDaemon(True)
t.start()
```

daemon默认是False，将其设置为True，再Start，就可以解决问题。daemon为True，就是我们平常理解的后台线程，用Ctrl-C关闭程序，所有后台线程都会被自动关闭。如果daemon属性是False，线程不会随主线程的结束而结束，这时如果线程访问主线程的资源，就会出错。

#### 方法三（matplotlib）

```python
# do this before importing pylab or pyplot
import matplotlib
matplotlib.use('Agg')
import matplotlib.pyplot as plt
```

## 7.关于windows安装wsl，出现WslRegisterDistribution failed with error: 0x8007019e The Windows Subsystem错误的解决方案

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/771d32fe960d35aa1507bf914ba262bf.png#pic_center)

### 解决方案

首先需要安装windows的子系统支持
步骤：
1、win+x，选择windows + powershell（管理员）

2、输入`Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Windows-Subsystem-Linux`（如下图所示）

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/e63d2351e73dcc9bc1066aff3a09d990.png#pic_center)

3、回车，输入Y或者y，然后系统重启

4、重启后打开ubuntu18.04的图标之后，会进入设置流程

### 其它问题解决方案

##### 复制黏贴

1、对wsl的terminal状态栏右键，选择属性（如下两张图）

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/1d71af51f430d2bdfaaa17584284ea3b.png#pic_center)

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/3dd6aa44a84264b4ec8affdb60b19d6b.png#pic_center)

2、需要把【将Ctrl+Shift+C/V用作复制/黏贴的快捷键】的选项打勾

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/6ce67961ccf8b21efe75a1d386951d25.png#pic_center)

## 8.cannot import name 'shard_checkpoint' from 'transformers.modeling_utils'

transformers包版本 4.46.3，将peft这个包升级为0.14.0，将sentence-transformers升级为3.4.1，autoawq为0.2.5

## 9.xinference.api.restful_api KeyError: ‘model.embed_tokens.weight‘

使用xinference运行qwen2选择8B量化运行时报错：[KeyError](https://so.csdn.net/so/search?q=KeyError&spm=1001.2101.3001.7020): [address=127.0.0.1:59995, pid=14340] 'model.embed_tokens.weight'

原因是：xinference在使用指定量化时，只能运行[bin文件](https://so.csdn.net/so/search?q=bin文件&spm=1001.2101.3001.7020)。而运行时生成的是safetensors文件

## 10. cannot import name 'shard_checkpoint'fn"transformers.modeling utils'

向报错的文件内加入此函数

```python
def shard_checkpoint(
    state_dict: Dict[str, torch.Tensor], max_shard_size: Union[int, str] = "10GB", weights_name: str = WEIGHTS_NAME
):
    """
    Splits a model state dictionary in sub-checkpoints so that the final size of each sub-checkpoint does not exceed a
    given size.

    The sub-checkpoints are determined by iterating through the `state_dict` in the order of its keys, so there is no
    optimization made to make each sub-checkpoint as close as possible to the maximum size passed. For example, if the
    limit is 10GB and we have weights of sizes [6GB, 6GB, 2GB, 6GB, 2GB, 2GB] they will get sharded as [6GB], [6+2GB],
    [6+2+2GB] and not [6+2+2GB], [6+2GB], [6GB].

    <Tip warning={true}>

    If one of the model's weight is bigger than `max_shard_size`, it will end up in its own sub-checkpoint which will
    have a size greater than `max_shard_size`.

    </Tip>

    Args:
        state_dict (`Dict[str, torch.Tensor]`): The state dictionary of a model to save.
        max_shard_size (`int` or `str`, *optional*, defaults to `"10GB"`):
            The maximum size of each sub-checkpoint. If expressed as a string, needs to be digits followed by a unit
            (like `"5MB"`).
        weights_name (`str`, *optional*, defaults to `"pytorch_model.bin"`):
            The name of the model save file.
    """
    logger.warning(
        "Note that `shard_checkpoint` is deprecated and will be removed in v4.44. We recommend you using "
        "split_torch_state_dict_into_shards from huggingface_hub library"
    )
    max_shard_size = convert_file_size_to_int(max_shard_size)

    sharded_state_dicts = [{}]
    last_block_size = 0
    total_size = 0
    storage_id_to_block = {}

    for key, weight in state_dict.items():
        # when bnb serialization is used the weights in the state dict can be strings
        # check: https://github.com/huggingface/transformers/pull/24416 for more details
        if isinstance(weight, str):
            continue
        else:
            storage_id = id_tensor_storage(weight)

        # If a `weight` shares the same underlying storage as another tensor, we put `weight` in the same `block`
        if storage_id in storage_id_to_block and weight.device != torch.device("meta"):
            block_id = storage_id_to_block[storage_id]
            sharded_state_dicts[block_id][key] = weight
            continue

        weight_size = weight.numel() * dtype_byte_size(weight.dtype)
        # If this weight is going to tip up over the maximal size, we split, but only if we have put at least one
        # weight in the current shard.
        if last_block_size + weight_size > max_shard_size and len(sharded_state_dicts[-1]) > 0:
            sharded_state_dicts.append({})
            last_block_size = 0

        sharded_state_dicts[-1][key] = weight
        last_block_size += weight_size
        total_size += weight_size
        storage_id_to_block[storage_id] = len(sharded_state_dicts) - 1

    # If we only have one shard, we return it
    if len(sharded_state_dicts) == 1:
        return {weights_name: sharded_state_dicts[0]}, None

    # Otherwise, let's build the index
    weight_map = {}
    shards = {}
    for idx, shard in enumerate(sharded_state_dicts):
        shard_file = weights_name.replace(".bin", f"-{idx+1:05d}-of-{len(sharded_state_dicts):05d}.bin")
        shard_file = shard_file.replace(
            ".safetensors", f"-{idx + 1:05d}-of-{len(sharded_state_dicts):05d}.safetensors"
        )
        shards[shard_file] = shard
        for key in shard.keys():
            weight_map[key] = shard_file

    # Add the metadata
    metadata = {"total_size": total_size}
    index = {"metadata": metadata, "weight_map": weight_map}
    return shards, index
```

## 11.关闭deepseekR1深度思考

[Help: 在ollama上运行时, 怎么才能让它不返回标签 ? When running on ollama, how can I make it not return the < think > tag? · Issue #23 · deepseek-ai/DeepSeek-R1](https://github.com/deepseek-ai/DeepSeek-R1/issues/23)github相关issue

vllm 已经支持了
https://docs.vllm.ai/en/latest/features/reasoning_outputs.html

亲测
`vllm serve /mnt/nas/deepseek-ai/DeepSeek-R1-Distill-Qwen-32B/ --dtype auto --port 8000 --tensor-parallel-size 2 --gpu-memory-utilization 0.95 --served-model-name Qwen-14B-Chat --max-model-len 32000 --enable-reasoning --reasoning-parser deepseek_r1`
可行

## 12.GLM-4V-9B TypeError: ChatGLMTokenizer._pad() got an unexpected keyword argument ‘padding_side‘

在模型文件 `glm-4v-9b/tokenization_chatglm.py` 中，在 `def _pad()` 方法中增加参数如下。

```python
def _pad(
            self,
            encoded_inputs: Union[Dict[str, EncodedInput], BatchEncoding],
            max_length: Optional[int] = None,
            padding_strategy: PaddingStrategy = PaddingStrategy.DO_NOT_PAD,
            padding_side:Optional[str] =None,#增加这个
            pad_to_multiple_of: Optional[int] = None,
            return_attention_mask: Optional[bool] = None,
    ) -> dict:
```

## 13.centos安装gmpy2

在CentOS上安装gmpy2库需要一些步骤，因为gmpy2依赖于一些底层的数学库如GMP、MPFR和MPC。以下是详细的步骤：

更新系统软件包：
首先确保你的系统软件包是最新的。

```
sudo yum update -y
```

安装依赖库：
安装gmpy2所需的开发工具和库。你需要安装GCC编译器以及GMP、MPFR和MPC库。

```
sudo yum groupinstall "Development Tools" -y
sudo yum install gmp-devel mpfr-devel libmpc-devel python3-devel -y
```

安装依赖：
确保已安装GMP和MPFR，因为MPC依赖于这两个库。

```
sudo yum install gmp-devel mpfr-devel
```

下载MPC源码：
访问MPC项目的官方网站或其GitHub页面找到最新版本的源码包，或者直接使用wget下载，例如：

```
wget https://ftp.gnu.org/gnu/mpc/mpc-1.3.1.tar.gz
```

解压源码包：

```
tar -xzf mpc-1.3.1.tar.gz
cd mpc-1.3.1
```

配置、编译和安装：
运行以下命令进行配置、编译和安装MPC。你可以通过--prefix指定安装路径，这里以默认路径为例。

```
./configure
make
sudo make install
```

安装gmpy2：

```
pip3 install gmpy2
```

## 14.解决：Could not find library geos_c or load any of its variants [‘libgeos_c.so.1‘, ‘libgeos_c.so‘]

### 错误：

```bash
OSError: Could not find library geos_c or load any of its variants ['libgeos_c.so.1', 'libgeos_c.so']
```

### 原因：

```bash
# Linux系统安装shapely后
pip3 install shapely
```



### 解决方法：

```bash
# ubuntu系统执行：
sudo apt-get install libgeos-dev
# CentOS系统执行：
sudo yum install geos-devel
```

## 15.python39 gradio启动 报错 TypeError: argument of type ‘bool‘ is not iterable

```
# pydantic版本问题，安装此版本
pip install pydantic==2.10.6
```

## 16.Linux下Gradio无法launch的解决方案

需要下载frcp，并放到gradio的目录下

>Could not create share link. Missing file: /home/gabriel/.local/lib/python3.9/site-packages/gradio/frpc_linux_aarch64_v0.2.  
Please check your internet connection. This can happen if your antivirus software blocks the download of this file. You can install manually by following these steps:  
```
1. Download this file: https:*//cdn-media.huggingface.co/frpc-gradio-0.2/frpc_linux_aarch64* 
2. Rename the downloaded file to: frpc_linux_aarch64_v0.2 
3. Move the file to this location:/home/gabriel/.local/lib/python3.9/site-packages/gradio
```

frpc_linux_aarch64名称不对，应该是frpc_linux_arm64

## 17.Fix patch error "Hunk #* FAILED at * (different line endings)

Apply Patch的时候后有时候会遇到诡异的问题，明明patch是对的，却打不上，提示如下错误：

```
Fix patch error "Hunk #* FAILED at * (different line endings)"
```

有一种可能是Windows和Uinix的文件line ending不同导致的，如果你是工作在Linux上，一个行之有效的解决方法是把Windows格式(dos)的文件转换为Unix

```bash
$ apt install -y dos2unix
$ dos2unix <file_name>                           #把指定文件转为unix格式     
$ find . -type f -exec dos2unix {} \;           #把当前目录下所有的文件转为unix格式
```

如果转化格式后，还打不上patch，可以提交修改之后再试，一般就可以成功了

```bash
$ git add .
$ git commit -m "<your comment>"
$ git am <your_patch>
```

## 18.Git忽略文件和操作系统尾部换行符问题

说我正在提交一个CRLF文件到仓库上，问我要怎么处理，这个CRLF其实是不同操作系统的尾部换行符的格式，CRLF是Carriage Return Line Feed的缩写，中文意思是回车换行，LF是Line Feed的缩写，中文意思是换行

假如你正在Windows上写程序，又或者你正在和其他人合作，他们在Windows上编程，而你却在其他系统上，在这些情况下，就可能会遇到行尾结束符问题，这是因为Windows使用回车和换行两个字符来结束一行，而Mac和Linux只使用换行符一个字符。虽然这是小问题，但它会极大地扰乱跨平台协作，在提交时产生非常多的冲突

对于不同的平台，有不同的处理方案：

1. Git可以在你提交时自动地把行结束符CRLF转换成LF，而在签出代码时把LF转换成CRLF。设置core.autocrlf来打开此项功能，如果是在Windows系统上，就把它设置成true，这样当签出代码时，LF会被转换成CRLF：

   ```csharp
   $ git config --global core.autocrlf true
   ```

2. Linux或Mac系统使用LF作为行结束符，因此你不想Git在签出文件时进行自动的转换；当一个以CRLF为行结束符的文件不小心被引入时你肯定想进行修正，把core.autocrlf设置成input来告诉Git在提交时把CRLF转换成LF，签出时不转换：

   ```python
   $ git config --global core.autocrlf input
   ```

这样会在Windows系统上的签出文件中保留CRLF，会在Mac和Linux系统上，包括仓库中保留LF。

1. 如果你是Windows程序员，且正在开发仅运行在Windows上的项目，可以设置false取消此功能，把回车符记录在库中：

   ```csharp
   $ git config --global core.autocrlf false
   ```

   上面三条总结起来就是下面的情形，x是你当前使用的系统所使用的换行方式

```cobol
1) true:             x -> LF -> CRLF
2) input:            x -> LF -> LF
3) false:            x -> x -> x
项目已经存在换行符不同的问题的解决方案：
```

- 如果当前开发有多个分支且各分支不同步，需要每个分支进行一次转换
- 如果只有一个分支或多个分支处于同一节点。可以从master切换一个新分支，进行转换，然后commit ，将此分支合并到所有分支。
- 将修改过的分支push到gitlab，让其他成员更新代码即可。
- Ps:由于每个人系统不同或者就是git的问题，可能出现更新完代码换行符不变，这时以服务器上的代码为准重新clone一份最新代码即可

## 19.解决 geos_c.dll 缺失问题
在使用 *pip install shapely* 安装 Shapely 库后，可能会遇到缺少 *geos_c.dll* 文件的错误。这通常是因为安装不完全导致的。

```
Could not find module 'C:\Users\Administrator\anaconda3\Library\bin\geos_c.dll' (or one of its dependencies). Try using the full path with constructor syntax
```

解决方法

1. 使用 Conda 安装 Shapely

最简单的方法是使用 Conda 来安装 Shapely，这样可以避免缺少 *geos_c.dll* 文件的问题[1]

```
conda install shapely
```

2. 手动下载并放置 geos_c.dll

如果你已经使用 *pip* 安装了 Shapely，但仍然遇到错误，可以手动下载 *geos_c.dll* 文件并将其放置在 Anaconda 的 Library/bin 目录下

**步骤如下：**
1. 访问 [dll-files.com](https://www.dll-files.com/geos_c.dll.html) 下载 *geos_c.dll* 文件。
2. 解压下载的文件。
3. 将 *geos_c.dll* 文件复制到 *C:\Users\Administrator\anaconda3\Library\bin* 目录下。
4. 重新导入 Shapely 包。

通过以上方法，你应该能够解决 *geos_c.dll* 缺失的问题，并成功运行 Shapely 库。

## 20.xinference启动报错Cluster is not available after multiple attempts

```bash
File "D:\devtools\anaconda3\envs\xrb\lib\site-packages\xinference\deploy\local.py", line 123, in main
raise RuntimeError("Cluster is not available after multiple attempts")
RuntimeError: Cluster is not available after multiple attempts
```

```bash
windows系统配置了如下环境变量解决了问题：
XINFERENCE_DISABLE_HEALTH_CHECK=1
XINFERENCE_DISABLE_METRICS=1
XINFERENCE_HEALTH_CHECK_ATTEMPTS=18
XINFERENCE_HEALTH_CHECK_INTERVAL=30
XINFERENCE_HEALTH_CHECK_TIMEOUT=30
```

## 21.xinference自1.5.1版本后Drop internal compression logic for `transformers` quantization, using bnb config instead

详情查看下面链接

[REF: Drop internal compression logic for `transformers` quantization, using bnb config instead by ChengjieLi28 · Pull Request #3324 · xorbitsai/inference](https://github.com/xorbitsai/inference/pull/3324)

新版`transformers` quantization eg:

> [!NOTE]
>
> 1.Remove internal compression logic for quantization in transformers engine
> 2.Remove all 4-bit and 8-bit in model specs in json. Now when model_format == "pytorch", only "none" quantization is provided by default.
> Now if you want to do quantization in transformers engine:
>
> See: https://huggingface.co/docs/transformers/en/main_classes/quantization#transformers.BitsAndBytesConfig for all the options.
>
> 3.Fix issues that all the vision / audio models cannot use transformers bnb quantization.

```python

model_uid = client.launch_model(
...
quantization_config={"load_in_4bit": True}
)
```
