---
layout: post
title: 'MongoDB数据导入导出'
subtitle: '把本地Mongodb中的数据导入(批量插入)到服务器的数据库中'
author: 'Max'
header-style: text
tags:
  - Mongo DB
---

## MongoDB 数据导入导出

- ##### 导出数据

```
mongoexport -d admin -c users -o outdatafile.json

# 选项解释:
# -d: 指明使用的库, 本例中为” admin”
# -c: 指明要导出的表, 本例中为”users”
# -o: 指明要导出的文件名, 本例中为”outdatafile.json”
# 如果出现error connecting to db server: no reachable servers错误，将绑定的ip解绑即可
```

- ##### 连接远程数据库并导入

```
mongoimport -h ${ip} -d admin -c users --file ./outdatafile.json --upsert

# 选项解释:
# -d: 指定把数据导入到哪一个数据库中
# -c: 指定把数据导入到哪一个集合中
# --type: 指定导入的数据类型
# --file: 指定从哪一个文件中导入数据
# --headerline: 仅适用于导入csv,tsv格式的数据，表示文件中的第一行作为数据头
# --upsert: 以新增或者更新的方式来导入数据
```
