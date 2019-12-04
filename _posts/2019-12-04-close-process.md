---
layout: post
title: 'Win10结束占用端口进程'
subtitle: ''
author: 'Max'
header-style: text
tags:
  - win10
---

- 查找对应的 PID

```
netstat -aon|findstr "${port}"
```

- 查找占用进程

```
tasklist|findstr "${PID}"
```

- 结束进程

  1. 打开资源管理器， 找到对应 PID 的进程， 直接结束（没有 PID 列，可以直接右键添加）
  2. 直接命令行结束

  ```
  taskkill /f /t /im 进程名
  ```
