glob是Python标准库之一，主要作用就是依据Unix Shell规则找出对应位置与之匹配的文件名。主要方法就是`glob.glob(pattern)`

这里的规则`pattern`与正则不同，仅仅有以下几个：

(1) `*` 匹配0个或者一个

	glob.glob('/home/beginman/learn/*.sh')  
	# 匹配该目录下shell脚本

对于多级目录`glob`并不会递归搜索，可以用`*`进行手动递归

	glob.glob('/home/beginman/*/*/*.py')


 (2) `?`匹配单个字符

 	glob.glob('/home/beginman/learn/test?.py')
	# 输出 ：test1.py test2.py  (省略..)



(3) `[]`区间，如`[0-9]`,`[a-z]`等

	#!/usr/bin/env python
	# coding=utf-8
	import itertools as it, glob, os

	def chainFiles(*patterns):
	    return it.chain.from_iterable(glob.glob(pattern) for pattern in patterns)

	for file in chainFiles('*.sh', '*.txt'):
	    print '%-15s' %file, os.path.realpath(file)

	# 输出：
	tty.sh          /home/beginman/learn/tty.sh
	1.sh            /home/beginman/learn/1.sh
	newTest.txt     /home/beginman/learn/newTest.txt
	test.txt        /home/beginman/learn/test.txt
	newNumber.txt   /home/beginman/learn/newNumber.txt
	1.txt           /home/beginman/learn/1.txt
