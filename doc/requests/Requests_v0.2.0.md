# 文章转自 [这位大佬](https://github.com/wangshunping/read_requests) 再加上我的思考

### 0X01

这篇文章拆的是v 0.2.0版本。

根据 HISTORY.rst ，requests 是从 0.0.1 版本开始的，但是 github 的版本库里面找不到 v0.0.1版本的了。所以从 v0.2.0 开始。

下面是版本信息。

```
History
-------

0.2.0 (2011-02-14)
++++++++++++++++++

* Birth!


0.0.1 (2011-02-13)
++++++++++++++++++

* Frustration
* Conception

```

2011-02-14， requests v0.2.0正式发布，看日期，情人节呐，这个轮子肯定是送给自己情人节礼物：）

话说我今年送给自己什么情人节礼物来着.... （脑补思索）

可以看到0.01 版本中用到了 frustration ，结果第二天就发了新版本：）

### 0X03

下面是 README.rst 信息

```

Requests: The Simple (e.g. usable) HTTP Module
==============================================

Most existing Python modules for dealing HTTP requests are insane. I have to look up *everything* that I want to do. Most of my worst Python experiences are a result of the various built-in HTTP libraries (yes, even worse than Logging). 

But this one's different. This one's going to be awesome. And simple.

Really simple. 

```

这是readme的第一部分信息。我用我专业的英文六级水平翻译一下。

Requests: 简单好用的HTTP 模块

大部分存在的处理HTTP请求的模块太傻逼了，我找了整个互联网，然后我准备自己干这个了。 由于我 python 的经验不够，这个库的大部分都是基于python 标准库HTTP 模块的（是的，比Logging 还搓）。

但是！这个库跟其他那些傻逼是不一样的！这个库会变的碉堡了！嗯，还有简单。

真的超级简单 ：）


哈哈哈，看了这个README 我就笑了，我喜欢狂妄的年轻人。

```
Features
--------

- Extremely simple GET, HEAD, POST, PUT, DELETE Requests
    + Simple HTTP Header Request Attachment
    + Simple Data/Params Request Attachment
- Simple Basic HTTP Authentication
    + Simple URL + HTTP Auth Registry

```

能GET, HEAD, POST, PUT, DELETE， 能添加请求头，能添加参数和数据，能进行简单的HTTP 验证。

嗯，都是基本功能。

```

Usage
-----

It couldn't be simpler. ::

    >>> import requests
    >>> r = requests.get('http://google.com')


HTTPS? Basic Authentication? ::
    
    >>> r = requests.get('https://convore.com/api/account/verify.json')
    >>> r.status_code
    401

    
Uh oh, we're not authorized! Let's add authentication. ::
    
    >>> conv_auth = requests.AuthObject('requeststest', 'requeststest')
    >>> r = requests.get('https://convore.com/api/account/verify.json', auth=conv_auth)
    
    >>> r.status_code
    200 
    
    >>> r.headers['content-type']
    'application/json'
    
    >>> r.content
    '{"username": "requeststest", "url": "/users/requeststest/", "id": "9408", "img": "censored-long-url"}'

```

嗯。跟现在的用法都是一样简洁 ：）

API 部分，安装部分，就直接跳过吧。
### 0X04

requests 代码之前，先从测试代码开始吧。

```python
class RequestsTestSuite(unittest.TestCase):
	"""Requests test cases."""
	
	def setUp(self):
		pass

	def tearDown(self):
		"""Teardown."""
		pass
		
	def test_invalid_url(self):
		self.assertRaises(ValueError, requests.get, 'hiwpefhipowhefopw')

	def test_HTTP_200_OK_GET(self):
		r = requests.get('http://google.com')
		self.assertEqual(r.status_code, 200)

	def test_HTTPS_200_OK_GET(self):
		r = requests.get('https://google.com')
		self.assertEqual(r.status_code, 200)

	def test_HTTP_200_OK_HEAD(self):
		r = requests.head('http://google.com')
		self.assertEqual(r.status_code, 200)

	def test_HTTPS_200_OK_HEAD(self):
		r = requests.head('https://google.com')
		self.assertEqual(r.status_code, 200)

	def test_AUTH_HTTPS_200_OK_GET(self):
		auth = requests.AuthObject('requeststest', 'requeststest')
		url = 'https://convore.com/api/account/verify.json'
		r = requests.get(url, auth=auth)

		self.assertEqual(r.status_code, 200)

```
凭我多年(半年)TDD的经验，这份测试肯定是代码写完之后补的。并且写完代码之后很开心，写测试的时候满脑子想着快点丢到github 上面去哈哈哈哈，只测了get 和 head ，当然也可能他找不到 url去测post :)

self.assertRaises 这个用法我没用过，去翻了一个标准库文档，文档里面有这样一个例子，还是很好玩的：

```python
def test_split(self):
    # check that s.split fails when the separator is not a string
    # 检查 当 分隔符不是一个字符的时候 s.split 的失败
    with self.assertRaises(TypeError):
        s.split(2)

```
函数定义：
assertRaises(exc, fun, *args, **kwds)
简单介绍一下 split 
> This is the opposite of concatenation which merges or combines strings into one. 这跟连接合并字符串完全相反
```python
x = ‘blue,red,green’
x.split(“,”)
 
[‘blue’, ‘red’, ‘green’]
>>>
 
>>> a,b,c = x.split(“,”)
 
>>> a
‘blue’
 
>>> b 
‘red’
 
>>> c
‘green’
```

### 0x05

终于开始撸代码了，requests 包只包含一个__init__.py 和 core.py, core.py 405行。

当使用 requests.get('www.baidu.com')时，直接到requests 包定义的相应函数，以 get 举例。

```python
def get(url, params={}, headers={}, auth=None):
	"""Sends a GET request. Returns :class:`Response` object.

	:param url: URL for the new :class:`Request` object.
	:param params: (optional) Dictionary of GET Parameters to send with the :class:`Request`.
	:param headers: (optional) Dictionary of HTTP Headers to sent with the :class:`Request`.
	:param auth: (optional) AuthObject to enable Basic HTTP Auth.
	"""
	
	r = Request()
	
	r.method = 'GET'
	r.url = url
	r.params = params
	r.headers = headers
	r.auth = _detect_auth(url, auth)
	
	r.send()
	
	return r.response

```
先会实例化一个 Request, Request 类定义在47行。

