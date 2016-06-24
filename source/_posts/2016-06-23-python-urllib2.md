---
title: python网络请求库urllib
categories: python
tags: urllib
date: 2016-06-23 15:21:59
updated: 2016-06-23 15:21:59
---

Python也提供了自带网络请求库有，urllib，urllib2
* urllib          # 初级url请求库，提供最基本的url请求，header都不支持
* urllib2         # 高级url请求库，通常与urllib库一起使用
* cookielib       # Cookie容器处理

<!-- more -->

## 1. Opener
在Python中使用Opener对象来请求的url资源，使用`urlopen`方法则调用默认的opener请求
```python
import urllib2
# 使用默认的opener请求数据（Get）
response = urllib2.urlopen('http://python.org/')
html = response.read()

# 可以把url封装成Request对象进行请求（Get）
req = urllib2.Request('http://www.python.org')
response = urllib2.urlopen(req)
html = response.read()
```

### 2. Post请求
上面处理的是get请求，下面看看Post请求，只要在Request对象设置了data参数或在urlopen设置了data参数，就会被识别为post请求
```python
req = urllib2.Request('http://www.python.org')
values = {'name': 'bomo', 'age': 18}
data = urllib.urlencode(values)
req = urllib2.Request(url, data)
response = urllib2.urlopen(req)
html = response.read()
```
## 2. Header
```python
req = urllib2.Request('http://www.python.org')
values = {'name': 'bomo', 'age': 18}
data = urllib.urlencode(values)
headers = {'User-Agent': "Mozilla/4.0 (compatible; MSIE 5.5; Windows NT)"}
response = urllib2.urlopen(req, headers, headers=headers data=data)
html = response.read()
```
## 3. Cookie
Web请求相当于一条请求管道，在请求的过程中可以有多个操作，如缓存处理，Cookie处理，URL跳转处理，代理处理等等，在urllib2中被定义为handler，cookie处理相当于opener的一个handler，一个opener可以有多个handler，通过不同的handler处理做不同的处理，如可以通过自带的方法创建handler，如有urllib2.ProxyHandler，urllib2.HTTPRedirectHandler等


Python中Cookie的基本使用如下
```python
import urllib2
import cookielib

filename = 'cookie.txt'
# 构造一个Cookie容器来保存Cookie
cookie = cookielib.LWPCookieJar(filename)
# 利用urllib2库的HTTPCookieProcessor对象来创建cookie处理器
handler = urllib2.HTTPCookieProcessor(cookie)
# 通过handler来构建opener
opener = urllib2.build_opener(handler)
# 有了opener就可以请求了
response = opener.open("http://www.baidu.com")
# 请求完成后，可以获取到cookie的值，这里我们保存到文件
cookie.save(ignore_discard=True, ignore_expires=True)
```

刚刚获取到cookie并存到了文件，这时候我们直接从文件读取出cookie来使用
```python
import urllib2
import cookielib

filename = 'cookie.txt'
cookie = cookielib.LWPCookieJar(filename)
cookie.load('cookie.txt', ignore_discard=True, ignore_expires=True)
opener = urllib2.build_opener(urllib2.HTTPCookieProcessor(cookie))
req = urllib2.Request("http://www.baidu.com")
response = opener.open(req)
print response.read()
```

## 4. 重定向问题
默认情况下，Python请求到301/302的结果的时候，会自动进行重定向请求，如果不需要跳转怎么办，我们可以添加一个handler，不处理重定向的操作
```python
import urllib2

class RedirectHandler(urllib2.HTTPRedirectHandler):
    def http_error_301(self, req, fp, code, msg, headers):
        pass
    def http_error_302(self, req, fp, code, msg, headers):
        pass

opener = urllib2.build_opener(RedirectHandler)
opener.open('http://www.google.cn')
```

## 5. 参考
* [urllib2库官方介绍](https://docs.python.org/2/library/index.html)
* [BeautifulSoup官方介绍](https://www.crummy.com/software/BeautifulSoup/bs4/doc.zh/)
