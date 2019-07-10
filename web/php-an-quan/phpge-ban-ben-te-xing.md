# PHP 各版本特性

## php5.2以前

1. \_\_autoload加载类文件，但只能调用一次这个函数，所以可以用spl\_autoload\_register加载类

## php5.3

1. 新增了glob://和phar://流包装
   * glob用来列目录，绕过open\_baedir [http://php.net/manual/zh/wrappers.phar.php](http://php.net/manual/zh/wrappers.phar.php)
   * phar在文件包含中可以用来绕过一些后缀的限制 [http://php.net/manual/zh/wrappers.phar.php](http://php.net/manual/zh/wrappers.phar.php)
2. 新的全局变量**DIR**
3. 默认开启`<?= $xxoo;?>`，5.4也可用

## php5.4

1. 移除安全模式、魔术引号
2. register\_globals 和 register\_long\_arrays php.ini 指令被移除。
3. php.ini新增session.upload\_progress.enabled，默认为1，可用来文件包含

   [http://php.net/manual/zh/session.configuration.php](http://php.net/manual/zh/session.configuration.php)

   [http://php.net/manual/zh/session.upload-progress.php](http://php.net/manual/zh/session.upload-progress.php)

## php5.5

1. 废除preg\_replace的/e模式\(不是移除\)

   当使用被弃用的 e 修饰符时, 这个函数会转义一些字符\(即：'、"、  和 NULL\) 然后进行后向引用替换。

   [http://php.net/manual/zh/function.preg-replace.php](http://php.net/manual/zh/function.preg-replace.php)

## php5.6

1. 使用 ... 运算符定义变长参数函数

   [http://php.net/manual/zh/functions.arguments.php\#functions.variable-arg-list](http://php.net/manual/zh/functions.arguments.php#functions.variable-arg-list)

## php7.0

1. 十六进制字符串不再是认为是数字
2. 移除asp和script php标签

   ```text
   <% %>
   <%= %>
   <script language="php"></script>
   ```

3. 在后面的版本中assert变成语言结构，这将意味着很多一句话不能使用。

   目前经过测试,可使用的有。

   ```text
   call_user_func('assert', 'phpinfo();');
   ```

## php7.1

[http://php.net/manual/zh/migration71.new-features.php](http://php.net/manual/zh/migration71.new-features.php)

1. 废除mb\_ereg\_replace\(\)和mb\_eregi\_replace\(\)的Eval选项

## Refference

* [官方文档](http://php.net/manual/zh/appendices.php)
* [l3m0n:php各版本的姿势](http://www.cnblogs.com/iamstudy/articles/study_from_php_update_log.html)

