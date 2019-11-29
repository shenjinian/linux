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

**3.1 命令格式**

在`conda`环境中，常用的命令格式为：

```
conda [命令 [参数]] 
```

#### 3.2 包管理

与`python -m pip list`类似，conda可以列出当前环境下的所有包：

```
conda list
```

#### 3.3 版本与升级

`conda`有一套特别的机制，用于管理和维护依赖库之间的关系。在不同版本的`conda`中，我们可以直接使用的Python与依赖库的版本都不同，为了确定当前使用的`conda`版本，可以运行以下命令：

```
conda --version
```

有时，我们想用的某个库在`conda`中有问题，或者默认模块安装的版本比较旧，可以先尝试升级解决：

```
conda update conda
```

#### 4. 虚拟环境的创建与管理