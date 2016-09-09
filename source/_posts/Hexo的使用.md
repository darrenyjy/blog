---
title: Hexo的使用
date: 2016-05-25 20:59:06
tags:
- Hexo
categories:
- Hexo

---

# Hexo的使用

### 让搜索引擎收录
利用文件验证的时候记得在下载的html前面加上
		
	layout: false  //分号后面一定要有空格

### 部署步骤

每次部署的步骤，可按以下三步来进行。
```
	hexo clean
	hexo generate
	hexo deploy
```

### 一些常用命令：

* hexo new "postName" #新建文章

* hexo new page"pageName" #新建页面

* hexo generate #生成静态页面至public目录

* hexo server #开启预览访问端口（默认端口4000，'ctrl + c'关闭server）

* hexo deploy #将.deploy目录部署到GitHub

