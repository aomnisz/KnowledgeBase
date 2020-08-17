# LOCALHOST write-up 及一些有用的 HTTP 头部

进入题目链接 http://web.jarvisoj.com:32774/ ，看到提示：

> localhost access only!!

马上反映过来，应该是改 HTTP 的头了。之前曾经搞过反向代理，知道目标服务器判断请求的源 IP 是通过 HTTP 的头部的一个参数来的。

开浏览器代理，把请求导向 burp ，然后发去 Repeater 。在 HTTP 的头部的位置加上一行 `X-Forward-For: 127.0.0.1` ，点击 Send ！咦，怎么没反应。。。

检查了半天，~~还看了别人的write-up，~~才发现原来是 Forward**ed** 。改一下，再发，就可以了。

```
X-Forwarded-For: 127.0.0.1
```

顺便，HTTP 头部是不分大小写的，所以看到别人是写着 `X-FORWARDED-FOR` 。

最后总结一下一些有用的 HTTP 头部吧，省得以后再犯这种低级错误。

| 字段名           | 说明                                                                                     |
| :--------------- | :--------------------------------------------------------------------------------------- |
| Accept           | 能够接受的回应内容类型（Content-Types）                                                  |
| Accept-Charset   | 能够接受的字符集                                                                         |
| Cookie           | 之前由服务器通过 Set-Cookie 发送的 Cookies                                               |
| Origin           | 发起一个针对跨来源资源共享的请求                                                         |
| Referer          | 表示浏览器所访问的前一个页面                                                             |
| User-Agent       | 浏览器的浏览器身份标识字符串                                                             |
| X-Forwarded-For  | 用于标识某个通过超文本传输协议代理或负载均衡连接到某个网页服务器的客户端的原始互联网地址 |
| X-Forwarded-Host | 一个事实标准 ，用于识别客户端原本发出的 Host 请求头部。                                  |

~~好像也没多少的样子~~
