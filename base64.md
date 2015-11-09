

	import base64
	s='beginman'
	bs =  base64.b64encode(s)
	print bs, type(bs)              # YmVnaW5tYW4= &lt;type 'str'&gt;
	print base64.b64decode(bs)      # beginman

- 采用Base64编码具有不可读性，即所编码的数据不会被人用肉眼所直接看到。
- 生成的编码都是ascii字符
- 速度快，适合http传输
- 编码比较长，非常容易被破解，仅适用于加密非关键信息的场合


base64应用场景：


- mail
- URL,还需要url-encoding将base64特殊字符如+,=装换成%xx的模式
- HTML中内嵌图片，`img src='data:image/png;base64，base64code'`
- 简单的，非明文显示的加密

base64模块真正用的上的方法只有8个，分别是encode, decode, encodestring, decodestring, b64encode,b64decode, urlsafe_b64decode,urlsafe_b64encode。

- encode,decode一组，专门用来编码和解码文件的,也可以对StringIO里的数据做编解码；
- encodestring,decodestring一组，专门用来编码和解码字符串；
- b64encode和b64decode一组，用来编码和解码字符串，并且有一个替换符号字符的功能。这个功能是这样的：因为base64编码后的字符除 了英文字母和数字外还有三个字符 + / =, 其中=只是为了补全编码后的字符数为4的整数，而+和/在一些情况下需要被替换的，b64encode和b64decode正是提供了这样的功能。至于什么情况下+和/需要被替换，最常见的就是对url进行base64编码的时候。
- urlsafe_b64encode和urlsafe_b64decode 一组，这个就是用来专门对url进行base64编解码的，实际上也是调用的前一组函数。


如下实例：

	import base64
	import StringIO

	s = 'beginman'
	bs = base64.encodestring(s)
	print bs                # YmVnaW5tYW4=

	c = StringIO.StringIO()
	d = StringIO.StringIO()
	e = StringIO.StringIO()

	c.write(s)
	print c.getvalue()      # beginman

	c.seek(0)

	"""
	def encode(input, output):
	    #Encode a file.
	"""
	base64.encode(c, d)     # 对StringIO内的数据进行编码
	print d.getvalue()      # YmVnaW5tYW4

	d.seek(0)
	base64.decode(d, e)     # 对StringIO内的数据进行解码
	print e.getvalue()      # beginman

	bb = base64.urlsafe_b64encode('beginman+24/python=geek')
	print bb                # YmVnaW5tYW4rMjQvcHl0aG9uPWdlZWs=
	print base64.urlsafe_b64decode(bb)

对文件加密：

	import base64
	with open('a.txt') as f1:
	    with open('b.txt', 'w') as f2:
	        base64.encode(f1, f2)
