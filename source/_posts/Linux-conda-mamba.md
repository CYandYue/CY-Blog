---
title: 用Mamba代替Conda -- 介于UV和Conda之间的解法
author: CYandYUE
index_img: /index_img/35_conda_mamba.png
date: 2026-05-29 21:22:04
tags:
- Linux
- Env
categories:
- Linux
- Env
---

## 0. Motivation
最近在服务器上配环境的时候，又被 `conda install` 卡住了

尤其是装 CUDA, PyTorch, conda-forge 这种依赖比较复杂的包时，conda 可能会在 `Solving environment` 那一步想很久

后来发现其实可以直接用 `mamba`

简单理解：`mamba` 就是一个更快的 `conda`

## 1. Overall
`mamba` 和 `conda` 用的是同一套环境、同一套 package、同一套 channel

也就是说它不会重新发明一套 Python 环境管理方式

你原来的 conda env 还是那些 env，只是把很多命令里的 `conda` 换成 `mamba`

比如：
```bash
conda install -n conceptgraph -c nvidia cuda-nvcc=11.8.89
```

可以写成：
```bash
mamba install -n conceptgraph -c nvidia cuda-nvcc=11.8.89
```

它快主要是因为依赖求解器是 C++ 写的，并且下载、处理 repodata 等步骤也更激进一些

所以在依赖关系很复杂的时候，体验会明显好很多

## 2. Install
如果已经装好了 miniconda/anaconda，最简单的方法是在 `base` 环境里装：

```bash
conda install -n base -c conda-forge mamba
```

装完之后可以确认一下：

```bash
mamba --version
which mamba
```

之后不需要每次都 `conda activate base`

只要当前 shell 已经能用 `conda`，一般就可以直接用 `mamba`

## 3. Command lines
我举一些例子，其实就是command里把conda换成mamba

创建环境：

```bash
mamba create -n env_name python=3.10
```

激活环境：

```bash
conda activate env_name
```

安装包：

```bash
mamba install numpy pandas matplotlib
```

给指定环境安装包：

```bash
mamba install -n env_name numpy
```



## 4. Notes
`mamba` 不是 pip 的替代品

如果一个包本来就是用 pip 装的，还是用 pip

```bash
mamba install python cudatoolkit pytorch numpy
pip install some_python_only_package
```

也就是说，底层依赖、CUDA、PyTorch 这些优先用 `mamba/conda`

纯 Python 包或者 conda 里没有的包再用 `pip`

另外尽量不要混太多 channel

比如一个环境里一会儿 `defaults`，一会儿 `conda-forge`，一会儿 `nvidia`

不是不能用，但依赖冲突和求解变慢的概率会更高

如果主要用 conda-forge，可以设置 strict channel priority：

```bash
conda config --set channel_priority strict
```

## 5. micromamba
还有一个更轻量的版本叫 `micromamba`

它不依赖 conda 本体，是一个独立的可执行文件

适合 Docker、CI、或者从零开始搭一个很干净的环境

但如果你已经有 miniconda/anaconda，并且只是想让装包更快，用 `mamba` 就够了

## 6. Reference
- [Mamba Installation](https://mamba.readthedocs.io/en/stable/installation/mamba-installation.html)
- [Mamba User Guide](https://mamba.readthedocs.io/en/stable/user_guide/mamba.html)
- [Micromamba Installation](https://mamba.readthedocs.io/en/stable/installation/micromamba-installation.html)
- [Miniforge](https://github.com/conda-forge/miniforge)
