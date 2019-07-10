---
title: php代码审计小总结
date: '2017-07-14T22:00:50.000Z'
tags:
  - php
  - 代码审计
categories: Web安全
copyright: true
---

# PHP 代码审计小结

php代码审计小总结 

## 0x01 命令执行

### php代码执行

* eval\(\)
* assert\(\)
* preg\_replace + '/e'
* call\_user\_func\(\)
* call\_user\_func\_arra\(\)
* create\_function
* array\_map\(\)

### 系统命令执行

* system\(\)
* passthru\(\)
* exec\(\)
* pcntl\_exec\(\)
* shell\_exec\(\)
* popen\(\)
* proc\_open\(\)
* \`\(反单引号\)
* ob\_start\(\)
* escapeshellcmd\(\) // 该函数用于过滤

## 0x02 文件上传

* move\_uploaded\_file\(\)
* getimagesize\(\) //验证文件头只要为GIF89a，就会返回真

## 0x03 文件删除

* unlink\(\)
* session\_destroy\(\)

## 0x04 文件包含

### 本地文件包含

* require\(\)
* include\(\)
* include\_once\(\)
* require\_once\(\)

### 远程文件包含

* allow\_url\_include = on

## 0x05 文件读取

### 读文件

* hightlight\_file\($filename\);
* show\_source\($filename\);
* print\_r\(php\_strip\_whitespace\($filename\)\);
* print\_r\(file\_get\_contents\($filename\)\);
* readfile\($filename\);
* print\_r\(file\($filename\)\); // var\_dump
* fread\(fopen\($filename,"r"\), $size\);
* include\($filename\); // 非php代码
* include\_once\($filename\); // 非php代码
* require\($filename\); // 非php代码
* require\_once\($filename\); // 非php代码
* print\_r\(fread\(popen\("cat flag", "r"\), $size\)\);
* print\_r\(fgets\(fopen\($filename, "r"\)\)\); // 读取一行
* fpassthru\(fopen\($filename, "r"\)\); // 从当前位置一直读取到 EOF
* print\_r\(fgetcsv\(fopen\($filename,"r"\), $size\)\);
* print\_r\(fgetss\(fopen\($filename, "r"\)\)\); // 从文件指针中读取一行并过滤掉 HTML 标记
* print\_r\(fscanf\(fopen\("flag", "r"\),"%s"\)\);
* print\_r\(parse\_ini\_file\($filename\)\); // 失败时返回 false , 成功返回配置数组

### 列目录

* print\_r\(glob\("\*"\)\); // 列当前目录
* print\_r\(glob\("/\*"\)\); // 列根目录 print\_r\(scandir\("."\)\);
* print\_r\(scandir\("/"\)\);
* `$d=opendir(".");while(false!==($f=readdir($d))){echo"$f\n";}`
* `$d=dir(".");while(false!==($f=$d->read())){echo$f."\n";}`

### 超全局变量

* [$GLOBALS](http://php.net/manual/zh/language.variables.superglobals.php)

## 0x06 变量覆盖

* extract\(\)
* import\_request\_variables\(\)
* parse\_str\(\)
* mb\_parse\_str\(\)
* 全局变量覆盖：register\_globals为ON，$GLOBALS

## 0x07 php序列化函数

* serialize\(\)
* unserialize\(\)
* ini\_set\('session.serialize\_handler', 'php\_serialize'\);

## 0x08 Reference

* [代码审计入门总结](http://blog.neargle.com/SecNewsBak/drops/%E4%BB%A3%E7%A0%81%E5%AE%A1%E8%AE%A1%E5%85%A5%E9%97%A8%E6%80%BB%E7%BB%93.html)
* [php花式读取文件函数汇总](http://www.jianshu.com/p/33bc37ef72cc)
* [代码审计小结](https://chybeta.github.io/2017/07/14/php%E4%BB%A3%E7%A0%81%E5%AE%A1%E8%AE%A1%E5%B0%8F%E6%80%BB%E7%BB%93/)

