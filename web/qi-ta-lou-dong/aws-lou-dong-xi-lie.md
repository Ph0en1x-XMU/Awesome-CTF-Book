# 漏洞一：S3 Bucket信息泄露
## 介绍
亚马逊简单存储服务（S3）是一种AWS服务，主要用以向用户提供一种安全的数据存储方式。

## 渗透流程
1. 利用nslookup确定是否是s3服务
![](https://github.com/CHYbeta/chybeta.github.io/blob/master/images/pic/20170731/1.jpg?raw=true)

2. Bucket名称枚举。使用子域，域和顶级域的组合来确定目标S3上是否有bucket，其格式为
```
bucketname.s3.amazonaws.com
```
假设目前已经确认www.example.com使用了s3服务：

    1. 用浏览器访问：http://www.example.me.s3.amazonaws.com 若成功则返回xml样式。
![](https://github.com/CHYbeta/chybeta.github.io/blob/master/images/pic/20170731/2.jpg?raw=true)
    2. 利用awscli工具探测。
```
aws s3 ls s3://$bucketname/ --region $region
```

3.利用。当确定Bucket后，就可以直接利用awscli进行各种命令。
+ 列举目录
```
aws s3 ls s3://$bucketname

```

+ 上传
```
aws s3 mv localfile s3://$bucketname
```
    

## 参考
+ [针对亚马逊云存储器S3 BUCKET的渗透测试 ](http://www.freebuf.com/articles/web/135313.html)

# 漏洞实践
+ [flAWS challenge](http://flaws.cloud/)
+ [Bugs_Bunny CTF 2017 – Walk walk](https://florentbesnard.com/2017/bugs_bunny-ctf-walk-walk/)
# 参考
+ [AWS常见安全问题汇总](http://www.freebuf.com/articles/system/129667.html)