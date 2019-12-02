### Miniconda管理 Python 版本和虚拟环境



#### 1. 概念：

- Conda：包、依赖和环境管理器。
- Anaconda（某种蟒蛇的名字）：面向数据科学的 Python 发行版，包含 conda、conda-build、Python 和 100+ 常用的数据科学常用的库及其依赖。
- Miniconda：精简版的 Anaconda，也是一个 Python 发行版，只包含 conda、Python 和一些基本的包。

#### 2. Conda安装

 Miniconda 安装包可以到 https://mirrors.tuna.tsinghua.edu.cn/anaconda/miniconda/ 下载。 

首先利用`wget`下载安装脚本文件：

```
wget http://mirrors.tuna.tsinghua.edu.cn/anaconda/miniconda/Miniconda-1.6.0-Linux-x86_64.sh
```

利用`chmod`命令修改sh文件为可执行文件，然后运行安装脚本：

```
chmod 755 Miniconda3-latest-Linux-x86_64.sh
sh Miniconda3-latest-Linux-x86_64.sh
```

在出现的提示界面中，根据提示选择yes或no。一般来说，我们保持默认即可，但需要留意下最后一步会自动在`.bashrc`文件添加`conda`的`PATH`路径。如果`conda`的环境存在与你日常使用的程序有冲突的命令，就有可能会出现问题。

当然，还有一种方式是在添加`PATH`路径时选择no，然后在每次需要conda的时候手动找到conda下的`active`命令激活下。这种方式比较灵活，如果不嫌麻烦建议使用这种方式。

> 注意不要把激活conda与激活虚拟环境搞混。

#### 3  Conda常用命令

##### 3.1 命令格式

在`conda`环境中，常用的命令格式为：

```
conda [命令 [参数]] 
```

##### 3.2 包管理

与`python -m pip list`类似，conda可以列出当前环境下的所有包：

```
conda list
```

##### 3.3 版本与升级

`conda`有一套特别的机制，用于管理和维护依赖库之间的关系。在不同版本的`conda`中，我们可以直接使用的Python与依赖库的版本都不同，为了确定当前使用的`conda`版本，可以运行以下命令：

```
conda --version
```

有时，我们想用的某个库在`conda`中有问题，或者默认模块安装的版本比较旧，可以先尝试升级解决：

```
conda update conda
```

#### 4. 虚拟环境的创建与管理

##### 4.1 创建虚拟环境 

使用 conda create 命令创建虚拟环境，使用 --name 选项（-n）指定虚拟环境名称：

```text
$ conda create -n nginx_log
```

使用 conda info 命令查看当前环境名称、Python 版本、虚拟环境文件夹位置、Conda 版本等等各种信息：

```text
$ conda info
```

使用 --envs 选项（-e）查看所有已创建的虚拟环境。在列出的虚拟环境中，使用星号（*）标识的是当前激活的虚拟环境：

```text
$ conda info -e
```

##### 4.2 激活虚拟环境

使用 conda activate 命令激活虚拟环境，添加虚拟环境名作为参数：

```text
$ conda activate nginx_log
```

激活以后会在命令行提示器前显示虚拟环境名称，比如：

```text
(nginx_log) $
```

不添加虚拟环境名称，就会重新激活基础环境（base）：

```text
$ conda activate
```

##### 4.3 退出虚拟环境

```
$ conda deactivate
```

##### 4.4 设置虚拟环境的 Python 版本

在创建虚拟环境的时候，可以使用 python 参数指定 Python 版本。假设你使用的 Miniconda 默认版本是 Python 3.7，如果你想创建一个 Python 2.7 的虚拟环境，使用下面的命令：  

```text
$ conda create -n snakes python=2.7
```

你可以在虚拟环境名加上标识方便识别 Python 版本，比如：

```text
$ conda create -n snakes-py27 python=2.7
```

##### 4.5 在虚拟环境下，安装依赖

激活一个Conda的虚拟环境后，安装依赖主要用以下命令：

```
conda install pandas
```

