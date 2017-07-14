---
title: php代码审计小总结
author: chybeta
categories: Web安全
---

<!-- more -->
# 命令执行
## php代码执行
+ eval()
+ assert()
+ preg_replace + '/e'
+ call_user_func()
+ call_user_func_arra()
+ create_function
+ array_map()

## 系统命令执行
+ system()
+ passthru()
+ exec()
+ pcntl_exec()
+ shell_exec()
+ popen()
+ proc_open()
+ `(反单引号)
+ ob_start()
+ escapeshellcmd() // 该函数用于过滤

# 文件包含
# 本地文件包含
+ require()
+ include()
+ include_once()
+ require_once()

## 远程文件包含
+ allow_url_include = on

# 读取文件函数

## 读文件

+ hightlight_file($filename);
+ show_source($filename);
+ print_r(php_strip_whitespace($filename));
+ print_r(file_get_contents($filename));
+ readfile($filename);
+ print_r(file($filename)); // var_dump
+ fread(fopen($filename,"r"), $size);
+ include($filename); // 非php代码
+ include_once($filename); // 非php代码
+ require($filename); // 非php代码
+ require_once($filename); // 非php代码
+ print_r(fread(popen("cat flag", "r"), $size));
+ print_r(fgets(fopen($filename, "r"))); // 读取一行
+ fpassthru(fopen($filename, "r")); // 从当前位置一直读取到 EOF
+ print_r(fgetcsv(fopen($filename,"r"), $size));
+ print_r(fgetss(fopen($filename, "r"))); // 从文件指针中读取一行并过滤掉 HTML 标记
+ print_r(fscanf(fopen("flag", "r"),"%s"));
+ print_r(parse_ini_file($filename)); // 失败时返回 false , 成功返回配置数组


## 列目录
+ print_r(glob("*")); // 列当前目录
+ print_r(glob("/*")); // 列根目录 print_r(scandir("."));
+ print_r(scandir("/"));
+ `$d=opendir(".");while(false!==($f=readdir($d))){echo"$f\n";}`
+ `$d=dir(".");while(false!==($f=$d->read())){echo$f."\n";}`


## 超全局变量
+ [$GLOBALS](http://php.net/manual/zh/language.variables.superglobals.php)

# php序列化函数
+ serialize()
+ unserialize()
+ ini_set('session.serialize_handler', 'php_serialize');

# 文件上传
+ move_uploaded_file()
+ getimagesize() //验证文件头只要为GIF89a，就会返回真

# 变量覆盖
+ extract()
+ import_request_variables()
+ parse_str()
+ mb_parse_str()
+ 全局变量覆盖：register_globals为ON，$GLOBALS


# 文件删除
+ unlink()
+ session_destroy()

# Reference
+ [代码审计入门总结](http://blog.neargle.com/SecNewsBak/drops/%E4%BB%A3%E7%A0%81%E5%AE%A1%E8%AE%A1%E5%85%A5%E9%97%A8%E6%80%BB%E7%BB%93.html)
+ [php花式读取文件函数汇总](http://www.jianshu.com/p/33bc37ef72cc)
+ [Awesome-CTF-Book](https://book.ph0en1x.com/web/phpdai-ma-shen-ji-xiao-zong-jie.html)
