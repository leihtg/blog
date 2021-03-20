---
layout: post
title: python开发环境搭建
categories: python
tags: python conda
author: leihtg
---



* content
{:toc}


#### 安装conda

简介

> Conda 是一个开源的软件包管理系统和环境管理系统，用于安装多个版本的软件包及其依赖关系，并在它们之间轻松切换。 Conda 是为 Python 程序创建的，适用于 Linux，OS X 和Windows，也可以打包和分发其他软件。

下载地址

> https://docs.conda.io/en/latest/miniconda.html

安装完成之后就可以在环境中使用`conda`命令了

#### 使用conda创建python环境

> conda create -n tensorbase python=3.7.0

创建环境 `tensorbase` ，使用python的版本是3.7.0

设置镜像

```
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free/
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main/
conda config --set show_channel_urls yes

```

删除镜像命令

```
conda config --remove channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main/
```

进入刚才创建的环境

> activate tensorbase

1. 查看tensorflow各个版本

> anaconda search -t conda tensorflow

2. 查看详细信息

> anaconda show anaconda/tensorflow 

pip临时指定镜像

>  pip install -i https://pypi.tuna.tsinghua.edu.cn/simple --trusted-host pypi.tuna.tsinghua.edu.cn tensorflow==2.3.0

pip默认的配置路径 linux: `~/.pip/pip.conf`, win:  `%HOMEPATH%\pip\pip.ini`

```
[global] 
index-url = https://pypi.tuna.tsinghua.edu.cn/simple 
[install] 
trusted-host = pypi.tuna.tsinghua.edu.cn 
```



#### 使用jupyter记事本执行python

1. 安装jupyter

   > pip install jupyter

2. 运行jupyter

    ```bash
    juypter notebook    # 默认端口启动，8888
    jupyter notebook --port <port number>   # 指定端口启动
    jupyter notebook --no-browser   # 启动服务器但不在浏览器中打开
    ```

3. 生成配置文件

   > jupyter notebook --generate-config

   找到配置文件`jupyter_notebook_config.py`，设置常用选项

   ```bash
   #是否允许使用root用户
   c.NotebookApp.allow_root = False 
   #设置工作目录
   c.NotebookApp.notebook_dir = '/usr/local/games/.notebook'
   #允许远程登录
   c.NotebookApp.allow_remote_access = True
   c.NotebookApp.ip = '0.0.0.0'
   
   ```

4. ipykernel的安装与配置

    ```bash
    # 还在上一步中的虚拟环境中操作
    pip install ipykernel   # 安装ipykernel
    python -m ipykernel install --user --name=plotly    # 虚拟环境加入ipykernel中，name后接名称，最好和包中的内容相关，便于操作
    ```

5. Jupyter中Kernel相关

   ```bash
   jupyter kernelspec list # 列出juypter中所关联的所有kernel
   juypter kernelspec remove plotly    # 会卸载plotly的kernel
   ```