这条命令主要从默认的频道中去寻找pandas软件包。要注意，Conda里有频道的概念，类似电视机买回来一般都有个默认频道一样，默认的Conda有一个`defaults`的频道。频道即下载源。

##### 4.6 更改下载源

```
vim  /root/.condarc
```

```
channels:
  - defaults
show_channel_urls: true
default_channels:
  - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main
  - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free
  - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/r
custom_channels:
  conda-forge: https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud
  msys2: https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud
  bioconda: https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud
  menpo: https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud
  pytorch: https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud
  simpleitk: https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud
```

即可添加 Anaconda Python 免费仓库。Windows 用户无法直接创建名为 `.condarc` 的文件，可先执行 `conda config --set show_channel_urls yes` 生成该文件之后再修改。

配置完成后，可以通过下面命令来确认是否配置成功：

```
conda config --show
```

#### 5. 建议直接使用 pip来管理依赖

因为 Conda 安装库的时候默认使用 Conda 自己的仓库，这里包含的 720 多个库除了流行的 Python 包外大多是数据科学相关的包。更好的选择是使用官方 PyPI 仓库，这样可以确保你使用到最近更新的包，而且不会出现有些包找不到的情况。

我们要做的就是只用 Conda 的 Python 版本和虚拟环境管理功能，不用它来管理依赖。依赖管理（安装、卸载、更新等）仍然使用 pip 进行，或是进一步搭配 pip-tools 来管理依赖。

你需要在 conda 环境内使用下面的命令安装 pip：

```text
$ conda install pip
```

这样在执行 pip 命令时会使用虚拟环境内的 pip，而不是系统全局的 pip。这样做的副作用是会产生几个多余的依赖。

#### 6. Conda环境导出与恢复

Conda支持直接导出环境，命令如下：

```
conda env export > env.yml
```

这里，推荐在熟悉的情况下，去掉二级依赖库(依赖的依赖)。一方面减少文件内容，第二有可能二级依赖在后面会被取消。

环境恢复使用命令：

```
conda env create -n revtest -f=/tmp/env.yml
```

这里比较关键是导出的yaml文件，通过编译器查看可知，其是一个标准的yaml文件。

#### 7.  总结

##### 7.1 conda和pip区别

- conda还负责依赖检查和维护。Conda不仅仅安装Python库这么简单，他还能把Python库需要的外部依赖也同时安装进来，并且维护每个软件库对应的各种依赖版本关系，每次conda安装都要进行比较复杂的处理来维护好依赖关系。
- conda这个包管理命令不仅仅可以用在Python上，还可以用来管理R等其他语言。
- 不能提供egg或whl时，pip只能从源代码编译。而`conda install`一直都是安装编译好的二进制。
- conda默认就支持虚拟环境；而pip是靠`virtualenv`或`venv`来支持
- conda是Python的外部工具
- conda的托管网站是Anaconda，而pip的托管网站是PyPI（https://pypi.org/）

##### 7.2 Miniconda 的优缺点

**优点**

- 用法简单，易于上手
- pyenv+virtualenv/venv+virtualenvwrapper 工具组合的替代品
- 类似 virtualenvwrapper，可以在任意位置激活虚拟环境，而不是必须在项目根目录
- 支持管理不同的 Python 版本
- 兼容性很好，支持 Windows

**缺点**

- conda 和 pip/Poetry 组合存在潜在的冲突，但情况在改善
- 在 Windows 下，需要使用专用的命令行程序
- 依赖管理比较弱，需要搭配 pip/Poetry 来使用

既然一个 Miniconda 就能很好的胜任 Python 版本管理和虚拟环境管理的任务，为什么要用 pyenv+virtualenv/venv+virtualenvwrapper 呢？遗憾的就是依赖管理功能不够完善，和 pip/Poetry 搭配使用则可能会有潜在的冲突。所以，推荐觉得 virtualenv/venv+pip 搭配太麻烦的初学者使用；推荐能接受 Conda+pip/Poetry 这种搭配的人使用；推荐使用 Python 做数据科学相关工作的人使用。

对比之下，最稳定的解决方案大概还是 virtualenv/venv+pip+其他工具……