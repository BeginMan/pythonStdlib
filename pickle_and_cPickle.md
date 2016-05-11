# pickle和cPickle
Python专用的数据格式库，其他语言不能使用，如果选择通用方案，JSON是最佳选择。但是为什么存在picket这个玩意呢？总结如下：

- Python专用，不同版本不向后兼容，不能被其他语言识别
- 可以直接将Python对象保存到文件，而不需要先转换成字符串，也不用OS操作写入一个二进制文件中。pickle模块会创建一个python语言专用的二进制格式，你基本上不用考虑任何文件细节，它会帮你干净利落地完成读写独享操作，唯一需要的只是一个合法的文件句柄。
- cPickle是pickle得一个更快得C语言编译版本。

函数：

- dump(), dumps()
- load(), loads()

其中写入文件：

 	cPickle.dump(obj, file, protocol=0)

序列化对象，并将结果数据流写入到文件对象中。参数protocol是序列化模式，默认值为0，表示以文本的形式序列化。protocol的值还可以是1或2，表示以二进制的形式序列化。
一一对应。如下测试：

	In [19]: a = 100
	In [20]: pc = cPickle.dumps(a)
	In [23]: pc
	Out[23]: 'I100\n.'
	In [21]: b = cPickle.loads(pc) * 10000
	In [22]: b
	Out[22]: 1000000


