gzip 用于支持gzip文件。

基于Python2.x 的模块文档[gzip](https://docs.python.org/2/library/gzip.html).

>This module provides a simple interface to compress and decompress files just like the GNU programs gzip and gunzip would.

看[Lib/gzip.py](https://hg.python.org/cpython/file/2.7/Lib/gzip.py)源码可知晓，其方法很少，常用就是`open()`,也就是`GzipFile`类的快速操作。主要是`GzipFile`类。有`write()`,`read()`,`close()`,`flush()`,`seek()`,`readline`如下结构图：

![](https://raw.githubusercontent.com/BeginMan/pythonStdlib/master/media/gzip.png)

`compresslevel`这个可选的关键字参数来指定一个压缩级别,**默认的等级是9，也是最高的压缩等级。等级越低性能越好，但是数据压缩程度也越低。**

	# method 1. gzip.open
	with gzip.open('file.gz', 'wb') as f:
	    f.write('hello, world')

	# method 2.GzipFile
	zfile = gzip.GzipFile(mode='wb', compresslevel=9, filename='ts.gz')
	zfile.write('hello, world')
	zfile.close()

	# with StringIO
	txt = 'hello, world'
	buf = StringIO.StringIO()
	zfile = gzip.GzipFile(mode='wb', compresslevel=9, fileobj=buf)
	zfile.write(txt)
	zfile.close()

实例，数据base64并压缩上传往往会节省带宽和内存开销，服务端接收到数据进行还原工作：

	import base64
	import gzip
	import json
	from cStringIO import StringIO

	contacts_list = base64.b64decode(contacts_list)
	    try:
	        buf = StringIO(contacts_list)
	        f = gzip.GzipFile(mode='rb', fileobj=buf)
	        contacts_list = f.read()
	        contacts_list = json.loads(contacts_list) if not \
	            isinstance(contacts_list, list) else contacts_list
	    except:
	        f.close()

