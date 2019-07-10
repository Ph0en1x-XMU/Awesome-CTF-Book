# zip总结

## 0x00 伪加密

Zip伪加密的出现是因为zip二进制文件中有一位是标记是否加密的。若zip包实际上没有被加密但加密位却是01，那么在打开时就会提示有密码。（Mac OS以及部分Linux可以直接打开）

我们只要用16进制编辑器修改加密标志位为00即可破解。有关Zip文件头协议可以参考附录。

我们还可以利用工具ZipCenOp.jar工具修改，道理是一样的。

## 0x01 爆破密码

爆破主要是利用工具，结合密码所处域的所有可能性字符串进行爆破。有时我们知道密码的一部分，这时就可以缩小密码域的范围，缩小密码的域可以提升爆破速度，这时可以手工生成字典。

Windows下常用的爆破工具有：

1. Ziperello
2. AZPR

生成字典可以手动写脚本，也可以用一些工具，比如kali下的crunch。

## 0x02 明文攻击

明文攻击的条件是你已经知道压缩包中加密文件的一部分文件\(大于12bit\)，这时就可以进行明文攻击。详细原理不赘述。

在攻击前，拿到了已知文件后可以先用压缩工具对已知文件进行压缩看看CRC32是否和未知加密文件的CRC32一致，若不一致可以考虑换个压缩工具。若一致可进行明文攻击。

攻击也可用AZPR工具进行。

其他工具：\[UZPC\]\([http://www.chat.ru/~m53group](http://www.chat.ru/~m53group)\) \[PKCrack\]\([http://www.unix-ag.uni-kl.de/~conrad/krypto/pkcrack.html](http://www.unix-ag.uni-kl.de/~conrad/krypto/pkcrack.html)\)

**注意：当明文的大小比较小时，攻击速度会比较慢；即使有时没有恢复密码，也可以使用明文攻击，最后点保存还是能得到压缩包里内容的\(这点很坑。。\)。**

## 0x03 CRC32碰撞

当压缩包密码实在解不出来，但是压缩包内的内容比较短的时候可以用CRC32碰撞。

这里记录一个脚本：python2.7

```python
#!/usr/bin/env python

import binascii

dic = r"0123456789abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ!#$%&()*+,-./:;<=>?@[\]^_`{|}~ "

for x1 in dic:
    for x2 in dic:
        for x3 in dic:
            for x4 in dic:
                for x5 in dic:
                    s = x1+x2+x3+x4+x5
                    crc = binascii.crc32(s)&0xFFFFFFFF
                    if (crc==0x20AE9F17):
                        print hex(crc),s
```

## 0x04 压缩包损坏

此类需要了解[Zip文件协议](http://blog.csdn.net/ETF6996/article/details/51946250),结合 Binwalk 扫描损坏的文件包，然后利用16进制工具修复。

附录：Zip文件协议

```bash
压缩源文件数据区 
50 4B 03 04：这是头文件标记（0x04034b50）
14 00：解压文件所需 pkware 版本 
00 00：全局方式位标记（有无加密） 
08 00：压缩方式 
5A 7E：最后修改文件时间 
F7 46：最后修改文件日期 
16 B5 80 14：CRC-32校验（1480B516）
19 00 00 00：压缩后尺寸（25）
17 00 00 00：未压缩尺寸（23）
07 00：文件名长度 
00 00：扩展记录长度 
6B65792E7478740BCECC750E71ABCE48CDC9C95728CECC2DC849AD284DAD0500 
压缩源文件目录区 
50 4B 01 02：目录中文件文件头标记(0x02014b50) 
3F 00：压缩使用的 pkware 版本 
14 00：解压文件所需 pkware 版本 
00 00：全局方式位标记（有无加密，这个更改这里进行伪加密，改为09 00打开就会提示有密码了） 
08 00：压缩方式 
5A 7E：最后修改文件时间 
F7 46：最后修改文件日期 
16 B5 80 14：CRC-32校验（1480B516）
19 00 00 00：压缩后尺寸（25）
17 00 00 00：未压缩尺寸（23）
07 00：文件名长度 
24 00：扩展字段长度 
00 00：文件注释长度 
00 00：磁盘开始号 
00 00：内部文件属性 
20 00 00 00：外部文件属性 
00 00 00 00：局部头部偏移量 
6B65792E7478740A00200000000000010018006558F04A1CC5D001BDEBDD3B1CC5D001BDEBDD3B1CC5D001 
压缩源文件目录结束标志 
50 4B 05 06：目录结束标记 
00 00：当前磁盘编号 
00 00：目录区开始磁盘编号 
01 00：本磁盘上纪录总数 
01 00：目录区中纪录总数 
59 00 00 00：目录区尺寸大小 
3E 00 00 00：目录区对第一张磁盘的偏移量 
00 00：ZIP 文件注释长度
```

参考：

[https://bobao.360.cn/ctf/learning/203.html](https://bobao.360.cn/ctf/learning/203.html)

[http://www.blogsir.com.cn/safe/252.html](http://www.blogsir.com.cn/safe/252.html)

