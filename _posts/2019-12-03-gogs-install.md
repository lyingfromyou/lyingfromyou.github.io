---
layout: post
title: 'Docker安装gogs'
subtitle: ''
author: 'Max'
header-style: text
tags:
  - Docker
  - Gogs
---

> _前提：安装了 MySQL，新建一个 gogs 的数据库_

- 新建一个目录保存 gogs 数据

```
sudo mkdir /opt/docker/gogs
```

- 然后一条命令搞定 gogs

```
sudo docker run  -v /usr/share/zoneinfo/Asia/Shanghai:/etc/localtime  -v /opt/docker/gogs:/data -p 10022:22 -p 10080:3000 --name=gogs --privileged=true --restart=always -d gogs/gogs
```

- 然后我们直接访问 gogs, (你的 IP):10080, 可以看到安装界面

![](..\img\install-gogs\install-gogs-1.png)

修改数据库主机为 ： (虚拟机 IP) : (MySQL 端口号)

---

- 修改 Docker 配置的映射端口

![](..\img\install-gogs\install-gogs-2.png)

接下来的选项根据自己需要来配置就 OK 了，添加一个管理员账号之后，点击完成安装就可以看到 gogs 的界面了。

---

![](..\img\install-gogs\install-gogs-3.png)
