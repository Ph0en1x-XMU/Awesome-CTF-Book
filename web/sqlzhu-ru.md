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

## 路径相关：
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

## 字母/数字相关：
+ ASCII(): 获取字母的ascii码值
+ BIN(): 返回值的二进制串表示
+ CONV(): 进制转换
+ LOWER()：转成小写字母
+ UPPER(): 转成大写字母
+ HEX():十六进制编码
+ UNHEX()：十六进制解码


## 字符串截取
+ MID()
+ LEFT()
+ SUBSTR()
+ SUBSTRING()