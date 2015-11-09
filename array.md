array模块，用于提供基本数字,字符类型的数组,主要用于二进制上的缓冲区,流的操作

![](http://beginman.qiniudn.com/pyarray.jpg)

	import array
	myarr = array.array('l')        # 创建一个interger类型的数组
	for i in range(10):
	    myarr.append(i)

	print myarr                     # array('l', [0, 1, 2, 3, 4, 5, 6, 7, 8, 9])
	print type(myarr)               # &lt;type 'array.array'&gt;

	#删除第一个指定的X
	myarr.remove(3)
	#删除最后一个
	myarr.pop()
	print myarr[1],myarr[3]         # 1,4

	#插入元素
	myarr.insert(1, 100)

	print myarr                         # array('l', [0, 100, 1, 2, 4, 5, 6, 7, 8])

	#获取元素对应的索引
	print myarr.index(100)            # 1

	myarr.reverse()                     # 反序
	print myarr                         # array('l', [8, 7, 6, 5, 4, 2, 1, 100, 0])

对于不可变的字符串，如果修改则会发生异常
	
	st = 'beginman'
	st[1] = 'B'         # TypeError: 'str' object does not support item     assignment

可以使用array模块进行修改
	
	import array
	st = 'beginman'
	ar = array.array('c', st)
	ar[1] = 'B'
	print ar        # array('c', 'bBginman')
	print ar.tostring() # bBginman
	print ar.tolist()   # ['b', 'B', 'g', 'i', 'n', 'm', 'a', 'n']
	