---
title: "Http协议相关"
date: 2021-04-21T17:52:23+08:00
lastmod: 2021-04-21T17:52:23+08:00
draft: false
keywords: []
description: "manjoc'blog"
tags: ["http","dns","chrome"]
categories: ["http"]
author: "王清"
---

## http

### 运营商劫持

- [绕过运营商劫持](https://onebitbug.me/2013/12/19/escape-isp-http-hijacking/)

**# what is linux**

HTTP报文有关的笔记，以备不时之需。

HTTP报文格式

报文首部

空行（CR+LF）

报文主体

通常，不一定要有报文主体。

请求报文的报文首部

请求行

请求首部字段

通用首部字段

实体首部字段

其他

请求行：包含请求方法、请求URI和HTTP版本（还应该以回车换行符CRLF结尾），如：

GET /index.html HTTP/1.1

请求方法：

GET：最常见，向服务器请求某个资源

POST：起初用于向服务器输入数据。实际上，通常用于HTML表单数据的提交

HEAD：与GET的行为类似，但服务器返回的响应中只包含首部，不会返回主体部分

PUT：向服务器写入文档

DELETE：删除指定资源

TRACE：服务器回送收到的请求信息给客户端，主要用于诊断

CONNECT

OPTIONS：查询服务器支持的方法（通用或针对指定资源）

响应报文的报文首部

状态行

响应首部字段

通用首部字段

实体首部字段

其他

状态行：包含表明响应结果的状态码、原因短语和HTTP版本，如：

HTTP/1.1 200 OK

状态码（常见）：

1×× Informational 信息性状态码

100 Continue

101 Switching Protocols

2×× Success 成功状态码

200 OK 成功

204 No Content 成功，但不返回任何实体的主体部分

206 Partial Content 成功执行了一个范围（Range）请求

3×× Redirection 重定向状态码

301 Moved Permanently 永久性重定向，响应报文的Location首部应该有该资源的新URL

302 Found 临时性重定向，响应报文的Location首部给出的URL用来临时定位资源

303 See Other 请求的资源存在着另一个URI，客户端应使用GET方法定向获取请求的资源

304 Not Modified 客户端发送附带条件的请求（请求首部中包含如If-Modified-Since等指定首部）时，服务端有可能返回304，此时，响应报文中不包含任何报文主体。

307 Temporary Redirect 临时重定向。与302 Found含义一样。302禁止POST变换为GET，但实际使用时并不一定，307则更多浏览器可能会遵循这一标准，但也依赖于浏览器具体实现。

4×× Client Error 客户端错误状态码

400 Bad Request 请求报文中存在语法错误

401 Unauthorized需要认证，会有适当的首部一同返回

404 Not Found 服务器上无法找到请求的资源

5×× Server Error 服务器错误状态码

500 Internel Server Error 服务端在执行请求时发生了错误

503 Service Unavailable 服务器暂时无法提供服务，可以包含Retry-After首部

首部字段

HTTP首部字段按照实际用途被分为通用首部字段（General Header Fields）、请求首部字段（Request Header Fields）、响应首部字段（Response Header Fields）和实体首部字段（Entity Header Fields）。

按照首部字段在有代理时的不同行为，首部字段又可以分为端到端首部（End-to-end Header）和逐跳首部（Hop-by-hop Header）。逐跳首部只对单次转发有效，经过缓存或代理后不再转发，HTTP/1.1和之后的版本中，要使用逐跳首部时需提供Connection首部字段。端到端首部则会一直发送给最终接收目标。

通用首部字段

通用信息性首部字段：

Connection，两个作用：

控制不再转发给代理的首部字段（即逐跳首部）。应用程序会删除报文中所有在Connection首部中出现过的首部，如下示例：

**# 客户端请求首部GET / HTTP/1.1**

Upgrade: HTTP/1.1

Connection: Upgrade

**# 经过代理服务器后发送给Web服务器的首部**

GET / HTTP/1.1

管理持久连接

Connection: close HTTP/1.1默认都是持久连接，使用close后会明确断开连接

Connection: keep-alive HTTP/1.1之前的版本默认都是非持久连接，使用keep-alive可以维持持久连接

Date 创建HTTP报文的时间和日期

Trailer 说明在报文主体后记录了哪些首部字段

Transfer-Encoding 传输报文主体时采用的编码方式

Upgrade 用于检测HTTP协议及其他协议是否可使用更高的版本进行通信

Via 追踪客户端与服务器之前的请求和响应报文的传输路径

通用缓存首部字段：

Cache-Control 管理缓存信息，是HTTP/1.1引入的一个复杂首部。

请求指令

no-cache 客户端不接收缓存过的响应

no-store 不缓存响应或请求的任何内容

max-age = [秒] 告诉缓存服务器，如果缓存时间没超过指定时间，就返回缓存

max-stale( = [秒]) 即使缓存过期，只要小于该值，也照常接收

min-fresh = [秒]

no-transform 缓存不能改变实体主体的媒体类型

only-if-cached 只有缓存服务器有缓存指定资源时才返回，否则，返回504 Gateway Timeout

cache-extension

响应指令

public 其他用户也可以使用该缓存

private 只有特定用户能使用该缓存

no-cache 缓存服务器可以缓存，但是每次提供给客户端前都必须与服务器确认有效期

no-store 不缓存响应或请求的任何内容

no-transform 缓存不能改变实体主体的媒体类型

must-revalidate 返回缓存时，必须再次验证。会使max-stale无效

proxy-revalidate 告知缓存服务器，客户端带有该指令时必须验证缓存有效性

max-age = [秒] 在指定时间内不需要像源服务器确认。HTTP/1.1优先处理max-age，HTTP/1.0优先处理Expires

s-maxage = [秒] 与max-age功能相同，但s-maxage只适用于供多位用户使用的公共缓存服务器

cache-extension

Pragma HTTP/1.1以前的遗留字段Pargma: no-cache与Cache-Control: no-cache功能一致，只用在客户端发送请求时

请求首部字段

请求信息性首部字段：

From 请求来自何方，格式是客户端用户的有效电子邮件地址

Host 服务器的主机名和端口号

Referer 这次请求的URL是从哪里获得的

User-Agent 客户端的浏览器或代理信息

Accept首部字段：

Accept 客户端通过该首部字段告诉服务器自己可以接收哪些媒体类型，如text/html、image/**、**/*。此外，还有可以权重系数（q值）来表示媒体类型的优先级。

Accept-Charset 客户端可以接收哪些字符集，也可以有q值

Accept-Encoding 客户端支持的内容编码及内容编码的优先级顺序。

gzip 由文件压缩程序gzip生成的编码格式

compress 由UNIX文件压缩程序compress生成的编码格式

deflate 组合使用zlib格式及由deflate压缩算法生成的编码格式

identify 不执行压缩或不会变化的默认编码格式

Accept-Language 客户端能够处理的自然语言集（中文、英文等）

TE 客户端能够处理的传输编码，还可以指定伴随trailer字段的分块传输编码方式

条件请求首部字段：

Expect 客户端通过该首部字段告知服务器它们需求某种行为，现在该首部与响应码100 Continue紧密相关。如果服务器无法理解该首部的值，就应该返回417 Expectation Failed

If-Match 服务器会比对该字段的值和资源的ETag值，仅当两者一致时，才会执行请求，否则，返回412 Precondition Failed。该字段值为*时，会忽略ETag值

If-Modified-Since 该字段值应该是一个日期，如果服务器上资源的更新时间较该字段值新则处理该请求，否则，返回304 Not Modified

If-None-Match 与If-Match相反，该字段的值与请求资源的ETag不一致时，处理该请求

If-Range 该字段的值（ETag或时间）与资源的ETag或时间一致时，作为范围请求处理（参加首部字段Range）。否则，返回全体资源

If-Unmodified-Since 与If-Modified-Since相反，服务器上资源的更新时间早于该字段值时处理请求，否则，返回412 Precondition Failed

Range 范围请求，只获取部分资源。如Range: bytes=5001-10000，表示获取从第5001字节至10000字节的资源。成功处理范围请求时返回206 Partial Content响应，无法处理范围请求时返回200 OK响应及全部资源

安全请求首部字段：

Authorization 向服务器回应自己的身份验证信息。客户端收到来自服务器的401 Authentication Required响应后，要在其请求中包含这个首部

Cookie HTTP/1.1中没有定义，用于客户端识别和跟踪的扩展首部

代理请求首部字段：

Max-Forwards 只能和TRACE方法一起使用，指定经过代理或其他中间节点的最大数目。每个收到带此首部的TRACE请求的应用程序，在请求转发之前都要将这个值减1；如果应用程序收到请求时，该首部值为0，则立即回应一条200 OK响应

Proxy-Authorization 与Authorization类似，用于客户端与代理服务器之间的身份验证

响应首部字段

响应信息性首部字段：

Age 响应已经产生了多长时间。HTTP/1.1规定缓存服务器在创建响应时必须包含Age首部

Location 客户端应重定向到指定URI，基本配合3**响应出现

Retry-After 告诉客户端多久之后再次发送请求。主要配合503 Service Unavailable使用，或与3**响应一起使用

Server HTTP服务器的应用程序信息

Warning

协商首部字段：

Accept-Ranges 服务器是否能处理范围请求，bytes表示能，none表示不能

Vary

通知客户端，服务器端的协商中会使用哪些来自客户端请求的首部

缓存控制：对某次请求，响应报文的Vary中会指定一些首部名称，客户端后续请求相同资源时，这些首部与缓存的那次请求完全一致时才会返回缓存的资源

安全响应首部字段：

Proxy-Authorizate 与WWW-Authenticate类似，用于代理与客户端之间的认证，407 Proxy Authentication Required响应必须包含该首部

Set-Cookie 非HTTP/1.1标准首部

WWW-Authenticate 告诉客户端访问所请求资源的认证方案，401 Unauthorized响应中肯定有该首部

实体首部字段

实体首部字段是在请求报文和响应报文中的实体部分所使用的首部，用于补充内容的更新时间等与实体相关的信息

实体信息性首部字段：

Allow 通知客户端可以对特定资源使用那些HTTP方法。405 Method Not Allowed响应中必须包含该首部内容首部字段：

Content-Encoding 告诉客户端实体的主体部分选用的内容编码方式。具体方式参见Accept-Encoding

Content-Language 告诉客户端实体主体使用的自然语言（中文、英文等）

Content-Length 表明实体主体部分的大小（单位：字节）。对实体主体进行内容编码传输时，不能再使用该首部字段

Content-Location 报文主体部分相对应的URI

Content-MD5 一串由MD5算法生成的值。对于检查在传输过程中数据是否被无意的修改非常有用，但不能用于安全目的，因为报文如果被有意的修改，该字段的值也可以计算后作相应修改

Content-Range 针对范围请求，提供了请求实体在原始实体内的位置（范围），还给出了整个实体的长度

Content-Type 响应报文中对象的媒体类型

实体缓存首部字段：

ETag 实体标记，就是一种标识资源的方式

Expires 资源失效日期，当Cache-Control有指定max-age指令时，会优先处理max-age

Last-Modified 资源最终修改时间