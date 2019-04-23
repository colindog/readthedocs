请求
==============
访问保全网的API需要使用https协议，并进行数字签名。


路径
--------------

八仙过海会提供向后兼容的API，不同的API版本会以比如v1、v1.1这样的字符串标记，当前的版本为v1。

API地址：`https://www.baxianguohai.com/api/v1`

请求路径=API地址+接口名称，比如添加接口的名称是/add，则请求路径为
`https://www.baxianguohai.com/api/v1/add`


方法
--------------

所有的请求均为HTTP POST方法

参数
--------------

.. _Unix Time: https://en.wikipedia.org/wiki/Unix_time>

=================  =====================================================================================
参数名 				描述 
=================  =====================================================================================
request_id         请求ID，由接入方创建唯一字符串，长度不超过32位
access_key         访问识别码，当您在八仙过海网成功上传RSA公钥后会获得一个access_key
time_uni           请求时间, 必须以 `Unix Time`_ 的格式发送, time_uni与服务器时间不得超过正负400秒，否则请求将视为无效
payload            被签名的数据体部分，用JSON进行编码，不同的请求的payload数据一般不同
signature          使用你的RSA私钥进行签名后的字符串，具体签名的方法后面会进一步描述
=================  ===================================================================================== 

例如::

	{
		"request_id": "KmWor6FKfKgzcDnncfajdM",
		"access_key": "VMVSHrR7Q3U5DsLOhKVE0s",
		"tonce": 1555296075,
		"payload": {
			"product_id": "iaZJAhLaexqHcsh2tOG5pn",
		}
		"signature": "GSUW39JzUp3drCg7pfaTWFWcMXonPlZoaOVyM"
	}
