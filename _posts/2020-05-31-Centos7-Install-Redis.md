---
layout: post
title: 'Centos7安装Redis'
subtitle: 'Centos7安装Redis'
author: 'Max'
header-style: text
tags:
  - Redis
---

# Centos7 下面安装 Redis

### 更换 yum 源

1. 安装 wget

   ```
   yum install wget -y
   ```

2. 打开 mirrors.aliyun.com，选择 centos 的系统

3. 按照文档操作

4. 更新 yum

   ```
   yum update -y
   ```

### 下载安装 Redis

1. 在根目录上新建文件夹 **soft**

   ```
   mkdir soft
   ```

2. 在 soft 目录下 下载 redis (版本可在 redis 官网选择)

   ```
   wget http://download.redis.io/releases/redis-6.0.4.tar.gz
   ```

3. 解压 redis

   ```
   tar xf redis-6.0.4.tar.gz
   ```

4. 在 redis 的目录下执行 **make**

   ```
   make
   ```

   1. 如果出现 "/bin/sh: cc: 未找到命令" , 安装 gcc. [参考这篇](https://www.cnblogs.com/blackBlog/p/12924891.html)

      ```
      # 升级到gcc 9.3：
      yum -y install centos-release-scl
      yum -y install devtoolset-9-gcc devtoolset-9-gcc-c++ devtoolset-9-binutils
      scl enable devtoolset-9 bash
      # 需要注意的是scl命令启用只是临时的，退出shell或重启就会恢复原系统gcc版本。
      # 如果要长期使用gcc 9.3的话：

      echo "source /opt/rh/devtoolset-9/enable" >>/etc/profile
      # 这样退出shell重新打开就是新版的gcc了
      ```

   2. 执行 **make distclean** 清除之前 make 错误

   3. 在 src 继续执行 **make**

5. 执行 **make test** 测试 (不测试的话, 可以不用装这个)

   1. 如果出现 "You need tcl 8.5 or newer in order to run the Redis test", 安装 tcl

      ```
      yum install tcl -y
      ```

6. cd 到 src 目录下, 可以看到生成了可执行程序, 执行 **./redis-server** 能执行成功并出现 redis 启动页面, 说明编译成功

7. 安装到自定义文件夹, 在 src 目录下执行

   ```
   # 会在redis6文件夹下生成bin目录, 包含执行程序
   make install PREFIX=/opt/soft/redis6
   ```

8. 设置环境变量

   ```
   vim /etc/profile

   export REDIS_HOME=/opt/soft/redis6
   export PATH=$PATH:$REDIS_HOME/bin

   source /etc/profile
   ```

9. 安装成服务

   ```
   cd /root/soft/redis-6.0.4/utils/
   ./install_server.sh (可以执行一次或多次)
   ```

   1. 一个物理机中可以有多个 redis 实例(进程), 通过 port 区分

   2. 可执行程序就一份在目录, 但是内存中的多个实例需要各自的配置文件, 持久化目录等资源

   3. service redis_6379 start/stop/status > linux /etc/init.d/\*\*\*

   4. 执行 ./install. server.sh 报错, 把 **install. server.sh** 中 #bail if this system is managed by systemd 开头的这一段代码全部注释掉就好

      ```
      This systems seems to use systemd.
      Please take a look at the provided example service unit files in this directory, and adapt and install them. Sorry!
      ```
