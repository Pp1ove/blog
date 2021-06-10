---
title: ssrf
author: Pp1ove
avatar: \img\custom\head.jpg
authorLink: 
authorAbout: Pp1ove
authorDesc: 一个好奇的人
categories: 技术
comments: false
date: 2021-01-24 09:48:04
tags: ssrf
photos: \img\article\12.png
---

## 常用协议

```
file:///
dict://
sftp://
ldap://
tftp://
gopher://
```

参考链接:https://www.cnblogs.com/-mo-/p/11673190.html

gopher协议:https://blog.csdn.net/qq_45089570/article/details/109643457

https://blog.csdn.net/Z_Grant/article/details/102913575?utm_medium=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-1.control&depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-1.control

https://www.cnblogs.com/flokz/p/11610048.html#autoid-3-0-0

### 引起SSRF漏洞的函数

```php
file_get_contents()
获取文件内容
fscockopen()
打开一个网络连接或者一个Unix套接字连接
curl_exec()
执行curl绘画,最好用的,几乎可以用来做各种ssrf攻击
```

#### redis

下载redis,修改



https://www.evi1s.com/archives/140/