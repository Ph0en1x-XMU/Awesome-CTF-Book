# 读取文件函数
（注：此部分来源[php花式读取文件函数汇总](http://www.jianshu.com/p/33bc37ef72cc)）
## 读文件
```php
hightlight_file($filename); 
show_source($filename); 
print_r(php_strip_whitespace($filename));
print_r(file_get_contents($filename));
readfile($filename); 
print_r(file($filename)); // var_dump 
fread(fopen($filename,"r"), $size); 
include($filename); // 非php代码 
include_once($filename); // 非php代码 
require($filename); // 非php代码 
require_once($filename); // 非php代码 
print_r(fread(popen("cat flag", "r"), $size)); 
print_r(fgets(fopen($filename, "r"))); // 读取一行 
fpassthru(fopen($filename, "r")); // 从当前位置一直读取到 EOF 
print_r(fgetcsv(fopen($filename,"r"), $size)); 
print_r(fgetss(fopen($filename, "r"))); // 从文件指针中读取一行并过滤掉 HTML 标记 
print_r(fscanf(fopen("flag", "r"),"%s")); 
print_r(parse_ini_file($filename)); // 失败时返回 false , 成功返回配置数组
```

## 列目录
```php
print_r(glob("*")); // 列当前目录 
print_r(glob("/*")); // 列根目录 print_r(scandir(".")); 
print_r(scandir("/")); 
$d=opendir(".");while(false!==($f=readdir($d))){echo"$f\n";}
$d=dir(".");while(false!==($f=$d->read())){echo$f."\n";}
```

## 超全局变量






