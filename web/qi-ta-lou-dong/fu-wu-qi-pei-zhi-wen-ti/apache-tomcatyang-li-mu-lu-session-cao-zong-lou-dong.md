# Apache Tomcat样例目录session操纵漏洞

## 0x01 介绍

Apache Tomcat默认安装包含”/examples”目录，里面存着众多的样例，其中session样例\(/examples/servlets /servlet/SessionExample\)允许用户对session进行操纵。

因为session是全局通用的，所以用户可以通过操纵 session获取管理员权限。

## 0x02 测试流程

访问：

```text
http://127.0.0.1:8080/examples/servlets/servlet/SessionExample
```

在Name of Session Attribute: 里输入login

在Value of Session Attribute:里输入admin

提交后显示login=admin已经写入session

再次打开index.jsp，显示成功登录

## 0x03 案例

* XDCTF-2015 web2 Apache Tomcat session操纵漏洞

