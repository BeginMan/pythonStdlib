
`binascii` [Convert between binary and ASCII](https://docs.python.org/2/library/binascii.html),
该模块比较简单，还是直接看代码吧。

- 十进制的转换：`hex`,`oct`
- 其他进制转十进制：`int(xx)`或`int('进制字符串形式', 进制)`
- ascii 码的字符表示： `chr(97)`
- 字符的ascii 码表示: `ord('a')`
- 将ascii 码转与进制互转: 需要`binascii`库，意思就是 `进制<->ascii`.例如`b2a_hex('a')`,  表示将ascii 码转为16进制，`b`:进制，`a`：ascii。

1.数字字符串表示

    print int('18')
    print int('0x12', 16)	# hex
    print hex(18) == '0x12'

2.ASCII char 互转


    print chr(97)
    # char to ASCII
    print ord('a')
    print ord('\xbc')

3.ascii与16进制互转


    import binascii
    print binascii.b2a_hex('Beginman')
    print binascii.b2a_hex('a')  # a对应ASCII码97,转换16进制则61

    print binascii.b2a_hex('方')    # e696b9
    print binascii.b2a_hex(u'方'.encode('utf-8'))  # e696b9, 不能直接使用unicode字符

    print binascii.a2b_hex('e696b9')

    # 该功能和binascii.hexlify(), binascii.unhexlify() 一样
    print binascii.hexlify('a'), binascii.unhexlify('61')


4.ascii与base64编码互转

    print binascii.b2a_base64('a')		# 带有换行符 \n
    import base64
    print base64.b64encode('a')			# 不带有换行符

    print base64.b64encode('a') == (binascii.b2a_base64('a')).replace('\n','')		# True


## 应用

关于`a2b_hex`，可以用[byte_string_to_hex](https://github.com/BeginMan/pytool/blob/master/datastruct/byte-to-hex.py#L11)来写原型；`b2a_hex`可以用[byte_string_from_hex](https://github.com/BeginMan/pytool/blob/master/datastruct/byte-to-hex.py#L22)来替代

**不过还是建议用标准库的好**

参考：

- [python 进制转换小结](http://blog.csdn.net/jiaxiaolei19871112/article/details/6772729)
- [py 图片>>base64>>二进制文本 转换](http://linux521.blog.51cto.com/4099846/1101775)

代码:

- [图片>>base64>>二进制文本 转换](https://gist.github.com/33e7e8f3a7707c79d80f)


