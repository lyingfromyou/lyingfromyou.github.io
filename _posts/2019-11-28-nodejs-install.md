---
layout: post
title: '搭建NodeJS环境'
subtitle: 'Liunx搭建Node环境并配置镜像'
author: 'Max'
header-style: text
tags:
  - node
  - liunx
---

### 1. 安装 node.js

#### 1.1 下载并解压 node 包

这里采用的是 node 的二进制安装包，安装位置我将会采用/opt/node 这个路径，该路径不存在的
话可以直接通过:

    sudo mkdir /opt/node

新建该路径。
cd 到/opt/node 之后，直接通过命令：

    sudo wget https://nodejs.org/dist/v10.10.0/node-v10.10.0-linux-x64.tar.xz

就可以完成 node 二进制包的下载。
然后通过:

    sudo tar -xvf node-v10.10.0-linux-x64.tar.xz

处理一下目录

    sudo mv node-v10.10.0-linux-x64/* /opt/node|sudo rm -rf node-v10.10.0-linux-x64 node-v10.10.0-linux-x64.tar.xz

这样，node 的 Linux 二进制文件都应该位于/opt/node 这个路径下了

#### 1.2 配置 node 环境变量

输入：

    sudo vim /etc/profile

编辑全局的环境，添加环境变量：

    export NODE_HOME=/opt/node
    export PATH=$PATH:$NODE_HOME/bin
    export NODE_PATH=$PATH:$NODE_HOME/lib/node_modules

之后通过：

    source /etc/profile

使当前更改生效即可。

---

#### Tips:

上面的设定是全局有效的，可能有人不喜欢这种方式，那么直接使用 vim ~/.bash_profile 编辑当前用户的配置即可。
这样,node 的安装基本上就算完事了，想要确认的话，输入

    npm -v

或者

    node -v

能够看到输出基本上就可以确定安装没有问题。

### npm 镜像配置

npm 的话一条命令就可以搞定：

    npm config set registry https://registry.npm.taobao.org

通过以下命令确定配置是否生效:

    npm config get registry
