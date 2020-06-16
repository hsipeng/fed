# HTTP



## http/https

1.0 协议缺陷:

- 无法复用链接，完成即断开，**重新慢启动和 TCP 3次握手**
- head of line blocking: **线头阻塞**，导致请求之间互相影响

1.1 改进:

- **长连接**(默认 keep-alive)，复用
- host 字段指定对应的虚拟站点
- 新增功能:
  - 断点续传
  - 身份认证
  - 状态管理
  - cache 缓存
    - Cache-Control
    - Expires
    - Last-Modified
    - Etag

2.0:

- 多路复用
- 二进制分帧层: 应用层和传输层之间
- 首部压缩
- 服务端推送

https: 较为安全的网络传输协议

- 证书(公钥)
- SSL 加密
- 端口 443



## 常见状态码 200 301 403 500

1xx: 接受，继续处理

200: 成功，并返回数据

201: 已创建

202: 已接受

203: 成为，但未授权

204: 成功，无内容

205: 成功，重置内容

206: 成功，部分内容

301: 永久移动，重定向

302: 临时移动，可使用原有URI

304: 资源未修改，可使用缓存

305: 需代理访问

400: 请求语法错误

401: 要求身份认证

403: 拒绝请求

404: 资源不存在

500: 服务器错误





## get/ post/option/head/put/delete

- get: 缓存、请求长度受限、会被历史保存记录
  - 无副作用(不修改资源)，幂等(请求次数与资源无关)的场景
- post: 安全、大数据、更多编码类型



## websocket

Websocket 是一个 **持久化的协议**， 基于 http ， 服务端可以 **主动 push**

- 兼容：
  - FLASH Socket
  - 长轮询： 定时发送 ajax
  - long poll： 发送 --> 有消息时再 response
- `new WebSocket(url)`
- `ws.onerror = fn`
- `ws.onclose = fn`
- `ws.onopen = fn`
- `ws.onmessage = fn`
- `ws.send()`



## tcp 三次握手/四次握手

* 三次握手

  建立连接前，客户端和服务端需要通过握手来确认对方:

  * 客户端发送 syn(同步序列编号) 请求，进入 syn_send 状态，等待确认
  * 服务端接收并确认 syn 包后发送 syn+ack 包，进入 syn_recv 状态
  * 客户端接收 syn+ack 包后，发送 ack 包，双方进入 established 状态



* 四次握手
  * 客户端 -- FIN --> 服务端， FIN—WAIT
  * 服务端 -- ACK --> 客户端， CLOSE-WAIT
  * 服务端 -- ACK,FIN --> 客户端， LAST-ACK
  * 客户端 -- ACK --> 服务端，CLOSED





## 跨域 

josnp nginx 反向代理 cros 跨域资源共享

- JSONP: 利用`<script>`标签不受跨域限制的特点，缺点是只能支持 get 请求

```javascript
function jsonp(url, jsonpCallback, success) {
  const script = document.createElement('script')
  script.src = url
  script.async = true
  script.type = 'text/javascript'
  window[jsonpCallback] = function(data) {
    success && success(data)
  }
  document.body.appendChild(script)
}
```

- 设置 CORS: Access-Control-Allow-Origin：*
- postMessage




## 安全
- XSS攻击: 注入恶意代码
  - cookie 设置 httpOnly
  - 转义页面上的输入内容和输出内容
- CSRF: 跨站请求伪造，防护:
  - get 不修改数据
  - 不被第三方网站访问到用户的 cookie
  - 设置白名单，不被第三方网站请求
  - 请求校验

* 中间人攻击



## 缓存

**浏览器缓存分类**

浏览器缓存分为强缓存和协商缓存，浏览器加载一个页面的简单流程如下：

1. 浏览器先根据这个资源的http头信息来判断是否命中强缓存。如果命中则直接加在缓存中的资源，并不会将请求发送到服务器。
2. 如果未命中强缓存，则浏览器会将资源加载请求发送到服务器。服务器来判断浏览器本地缓存是否失效。若可以使用，则服务器并不会返回资源信息，浏览器继续从缓存加载资源。
3. 如果未命中协商缓存，则服务器会将完整的资源返回给浏览器，浏览器加载新资源，并更新缓存。



**强缓存**



强缓存是利用http的返回头中的Expires或者Cache-Control两个字段来控制的，用来表示资源的缓存时间。



