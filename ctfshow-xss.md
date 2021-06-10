---
title: ctfshow-xss
author: Pp1ove
avatar: 'https://wx1.sinaimg.cn/large/006bYVyvgy1ftand2qurdj303c03cdfv.jpg'
authorLink: hojun.cn
authorAbout: 一个好奇的人
authorDesc: 一个好奇的人
categories: 技术
comments: true
date: 2021-02-24 12:23:28
tags:
keywords:
description:
photos:
---



# web316

反射性xss,

![image-20210224122408795](ctfshow-xss/image-20210224122408795.png)

在xss测试平台（https://xss.pt/xss.php）获取flag

# web317/318/319

317屏蔽了script标签,318屏蔽了img标签

但是还是有其他标签可以弹窗，如<iframe>标签,详细可以看:https://xz.aliyun.com/t/4067#toc-3

https://www.freebuf.com/articles/web/157589.html

具体做题就用平台中没有script和img标签的代码就可以了

但是317没出,可能是平台的问题

# web320/321/322

测了半天都没成功,原来是过滤了空格

用tab代替空格就可以了

# web323