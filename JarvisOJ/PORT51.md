# PORT51 write-up 及 cURL 使用方法

浏览器直接打开题目链接 http://web.jarvisoj.com:32770/ 之后，页面显示着

> Please use port 51 to visit this site.

一开始看到它提示说51端口，我还以为去访问它的51端口呢。于是我就把 URL 改成了 http://web.jarvisoj.com:51/ ，结果没反应。我才反应过来，是要我用本机的51端口去访问。

那么就想了，服务器它是怎么判断我用了51端口呢，通过 X-Forwarded-For 吗？可是我现在没有装 burp，~~下次一定装~~。既然有了思路，懒得装 burp 去改 HTTP 请求了，就直接看别人的 writeup 验证一下吧。

看到 writeup 之后，我太 naive 了。真正的题解是用 curl 去指定51端口访问服务器：

```
curl --local-port 51 http://web.jarvisoj.com:32770/
```

原来 curl 还有这功能，学到了学到了。马上到自己服务器上试一下，果然可以！

```
# curl --local-port 51-52 http://web.jarvisoj.com:32770/
<!DOCTYPE html>
<html>
<head>
<title>Web 100</title>
<style type="text/css">
	body {
		background:gray;
		text-align:center;
	}
</style>
</head>

<body>
	<h3>Yeah!! Here's your flag:PCTF{__HIDE__}</h3>	
</body>
</html>
```

然后查了一下 [curl 的文档](https://curl.haxx.se/docs/manpage.html)，`--local-port` 参数可以指定一个端口，也可以指定一个端口访问。有点好奇的我再去试了一下范围访问，结果发现就不行了。

```
# curl --local-port 51-54 http://web.jarvisoj.com:32770/
curl: (45) bind failed with errno 98: Address already in use
```

这种报错是之前遇到过的，端口被占用。很可能的原因是，刚刚访问过的51端口还没有释放。因为在 TCP 的四次挥手中，当最后的 ACK 回复发出后，有个 2MSL 的时间等待 TIME_WAIT 状态。就是这个状态使到其他程序没办法那么快的再次申请这个端口。之前用到办法是端口复用，程序去申请这个端口的时候注册为可复用，这样就可以在结束连接之后，快速再次申请端口。

然而我找了一圈，都没找到 curl 有相应的参数。（网上很多都是说PHP怎么搞的，因为PHP的一个函数重名了。）最后在 superuser 上找到了[一个迂回的方法](https://superuser.com/questions/670321/curl-multiple-post-requests-while-reusing-the-tcp-connection/1463585#1463585)，就是在第一个 URL 的后面加一个参数 `--next`，然后再接上想访问的第二个 URL 。运行结果如下所示：

```
# curl --local-port 51 http://web.jarvisoj.com:32770/ --next http://web.jarvisoj.com:32770/
<!DOCTYPE html>
<html>
<head>
<title>Web 100</title>
<style type="text/css">
	body {
		background:gray;
		text-align:center;
	}
</style>
</head>

<body>
	<h3>Yeah!! Here's your flag:PCTF{__HIDE__}</h3>	
</body>
</html>
<!DOCTYPE html>
<html>
<head>
<title>Web 100</title>
<style type="text/css">
	body {
		background:gray;
		text-align:center;
	}
</style>
</head>

<body>
	<h3>Yeah!! Here's your flag:PCTF{__HIDE__}</h3>	
</body>
</html>
```

emm... 虽然不是端口复用，也算是曲线救国回来了吧。

最后总结一下，我觉得 cURL 这个工具是非常强大的工具，要好好翻一下文档。

- `-K, --config <file>` 指定一个配置文件以读取curl参数。**这个在频繁需要发送类似包体的 POST 请求中比较适用。**
- `--basic`  说明远程主机采用 HTTP Basic 验证，需要搭配 `-u|--user` 一起使用。（通常这个是默认选项，不用指定，除非要用了覆盖之前设定的其他验证模式。）
- `-u, --user <user:password>` 指定用于服务器认证的用户名和密码。
- `--connect-timeout <seconds>` 指定超时时间（以秒为单位）。
- `-m, --max-time <seconds>` 允许整个操作最长的时间（以秒为单位）。
- `-D, --dump-header <filename>` 将收到的 HTTP 响应头写入到指定文件。
- `-c, --cookie-jar <filename>` 将 HTTP 请求结束后的 response 里的 cookies 写入到指定文件。
- `-b, --cookie <data|filename>` 从 data 或文件中读取 cookies 并添加到 request 的 header 中。data 的格式是 `"NAME1=VALUE1; NAME2=VALUE2"` 。
- `-d, --data <data>` 用 POST 请求将数据发送给服务器。类似的还有 `--data-ascii <data>`, `--data-binary <data>`, `--data-raw <data>`, `--data-urlencode <data>`。同时这个选项会覆盖掉 `-F, --form`, `-I, --head`, `-T, --upload-file`。
- `-F, --form <name=content>` 模拟用户填写表单的 HTTP 请求，可以上传二进制文件。类似的有 `--form-string <name=string>`。**挺强大的这个**
- `-G, --get` 在使用了 `-d, --data <data>` 参数之后，仍然发送 GET 请求。
- `--dns-servers <addresses>` 指定 DNS 服务器。
- `--doh-url <URL>` 指定 DNS-over-HTTPS (DOH) 服务器。
- `-I, --head` 发送 HEAD 请求。
- `-H, --header <header/@file>` 附带上额外的 HTTP 请求头。
- `--http0.9` 使用 HTTP 0.9 版本。类似的有 `-0，--http1.0`, `--http1.1`, `--http2-prior-knowledge`, `--http2`, `--http3`。
- `-i，--include` 在输出中包括 HTTP 响应标头。
- `-k, --insecure` 允许不安全的 SSL 连接。
- `--interface <name>` 指定网卡。
- `-4，-ipv4` 域名仅解析成 IPv4 地址，同理有 `-6, --ipv6`。
- `--keepalive-time <seconds>` 设置在发送 keepalive 探测之前连接需要保持空闲的时间以及各个 keepalive 探测之间的时间。如果使用 `--no-keepalive` ，则此选项无效。
- `--limit-rate <speed>` 限速。
- `--local-port <num/range>` 设置用于连接的本地端口。
- `-L, --location` 如果服务器返回 3xx 响应，则追过去。可以使用 `--max-redirs` 选项来限制要遵循的重定向数量。
- `-M, --manual` Manual. Display the huge help text. **万能的**
- `-:, --next` 告诉 curl 为以下 URL 和关联的选项使用单独的操作，这样可以发送多个URL请求，每个URL请求都有其自己的特定选项。例如同时发 GET 请求和 POST 请求。
- `-o, --output <file>` 将输出写到文件，而不是 stdout 。可以使用 `{}` 或 `[]` 来获取多个文档。
- `--path-as-is` 不要处理给定的 URL 路径中 `/../` 或 `/./` 的序列。
- `--preproxy [protocol://]host[:port]` 设置前置的 SOCKS 代理。支持 `socks4://`, `socks4a://`, `socks5://`, `socks5h://`，默认是 SOCKS4 。
- `-x, --proxy [protocol://]host[:port]` 设置代理，默认是 HTTP 代理。也能支持 SOCKS 代理，结合 `--preproxy` 可以套两层，cool。
- `-U, --proxy-user <user:password>` 指定用于代理身份验证的用户名和密码。
- **未完待续...**
- `-#, --progress-bar` 显示进度条
