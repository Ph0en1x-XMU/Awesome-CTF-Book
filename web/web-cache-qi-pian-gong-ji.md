# 原理
## 缓存实现方式
实现缓存有多种方式，其中有种方式是，将一台服务器部署在客户端和web服务器之间作为缓存服务器，其表现形式如下：
+ CDN（Content Delivery Network）
+ 负载均衡（Load balance）
+ 反向代理（Reverse proxy）

## 缓存工作流程

当访问不存在的URL时，如`http://www.example.com/home.php/non-existent.css`,浏览器发送get请求，依赖于使用的技术与配置，服务器返回了页面`http://www.example.com/home.php `的内容，同时URL地址仍然是`http://www.example.com/home.php/non-existent.css`，http头的内容也与直接访问`http://www.example.com/home.php `相同，cacheing header、content-type（此处为text/html）也相同。

具体环节如下：
1. 浏览器请求`http://www.example.com/home.php/no-existent.css `；
2. 服务器返回`http://www.example.com/home.php` 的内容(通常来说不会缓存该页面)；
3. 响应经过代理服务器；
4. 代理识别该文件有css后缀；
5. 在缓存目录下，代理服务器创建目录home.php，将返回的内容作为non-existent.css保存。

# 攻击流程
## 手动测试
1. 发送 `http://www.vuln.com/admin.php/no-existent.css`
2. 管理员（通常）查看，导致admin.php内容被缓存。
3. hacker访问 `http://www.vuln.com/admin.php/no-existent.css`，获取敏感信息。

## 工具
+ [Airachnid Burp Extension](https://github.com/SpiderLabs/Airachnid-Burp-Extension)

# 案例
+ [CTFzone17-TimeHackers](https://ctftime.org/writeup/6973)

# 参考
+ [Web Cache Deception Attack](http://omergil.blogspot.jp/2017/02/web-cache-deception-attack.html)
+ [浅析 Web Cache 欺骗攻击](http://bobao.360.cn/learning/detail/3828.html)
+ [On Web Cache Deception Attacks](https://blogs.akamai.com/2017/03/on-web-cache-deception-attacks.html)

