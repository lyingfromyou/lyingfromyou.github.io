---
layout: post
title: 'Centos7 查找 JDK 地址'
subtitle: 'Centos7 查找 JDK 地址'
author: 'Max'
header-style: text
tags:
  - Linux
  - Centos7
---

# Centos7 查找 JDK 地址

1. 找到 java 执行文件路径

   ```
   whereis java
   ```

   > 输出> java: /usr/bin/java

2. 根据执行执行文件找到对应的软连接指向的文件路径

   ```
   ls -lrt /usr/bin/java
   ```

   > 输出> /usr/bin/java -> /etc/alternatives/java

3. 进入 `/etc/alternatives` 目录，发现还不是源文件目录，继续找

   ```
   ls -lrt /etc/alternatives/java
   ```

   > 输出> /etc/alternatives/java -> /usr/java/jdk1.8.0_231-amd64/jre/bin/java



> #### /usr/java/jdk1.8.0_231-amd64 就是jdk的路径了



