

uuid也叫GUID,(UUID —— Universally Unique IDentifier；GUID —— Globally Unique IDentifier) 全局唯一标识符，128位个长二进制整数

	>>>uuid.uuid1()
	UUID('7df790cc-6a4e-11e4-9707-9cf387a6e308')
	>>>uuid.uuid4()
	UUID('87a31a3c-4819-462d-9a35-91d944b0bdad')
	>>uuid.uuid3(uuid.NAMESPACE_DNS,'222')
	UUID('c6b6add2-965d-3895-8397-14908b2d7a5e')

	没有uuid2()
