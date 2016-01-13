你想使用操作类文件对象的程序来操作文本或二进制字符串，那就使用`StringIO`,**此类中的大部分函数都与对文件的操作方法类似。**

	s=StringIO.StrngIO([buf])
	# 可选参数buf是一个str或unicode类型用来初始化StringIO,它将会与其他后续写入的数据存放在一起。

与文件不同：

- 不会再磁盘操作，而是只寄存在缓冲区(buf)
- 使用`read()`时必须通过`seek()`先设置"文件指针"的位置才能读取到内容
- `getvalue()`则获取其中数据
- 在Python3.x 则`from io import StringIO`
- 在Python3.x StringIO操作的只能是str，如果要操作二进制数据，就需要使用`BytesIO`。

如下：
	
	import StringIO
	s = StringIO.StringIO("hello")
	print s.getvalue() + '\n---------'
	s.write('world\n')
	s.write('中国')
	print 'read:', s.read()
	s.seek(0)
	print s.read() + '\n----------'
	print s.getvalue()

	# outputs:
	hello
	---------
	read: 
	world
	中国
	----------
	world
	中国

如果我们进入StringIO模块内，发现与文件对象十分类似。如`close()`释放缓冲区，`flush()`刷新内部缓冲区。

Python标准模块中还提供了一个`cStringIO`模块，它的行为与`StringIO`基本一致，但运行效率方面比`StringIO`更好。但使用 cStringIO模块时，有几个注意点： 

1. cStringIO.StringIO不能作为基类被继承；
2. 创建cStringIO.StringIO对象时，如果初始化函数提供了初始化数据，新生成的对象是只读的。所以下面的代码是错误的：s = cStringIO.StringIO("JGood/n"); s.write("OOOKKK")


关于其应用可参考：[flask-stringIOs.py](https://github.com/BeginMan/pytool/blob/master/IO/flask-stringIOs.py)


