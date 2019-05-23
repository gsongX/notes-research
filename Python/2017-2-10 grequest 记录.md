---
title: 2017-2-10 Grequest 记录
tags: greques
grammar_cjkRuby: true
---
# Grequest 记录
GRequests: Asynchronous Requests

GRequests allows you to use Requests with Gevent to make asynchronous HTTP Requests easily.
异步请求 GRequests允许您使用具有Gevent的Requests轻松地进行异步HTTP请求。


## 安装  
> pip install grequests

## 优化

```python
# 这种写法会造成程序崩溃
rs = (grequests.get(url) for url in url_chunk)
res_items = grequests.map(rs, timeout=10s) # this is the item that times out

# 这样写可以优化性能
rs = (grequests.get(u, timeout=3, proxies=proxies) for u in urls)
for r in grequests.imap(rs, size=200):
    print(r.content)
```
## 事例

```python 
# 爬取url 中的 url

import re
import requests
import grequests

proxies = {
  "http": "http://127.0.0.1:1080",
}

def get_url(html):
    m = re.findall(r'(https|http?:\/\/[^\s]+.com|cn|org)', html)
    return (m)

html= requests.get("http://www.best918.com/").text
urls = get_url(html)
gen = (grequests.get(i, proxies=proxies, timeout=3)  for i in urls)
for i in grequests.imap(gen, size=200):
    print(i.url)
    for k in get_url(i.text):
        urls.append(k)
        


```


