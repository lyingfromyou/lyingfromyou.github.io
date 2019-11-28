---
layout: post
title: '搭建Docker环境'
subtitle: '安装、卸载、配置Docker'
author: 'Max'
header-style: text
tags:
  - Docker
  - liunx
---

### 先决条件

#### 1. 首先卸载旧版本

    sudo yum remove docker \
                      docker-client \
                      docker-client-latest \
                      docker-common \
                      docker-latest \
                      docker-latest-logrotate \
                      docker-logrotate \
                      docker-engine

### 安装 docker

#### 1. 使用 yum 安装

- 设置存储库

  1. 安装所需的软件包

     ```
     sudo yum install -y yum-utils \
       device-mapper-persistent-data \
       lvm2
     ```

  2. 设置稳定的存储库

     ```
     sudo yum-config-manager \
         --add-repo \
         https://download.docker.com/linux/centos/docker-ce.repo
     ```

  3. 启用

     ```
     sudo yum-config-manager --enable docker-ce-nightly
     ```

- 安装`Docker engine`

  1. 安装最新版`docker`

     ```
     sudo yum install docker-ce docker-ce-cli containerd.io
     ```

  2. 安装指定版本`docker`

     1. 首先查看可用的版本

        ```
        yum list docker-ce --showduplicates | sort -r
        ```

     2. 安装指定版本的`docker`

        ```
        sudo yum install docker-ce-<VERSION_STRING> docker-ce-cli-<VERSION_STRING> containerd.io
        ```

        例如:

        ```
        sudo yum install docker-ce-19.03.5 docker-ce-cli-19.03.5 containerd.io
        ```

#### 2. 从软件包安装

1.  转到`https://download.docker.com/linux/centos/7/x86_64/stable/Packages/` 并下载`.rpm`要安装的 Docker 版本的文件。

2.  安装`Docker Engine-Community`，下面的路径更改为下载`Docker`软件包的路径。

```
sudo yum install /path/to/package.rpm
```

### 启动测试 docker

#### 1. 启动 docker

```
sudo systemctl start docker
```

#### 2. 通过运行 `hello-world` 映像来验证是否正确安装

```
docker run hello-world
```

### 卸载 Docker Engine

#### 1. 卸载 Docker 软件包

```
sudo yum remove docker-ce
```

#### 2. 删除容器和镜像等数据

```
sudo rm -rf /var/lib/docker
```

### 配置 docker 阿里镜像加速

```
sudo vim /etc/docker/daemon.json
```

在`daemon.json`中键入以下内容

```
{
  "registry-mirrors": ["https://b0kifnqu.mirror.aliyuncs.com"]
}
```

然后保存退出，通过:

```
 sudo systemctl restart docker
```

重启`docker`，之后通过:

```
sudo docker info
```

查看`docker`当前的配置，如果能够在打印出来的信息中看到：

```
 Registry Mirrors:
  https://b0kifnqu.mirror.aliyuncs.com/
```

那么就证明配置成功

设置`docker`开机自启动

```
sudo systemctl enable docker
```
