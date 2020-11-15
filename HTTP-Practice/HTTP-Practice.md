# HTTP Protocol 朝花夕拾

——常见且容易被**忽视**的内容



> 　　WebAPI 以及智能终端（Android、iOS、Ajax）的出现，要求我们加强对 HTTP 协议的理解。
>
> <small align="center">-- *[舌尖上的大胖](https://www.jianshu.com/u/9a09120967a0)*</small> 😋



[TOC]





## 常用工具

### [httpbin](http://httpbin.org/)

- HTTP Request & Response Service, written in Python + Flask.

- 可本地部署

### [Postman](https://www.getpostman.com/)

### [cURL](https://zh.wikipedia.org/wiki/CURL)

　　cURL 是一个利用 URL 语法在命令行下工作的文件传输工具。支持的通信协议有 FTP、FTPS、HTTP、HTTPS、TFTP、SFTP、Gopher、SCP、Telnet、DICT、FILE、LDAP、LDAPS、IMAP、POP3、SMTP 和 RTSP。



#### cURL 的使用

- GET

```bash
$ curl "http://httpbin.org/anything?param1=value1&param2=value2"
```

- POST

```bash
$ curl -d "item1=111&item2=222" "http://httpbin.org/anything?param1=value1&param2=value2"
```

使用数据文件

```bash
$ curl -d @data.txt http://httpbin.org/anything
```

- PUT

```bash
$ curl -X PUT -d "item1=111&item2=222" "http://httpbin.org/anything?param1=value1&param2=value2"
```

- DELETE

```bash
$ curl -X DELETE -d "item1=111&item2=222" "http://httpbin.org/anything?param1=value1&param2=value2"
```

- Cookie
  - 命令行传 Cookie 值

```bash
$ curl -b "Cookie_1=value1; Cookie_2=value2" "http://httpbin.org/anything"
```

- - 使用文件传 Cookie 值，如：cookie.txt

```bash
$ curl -b cookie.txt "http://httpbin.org/anything"
```

Cookie 文件格式：

```plain
Set-Cookie:k1=v1; Path=/
Set-Cookie:k2=v2; Path=/
Set-Cookie:k3=v3; Path=/
```

- 上传文件

```bash
$ curl --form "fileupload=@filename.txt" http://httpbin.org/anything
```

- 下载文件
  - `-o`（小写）输出到命令行中提供的文件名
  - `-O`（大写）输出到 URL 中带的文件名

```bash
$ curl -o mygettext.html http://www.gnu.org/software/gettext/manual/gettext.html
```

```bash
$ curl -O http://www.gnu.org/software/gettext/manual/gettext.html
```



## 基本规范

### URI = URN + URL

- [URI（统一资源标志符）](https://zh.wikipedia.org/wiki/%E7%BB%9F%E4%B8%80%E8%B5%84%E6%BA%90%E6%A0%87%E5%BF%97%E7%AC%A6)格式

```
scheme:[//[user[:password]@]host[:port]][/path][?query][#fragment]
```

```
                    hierarchical part
        ┌───────────────────┴─────────────────────┐
                    authority               path
        ┌───────────────┴───────────────┐┌───┴────┐
  abc://username:password@example.com:123/path/data?key=value&key2=value2#fragid1
  └┬┘   └───────┬───────┘ └────┬────┘ └┬┘           └─────────┬─────────┘ └──┬──┘
scheme  user information     host     port                  query         fragment

  urn:example:mammal:monotreme:echidna
  └┬┘ └──────────────┬───────────────┘
scheme              path
```

### URI 编码

- 先 UTF-8，后 percentEncoded
- 工具：[Web前端助手（FeHelper）](https://www.baidufe.com/fehelper)



### 常用 HTTP Method 及 REST

|   Method   |         标准 HTTP 中的作用         | REST 中的作用 |
| :--------: | :--------------------------------: | :-----------: |
|   `GET`    |                访问                |   获取资源    |
|   `POST`   |                提交                |   新增操作    |
|  `DELETE`  |                删除                |   删除资源    |
|   `PUT`    |                上传                |   修改资源    |
| **`HEAD`** | **只获取 HTTP Response 的 HEADER** |      N/A      |



## 基本认证

### HTTP Basic Auth

- 作用：HTTP 基本认证（其余认证方式参考 Postman）
- 效果演示
- 使用场景：如 Subversion 的权限控制
- 客户端请求方式
  - iOS、Android、Ajax 各 HTTP Client 自己的方式
  - HTTP Request Header

形式：

```http
Authorization: Basic Base64(Username:Password)
```

例如：

```http
Authorization: Basic QWxhZGRpbjpvcGVuIHNlc2FtZQ==
```

优点：

- 标准 HTTP 支持，实现简单，无需特殊服务器支持
- 不需要 HTTP Client 做特殊实现
- 对于安全级别要求不高的场景，配合 HTTPS 可以低成本实现服务器安全

缺点：

- 认证信息只做 Base64 编码，没有加密，不够安全
- 不能针对不同用户设置不同的密码



### 访问 HTTP Basic Auth

当访问一个需要 Basic Auth 服务的时候，如果没有带有认证信息，服务端会返回 401，Response 头部带有 `WWW-Authenticate`：

```http
HTTP/1.1 401 UNAUTHORIZED
WWW-Authenticate: Basic realm="Fake Realm"
```



### HTTP Basic Auth 配置方法（Nginx)

```nginx
server {
    auth_basic "Hello World";
    auth_basic_user_file passwd.txt;
}
```

或者

```nginx
server {
    location /xxx {
        auth_basic "Hello World";
        auth_basic_user_file passwd.txt;
    }
}
```



## 常见 HTTP HEADER 及作用

详情参考 [*HTTP Headers*](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers)，这里只列出部分常用头。

### Request 部分

##### `Content-Type`

指定内容格式，以 POST 请求举例，提交三组值：

```
key1=val1
key2=val2
key3=val3
```

当 `Content-Type` 不同时，需要以不同的格式组织提交内容（抓包查看通讯结果）

- `multipart/form-data`

```http
Content-Type: multipart/form-data; boundary=--------------------------059012866576624836076831
Content-Length: 377


----------------------------059012866576624836076831
Content-Disposition: form-data; name="key1"

val1
----------------------------059012866576624836076831
Content-Disposition: form-data; name="key2"

val2
----------------------------059012866576624836076831
Content-Disposition: form-data; name="key3"

val3
----------------------------059012866576624836076831--

```



- `application/x-www-form-urlencoded`

```http
Content-Type: application/x-www-form-urlencoded
Content-Length: 29

key1=val1&key2=val2&key3=val3
```



- `application/json`

```http
Content-Type: application/json
Content-Length: 56


{
  "key1": "val1",
  "key2": "val2",
  "key3": "val3"
}
```



##### `Cache-Control` 

- `no-cache`
- `no-store`
- `max-age=xxx`



##### `Content-Length`

指定内容长度



##### `User-Agent`

```http
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:12.0) Gecko/20100101 Firefox/12.0
```



##### `Authorization`

```http
Authorization: Basic QWxhZGRpbjpvcGVuIHNlc2FtZQ==
```



##### `Range`

　　在 HTTP 头中，`Range` 都表示“资源的 byte 形式数据的顺序排列，并且取其某一段数据”的意思。`Range` 头就是表示请求资源的从某个数值到某个数值间的数据，例如：

```http
Range: bytes=500-999
```

就是表示请求资源从 500 到 999 byte 的数据。数据的分段下载和多线程下载就是利用这个实现的。

就是表示请求资源从 500 到 999 byte 的数据。数据的分段下载和多线程下载就是利用这个实现的。

　　**这些功能要求服务端支持 *Range Request***



### Response 部分

##### `Content-Type`

作用：Response Body 部分的数据类型，用于告诉客户端返回的数据类型，以便做针对特定数据类型做相应处理。（代码演示浏览器对于相同 Body 的内容，不同的 Header 的处理方式）

应用场景：

- 浏览器根据 Response 的 Content-Type 决定内部打开或下载
- HTTP Client（iOS、Android、Ajax）请求数据时，根据 Response 的 Content-Type，决定服务端返回的 JSON 是作为对象处理，还是字符串处理



###### 常见的 Content-Type

- `text/plain`

  表示文本文件的默认值。一个文本文件应当是人类可读的，并且不包含二进制数据。

- `application/octet-stream`
  表示所有其他情况的默认值。一种未知的文件类型应当使用此类型。浏览器在处理这些文件时会特别小心，试图避免用户的危险行为。

- *`text/html`*

- *`application/json`*

- ...

[完整的MIME类型列表](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Basics_of_HTTP/MIME_types/Complete_list_of_MIME_types)

```java
try {
    resp.getWriter().print("<h1 style=\"color: yellow\">hello world</h1>");
    resp.setHeader("Content-Type", "application/octet-stream");
    resp.setHeader("Content-Disposition", "attachment; filename=test.txt;");
} catch (Exception e) {
    System.out.println(e);
}
```



**浏览器行为：**浏览器会根据 Response Header 中的 Content-Type 来判定内容的类型。

    1. 对于浏览器内置支持的类型，比如：Chrome 支持 PDF 的查看，当 Content-Type 为 `application/pdf` 时，浏览器会在内部进行打开；（指定了 `Content-Disposition` 的情况除外）
    2. 对于浏览器内置支持的类型，如果指定了 `Content-Disposition` 为附件，会启动下载；
    3. 对于浏览器不能识别的类型，或者没有明确指定类型的情况，如：`application/octet-stream` 也会启动下载。





##### `Content-Disposition`

```http
Content-Disposition: attachment; filename=xxx.xxx;
```

指定内容作为附件下载，同时指定下载使用的文件名，浏览器会使用这个文件名作为提示。如果文件名中包含敏感字符或者中文，可以约定使用 **先 UTF-8，后 percentEncoded** 的顺序进行编码。

如果为了更好的兼容性，可以在内容中指定编码的方式，如：

```http
Content-Disposition: attachment; filename=xxx.xxx; filename*=utf8''abc.xyz;
```

这样的目的是让低版本的浏览器可以直接从 `filename` 中直接读取，高版本浏览器可以从 `filename*` 中根据编码读取。对于文件名的内容，也是采用先按指定规则编码，然后 percentEncoded 编码的顺序进行。



##### `Content-Length`
作用：返回的数据长度

应用场景：可用于显示下载进度。（Charles Throttle 测试）



##### `Cache-Control`（详见 HTTP Cache）

- `no-cache`
- `no-store`
- `max-age=xxx`

作用：指定 Response 返回的内容的缓存策略

应用场景：由服务端指定资源的缓存控制方式



#### 跨域

- 避免其他网站脚本的恶意攻击
- `<script>`、`<img>` 等标签中 `src` 属性不受跨域的限制
- 可以使用 `jsonp` 的方式解决跨域限制
- 页面以**本地**方式集成到 [Cordova](https://cordova.apache.org/) 和 [Electron](https://electronjs.org/) 这样的外壳中时，不受跨域限制
- H5 的新特性，如果服务器允许其他站点的脚本访问的时候，需要在 Response Header 中指定：

```http
Access-Control-Allow-Origin: *
Access-Control-Allow-Methods: GET, POST
```

例如：http://httpbin.org



## HTTP 重定向

### HTTP Response 301 - Moved Permanently

- 作用：客户端根据 301 返回头中的 `location` 字段进行跳转
- 重定向是永久的重定向，搜索引擎在抓取新内容的同时也将旧的网址替换为重定向之后的网址。
- `location` 头

```http
location: http://www.baidu.com
```

### HTTP Response 302 - Found

302 重定向表示临时性转移（Temporarily Moved），当一个网页URL需要短期变化时使用。 

302 重定向是临时的重定向，搜索引擎会抓取新的内容而保留旧的网址。

### HTTP Response 303 - See Other / 307 Temporary Redirect

由于 302 的重定向不会管原始请求的方法，都会以 GET 方式访问重定向的位置，所以在 HTTP 1.1 中，302 拆分为 303 和 307，303 的行为保持与 302 一致，307 则可以保持与原始请求相同的方法访问重定向的位置，并且消息主体不会发生改变。





![HTTP Redirects](img/image-redirect.png)





## HTTP Cookie

### Cookie

- 浏览器及 Postman 中的体现
- 主要属性：`Domain`、`Path`、`ExpireDate`
- 其他属性
  - `Security`：只在使用 HTTPS 时才会带上该 Cookie
  - `HTTPOnly`：只通过 HTTP 处理，JS 不能处理
- 使用场景
  - 标识：CAS、验证码
  - 认证
- [Cookie 与 Session 的关系](http://justsee.iteye.com/blog/1570652)

- 跨域：相同域名/IP的服务即为同域，不受端口影响，这一点与 Ajax 的跨域有所区别。但Cookie的有效性还与 Path 有关



## HTTPS

参考资料：

[《理解 HTTPS 的工作原理》](http://www.codeceo.com/article/https-worker.html)

HTTPS 和 HTTP 协议相比提供了：

1. 数据完整性：内容传输经过完整性校验
2. 数据隐私性：内容经过对称加密，每个连接生成一个唯一的加密密钥
3. 身份认证：第三方无法伪造服务端（客户端）身份

其中，数据完整性和隐私性由 TLS Record Protocol 保证，身份认证由 TLS Handshaking Protocols 实现。

![Network Layer](img/image-network-layer.png)

![HTTPS](img/image-https.jpg)

- 作用
  - 通道加密（可通过抓包看结果）
  - 身份验证
    - 单向
    - 双向

- 证书
  - 公钥证书：`.crt`、`.pem`
  - 私钥：`.key`

![Certification](img/image-certification.png)





### HTTPS 的部署（Nginx）

服务端校验配置：

```nginx
server {
    listen              443 ssl;
    ssl_certificate     nginx.crt;
    ssl_certificate_key nginx.key;
}
```

打开客户端校验：

```nginx
server {
    ssl_client_certificate client_ca.crt;
    ssl_verify_client on;
}
```



### HTTPS 自签名证书的使用

- 什么是自签名证书（如：12306、银行）
- 与标准证书区别
- 为什么使用自签名证书
- 自签名证书的制作（OpenSSL）
- 自签名证书的类型（CA、基本证书）
- 在浏览器中的认证
- 在手机中的使用
  - 全局安装
  - App 集成



### 证书链

#### 证书链介绍

![img](img/image-certificate-chain.png)

　　有两种类型的证书颁发机构（CA）：根 CA 和中间 CA。如果想让一个设备信任一个证书，必须保证该证书设备信任的 CA 颁发。
　　如果证书不是由受信任的 CA 签发的，则连接设备（例如，网页浏览器）将检查发行 CA 的证书是否由可信 CA 颁发，等等，直到找到可信 CA 为止 （此时将建立可信的安全连接），或者找不到可信的 CA（此时设备通常会显示错误）。

　　SSL 证书列表（从根证书到最终用户证书）代表 SSL 证书链。

![img](img/image-certificate-chain-example.png)



#### 如果我不安装中级SSL证书会发生什么情况？

　　如果您未安装一个或多个中级 SSL 证书，则会破坏证书链。 这意味着您在特定（最终用户或中间）证书与其颁发者之间建立了缝隙。 当设备找不到证书的受信任颁发者时，证书和从中间证书直到最终证书的整个链都不可信。

　　因此，您的最终证书不会被信任。 Web浏览器将显示“无效证书”或“证书不可信”错误。

#### Nginx 中证书链的配置

证书整合

```bash
$ cat www.example.com.crt bundle.crt > www.example.com.chained.crt
```



```nginx
server {
    listen              443 ssl;
    server_name         www.example.com;
    ssl_certificate     www.example.com.chained.crt;
    ssl_certificate_key www.example.com.key;
    ...
}
```





### HTTPS 证书的验证（OpenSSL）

标准 HTTPS（非自签名） 服务器的验证

```bash
# 模拟一个客户端访问 SSL 服务器
$ openssl s_client -connect 192.168.20.51:443
```

自签名 HTTPS 服务器的验证

```bash
# 模拟一个客户端访问 SSL 服务器，自行指定证书（对于浏览器，证书需要导入）
$ openssl s_client -connect 192.168.20.51:443 -CAfile cafile.pem
```

双向验证时带上客户端证书

```bash
# 如果服务端要求客户端提供证书  则在加上 -cert 和-key参数
$ openssl s_client -connect 192.168.20.51:443 -cert client.crt -key client.key
```

验证证书合法性

```bash
# 验证一个证书是否是某一个CA签发，文件格式必须为 PEM
$ openssl verify -CAfile cafile.pem apache.crt
```









### 中间人攻击

![Man-in-the-middle Attack­](img/image-man-in-the-middle-attack.png)



## HTTP Cache

*背景：主要用于优化静态资源访问。但是对于动态数据，可能带来负面的影响。*

早期浏览器不允许开发人员自定义请求头，所以都是利用 Response 来指定缓存策略，在下次 Request 的时候携带相关标记。随着 Ajax 和 App 的发展，允许开发人员利用这些标记来控制缓存的使用。


| Response Header                     | Request Header                        | 作用                                                         | 引发问题                                             | 备注                                                         |
| ----------------------------------- | ------------------------------------- | ------------------------------------------------------------ | ---------------------------------------------------- | ------------------------------------------------------------ |
| `Expires: 过期具体时间`             | N/A                                   | 指定 Cache 内容过期时间                                      | 依赖于客户端时间                                     |                                                              |
| `Cache-Control: max-age=有效期秒数` | N/A                                   | 指定 Cache 内容有效期（秒）                                  | Cache 过期但服务端内容无变化时，仍旧会进行数据下载。 | 与 Expires 同时具备时，优先级高于 Expires。即使 Cache-Control 指定的有效期更长，也遵循此规则。参见 [RFC2616](https://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.9.3) |
| `Cache-Control: no-cache`           | `Cache-Control: no-cache`             | 使用 Cache 前检查服务端资源是否有更新。                      |                                                      | 服务端内容无变化时，返回 304                                 |
| `Cache-Control: no-store`           | `Cache-Control: no-store`             | 不使用缓存。                                                 | 不使用本地缓存，产生流量浪费。                       | N/A                                                          |
| `Last-Modified: 资源最后修改时间`   | `If-Modified-Since: 资源最后修改时间` | 标识服务端内容最后修改时间。即便 Cache 超期，服务端也可以告知客户端继续使用 Cache。 | “修改时间”变化不代表内容也变了                       | 服务端内容无变化时，返回 304                                 |
| `ETag: ETag 值`                     | `If-None-Match: ETag值`               | 使用 ETag 值判断服务端内容是否变化。                         | N/A                                                  | 服务端内容无变化时，返回 304                                 |

组合使用：

- `If-Modified-Since` 与 `If-None-Match` 组合使用，可以实现本地 Cache 未过期则继续使用，过期了检测服务端是否有变化，没变化也继续使用本地缓存。



### HTTP Cache 中一些关键差异

- 返回值差异
  - 对于缓存没有超期的情况
    在浏览器的 Dev Tool 中查看，仍会返回 `200`，只不过会有 `from disk cache` 或 `from memory cache` 的标识。但使用抓包工具看不到网络请求。
  - 对于缓存超期的情况
    如果服务端无变化的情况，服务端会返回 `304` 指示继续使用缓存的内容。
- `Cache-Control` 中取值的两个区别
  - `no-cache` 与 `no-store` 的区别
    - `no-cache` 每次都会检查服务端的内容是否改变，如果改变了才使用服务端的内容。
    - `no-store` 永远不会使用 Cache。
  - `no-cache` 与 `max-age=0` 的区别
    没搞懂



### 缓存规则在 SPA 页面中的使用建议

- 首页 index.html 最好指定为 `Cache-Control: no-cache`，甚至 `Cache-Control: no-store`，以保证随时获取最新内容
- 对于页面中引用的 JS 以及 CSS 等零散内容进行 Bundle，并充分利用缓存。页面在引用 Bundle 时，以 Hash 值或版本区分 Bundle 内容，避免缓存导致内容不更新
- 图片使用 `If-Modified-Since` 与 `If-None-Match` 组合方式使用缓存



### HTTP Cache 遗留问题

- 服务端的缓存策略如何设置？
- `Cache-Control` 的 `max-age=0` 与 `no-cache` 的区别？



### HTTP Cache 相关参考

- [《循序漸進理解 HTTP Cache 機制》](https://blog.techbridge.cc/2017/06/17/cache-introduction/)

- [《HTTP 缓存（Google Web Fundamentals）》](https://developers.google.com/web/fundamentals/performance/optimizing-content-efficiency/http-caching?hl=zh-cn)



## HTTP Proxy



### HTTP Proxy 种类

- Proxy（代理）

- Reverse Proxy（反向代理）

- 简易区分方法

  访问的 URL 与实际 URL 是否一致，一致则为**正向代理**，否则为**反向代理**。

- 常用服务器：Nginx、Apache

- 便捷开发工具：基于 Nodejs 的 http-server



### HTTP Reverse Proxy 简单配置

- Nginx

```nginx
server {
    listen              28080;
    location /hb/ {
        proxy_pass http://httpbin.org/;
    }
}
```

- http-server

```bash
hs -p28080 --cors --utc -c-1 --proxy http://httpbin.org/
```



## HTTP 相关的 Nginx 常用配置

### 文件服务器

```nginx
server {
    listen              29080;
    location /webroot/ {
        root /Volumes/Doc/MyCode/Nginx/;
        autoindex on;
    }
}
```



### 反向代理

```nginx
server {
    listen              28080;
    location /hb/ {
        proxy_pass http://httpbin.org/;
    }
}
```



### 负载均衡（最简单的轮询情况）

```nginx
http {
    # ... 省略其它配置
    upstream tomcats {
        server 192.168.0.100:8080;
        server 192.168.0.101:8080;
        server example.com:8080;
    }

    server {
        listen 80;
        location / {
            proxy_pass http://tomcats;
        }
    }
}
```

除了最简单的轮询之外，还有 weight、max_fails、fail_timeout、backup、down、max_conns、resolve 等参数，详见相关文档。

第三方的负载策略，包括 [Fair](https://www.nginx.com/resources/wiki/modules/fair_balancer/)、url_hash 等。



## 长连接

- HTTP 长连接方案
  - Polling（轮询）
  - Long-Polling（长时间轮询）
  - Streaming（串流）
- Web Socket（其实与 HTTP 没啥关系）



![複雜的 Comet 網頁應用程式架構](img/image-comet.jpg)



参考资料：

[《WebSocket 通訊協定簡介：比較 Polling、Long-Polling 與 Streaming 的運作原理》](https://blog.gtwang.org/web-development/websocket-protocol/)

