---
title: mongoDB搭建与配置
date: 2016-06-12 14:17:06
tags:
- MongoDB
categories:
- 数据库

---

### 下载及安装

1.1 在下面的地址选择适合自己的系统下载：
[MongDB下载官网](https://www.mongodb.com/download-center?jmp=docs&_ga=1.240239499.942098647.1465695750#community )
1.2 将zip文件解压放到盘符的根目录（如C：或D：），为了方便建议文件夹命名尽量简短如（d:\mongodb)
1.3 创建数据库文件的存放位置，比如d:/mongodb/data/db。启动mongodb服务之前需要必须创建数据库文件的存放文件夹，否则命令不会自动创建，而且不能启动成功。
1.4 打开cmd（windows键+r输入cmd）命令行，进入D:\mongodb\bin目录（如图先输入d:进入d盘然后输入cd d:\mongodb\bin），
输入如下的命令启动mongodb服务：

	D:/mongodb/bin>mongod --dbpath D:\mongodb\data\db


### 安装服务
2.1 在d:\mongodb\data下新建文件夹log（存放日志文件）并且新建文件mongodb.log
![文件夹相对位置](/uploads/6.mongo.png "文件夹相对位置") 
2.2 进入：D:/mongodb/bin输入

	mongod -dbpath "d:\mongodb\data\db" -logpath "d:\mongodb\data\log\mongodb.log" -install -serviceName "MongoDB"（这里必须以管理员身份运行）

此时服务已经安装成功，运行

	>net start mongodb   (开启服务)

	>net stop mongodb   (关闭服务)

2.3 删除MongoDB Service

	mongod -dbpath "d:\mongodb\data\db" -logpath "d:\mongodb\data\log\mongodb.log" -remove -serviceName "MongoDB";

### 基本操作
以下命令都是要进入mongodb shell才能操作，cmd进入：D:/mongodb/bin输入下mongo

3.1 新建数据库RealTimeClass

	use RealTimeClass;
	
3.2 新建用户并给与权限
	
	db.createUser({"user":"root","pwd":"123456","roles":[{role:"dbOwner",db:"RealTimeClass"}]});
	
注意一点，帐号是跟着库走的，所以在指定库里授权，必须也在指定库里验证(auth)。

```bash

> use admin
switched to db admin
> db.createUser(
...   {
...     user: "root",
...     pwd: "123456",
...     roles: [ { role: "userAdminAnyDatabase", db: "admin" } ]
...   }
... )
Successfully added user: {
    "user" : "dba",
    "roles" : [
        {
            "role" : "userAdminAnyDatabase",
            "db" : "admin"
        }
    ]
}

```
