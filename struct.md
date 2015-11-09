
## 1.进制转换

    from decimal import *
    def decimal_util(prec):
        # 十进制
        print getcontext()
        getcontext().prec=prec              # 设置精度为prec，如 prec=3
        print Decimal(1) / Decimal(7)       # 0.143
    
    print bin(10)       # decimal to binary
    print hex(10)       # decimal to hex
    print oct(10)       # decimal to octal
    
    print int('0b1010', 2)      # binary to decimal
    print int('0xa', 16)        # hex to decimal
    print int('012', 8)         # octal to decimal
    
    hex_num = 0xa
    binary_num = 0b1010
    print bin(hex_num)              # hex to binary
    print hex(binary_num)           # bin to hex
    

## 2.进制处理
binary 常用文件的存取和socket处理，在Python中常用`struct`来处理.常用`struct`的`Struct`类或直接使用该模块级函数。最重要的三个函数是`pack()`, `unpack()`, `calcsize()`

- `pack(fmt, v1, v2, ...)` :按照给定的格式(fmt)，把数据封装成字符串(实际上是类似于c结构体的字节流)
- `unpack(fmt, string)`:按照给定的格式(fmt)解析字节流string，返回解析出来的tuple
- `calcsize(fmt)`:计算给定的格式(fmt)占用多少字节的内存

同正则表达式，**格式指示符由字符串转换为一种编译表示，这种转换是消耗资源的**，所以`Struct`类实例调用相应的方法完成一次转换会更加高效。(注：测试发现模块级函数与Struct实例函数耗时相差无几).

所以有两种形式

	# 模块级函数
	import struct
	values = (1, 'ab', 2.4)
	s = struct.pack('I2sf', *values)

	# Struct类实例
	ss = struct.Struct('I2sf')
	bs = ss.pack(*values)

**格式指示符包含数据类型，可选数量以及字节序** 如下所示：


>注1.q和Q只在机器支持64位操作时有意思;注2.每个格式前可以有一个数字，表示个数
>注3.s格式表示一定长度的字符串，4s表示长度为4的字符串，但是p表示的是pascal字符串
注4.P用来转换一个指针，其长度和机器字长相关; 注5.最后一个可以用来表示指针类型的，占4个字节

![](http://7fvf56.com1.z0.glb.clouddn.com/struct_docs.png)


字节序

![](http://7fvf56.com1.z0.glb.clouddn.com/struct_docs2.png)

在[Python使用struct处理二进制](http://www.cnblogs.com/gala/archive/2011/09/22/2184801.html)中，关于二进制和普通文件读写的区别：


我们使用处理二进制文件时，需要用如下方法

	binfile=open(filepath,'rb')    读二进制文件

	binfile=open(filepath,'wb')    写二进制文件

那么和`binfile=open(filepath,'r')`的结果到底有何不同呢？

>不同之处有两个地方：

>第一，使用'r'的时候如果碰到'0x1A'，就会视为文件结束，这就是EOF。使用'rb'则不存在这个问题。即，如果你用二进制写入再用文本读出的话，如果其中存在'0X1A'，就只会读出文件的一部分。使用'rb'的时候会一直读到文件末尾。

>第二，对于字符串x='abc\ndef'，我们可用len(x)得到它的长度为7，\n我们称之为换行符，实际上是'0X0A'。当我们用'w'即文本方式写的时候，在windows平台上会自动将'0X0A'变成两个字符'0X0D'，'0X0A'，即文件长度实际上变成8.。当用'r'文本方式读取时，又自动的转换成原来的换行符。如果换成'wb'二进制方式来写的话，则会保持一个字符不变，读取时也是原样读取。所以如果用文本方式写入，用二进制方式读取的话，就要考虑这多出的一个字节了。'0X0D'又称回车符。linux下不会变。因为linux只使用'0X0A'来表示换行。


应用：[UserID <==> SessionID 加密互换](https://gist.github.com/BeginMan/7a0e734cfc5d1cf6392b)

(在家竟然能访问gist, 可惜在公司无法访问，被墙则自行翻墙)