**Expires**

缓存过期时间，用来指定资源到期的时间，是服务器端的具体的时间点。也就是说，Expires=max-age + 请求时间，需要和Last-modified结合使用。但在上面我们提到过，cache-control的优先级更高。 Expires是Web服务器响应消息头字段，在响应http请求时告诉浏览器在过期时间前浏览器可以直接从浏览器缓存取数据，而无需再次请求。

 ![img](https://images2018.cnblogs.com/blog/940884/201804/940884-20180423141609527-358000355.png)



**Cache-Control**

Cache-Control是一个相对时间，例如Cache-Control:3600，代表着资源的有效期是3600秒。由于是相对时间，并且都是与客户端时间比较，所以服务器与客户端时间偏差也不会导致问题。
Cache-Control与Expires可以在服务端配置同时启用或者启用任意一个，同时启用的时候Cache-Control优先级高。

1. **max-age** 指定一个时间长度，在这个时间段内缓存是有效的，单位是s。例如设置 Cache-Control:max-age=31536000，也就是说缓存有效期为（31536000 / 24 / 60 * 60）天，第一次访问这个资源的时候，服务器端也返回了 Expires 字段，并且过期时间是一年后。

 ![img](https://images2018.cnblogs.com/blog/940884/201804/940884-20180423141638673-1917674992.png)

在没有禁用缓存并且没有超过有效时间的情况下，再次访问这个资源就命中了缓存，不会向服务器请求资源而是直接从浏览器缓存中取。

2. **s-maxage** 同 max-age，覆盖 max-age、Expires，但仅适用于共享缓存，在私有缓存中被忽略。

3. **public** 表明响应可以被任何对象（发送请求的客户端、代理服务器等等）缓存。

4. **private** 表明响应只能被单个用户（可能是操作系统用户、浏览器用户）缓存，是非共享的，不能被代理服务器缓存。

5. **no-cache** 强制所有缓存了该响应的用户，在使用已缓存的数据前，发送带验证器的请求到服务器。不是字面意思上的不缓存。

6. **no-store** 禁止缓存，每次请求都要向服务器重新获取数据。
7. **must-revalidate**指定如果页面是过期的，则去服务器进行获取。这个指令并不常用，就不做过多的讨论了。



# 协商缓存

若未命中强缓存，则浏览器会将请求发送至服务器。服务器根据http头信息中的Last-Modify/If-Modify-Since或Etag/If-None-Match来判断是否命中协商缓存。如果命中，则http返回码为304，浏览器从缓存中加载资源。

**Last-Modify/If-Modify-Since**

浏览器第一次请求一个资源的时候，服务器返回的header中会加上Last-Modify，Last-modify是一个时间标识该资源的最后修改时间，例如Last-Modify: Thu,31 Dec 2037 23:59:59 GMT。



**ETag/If-None-Match**

与Last-Modify/If-Modify-Since不同的是，Etag/If-None-Match返回的是一个校验码（ETag: entity tag）。ETag可以保证每一个资源是唯一的，资源变化都会导致ETag变化*。ETag值的变更则说明资源状态已经被修改。服务器根据浏览器上发送的If-None-Match值来判断是否命中缓存。



# 既生Last-Modified何生Etag？

你可能会觉得使用Last-Modified已经足以让浏览器知道本地的缓存副本是否足够新，为什么还需要Etag（实体标识）呢？HTTP1.1中Etag的出现主要是为了解决几个Last-Modified比较难解决的问题：

1. Last-Modified标注的最后修改只能精确到秒级，如果某些文件在1秒钟以内，被修改多次的话，它将不能准确标注文件的修改时间

2. 如果某些文件会被定期生成，当有时内容并没有任何变化，但Last-Modified却改变了，导致文件没法使用缓存
3. 有可能存在服务器没有准确获取文件修改时间，或者与代理服务器时间不一致等情形

Etag是服务器自动生成或者由开发者生成的对应资源在服务器端的唯一标识符，能够更加准确的控制缓存。Last-Modified与ETag是可以一起使用的，服务器会优先验证ETag，一致的情况下，才会继续比对Last-Modified，最后才决定是否返回304。![img](https://images2018.cnblogs.com/blog/940884/201804/940884-20180423141945261-83532090.png)



![img](https://images2018.cnblogs.com/blog/940884/201804/940884-20180423141951735-912699213.png)