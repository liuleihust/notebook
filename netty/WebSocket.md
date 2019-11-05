## 简介

  **` http` 是基于 `tcp/ip`的，它也可以具有双向主动发送信息的能力，为什么 `web` 服务器不支持推送呢 ？**

大概就是 `http` 是给`web服务`使用的协议，而`web服务`什么最重要？当然是并发数了。如果采用`tcp`那样的有状态协议，只要一个用户开着浏览器，那么他与服务器的`http`连接就不会被切断。但是一个服务器能够同时开启的连接数是有限制的，这样的话这个网站就变成专门为这几百个人服务的了，即使靠增加服务器的方法，每增加一台服务器也就增加几百人而已。试想在这种情况下，`google`、`twitter`这些每秒访问量达到几百万上千万级别的网站怎么办？或者说一个接地气的例子，每当长假或年底的时候，每个人应该都很清楚`12306`是什么样一种蛋疼的状况吧？这种情况又该怎么办？

`Websocket`确实是有状态的，和`TCP`很像，但它的出现不是为了取代`http`，是为了当确实需要在网页中实现有状态连接时（例如网游）可以使用的一种辅助技术。

[构建消息推送系统之HTTP长连接实践](<https://blog.csdn.net/wangxindong11/article/details/78523350>)



> `websocket ` 相比于 `http` ,应该 冗余信息更少。 

首先Websocket是基于HTTP协议的，或者说**借用**了HTTP的协议来完成一部分握手。
在握手阶段是一样的（**握手阶段使用的是http 协议**）：

```http
GET /chat HTTP/1.1
Host: server.example.com
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Key: x3JJHMbDL1EzLkh9GBhXDw==
Sec-WebSocket-Protocol: chat, superchat
Sec-WebSocket-Version: 13
Origin: http://example.com
```

熟悉HTTP的童鞋可能发现了，这段类似HTTP协议的握手请求中，多了几个东西。
我会顺便讲解下作用。

```http
Upgrade: websocket
Connection: Upgrade
```

这个就是Websocket的核心了，告诉Apache、Nginx等服务器：**注意啦，窝发起的是Websocket协议，快点帮我找到对应的助理处理~不是那个老土的HTTP。**

```http
Sec-WebSocket-Key: x3JJHMbDL1EzLkh9GBhXDw==
Sec-WebSocket-Protocol: chat, superchat
Sec-WebSocket-Version: 13
```

首先，`Sec-WebSocket-Key` 是一个Base64 encode的值，这个是浏览器随机生成的，告诉服务器：**泥煤，不要忽悠窝，我要验证尼是不是真的是Websocket助理。**
然后，`Sec_WebSocket-Protocol` 是一个用户定义的字符串，用来区分同URL下，不同的服务所需要的协议。简单理解：**今晚我要服务A，别搞错啦~**
最后，`Sec-WebSocket-Version` 是告诉服务器所使用的Websocket Draft（协议版本），在最初的时候，Websocket协议还在 Draft 阶段，各种奇奇怪怪的协议都有，而且还有很多期奇奇怪怪不同的东西，什么Firefox和Chrome用的不是一个版本之类的，当初Websocket协议太多可是一个大难题。。不过现在还好，已经定下来啦~大家都使用的一个东西~ 脱水：**服务员，我要的是13岁的噢→_→**



然后服务器会返回下列东西，表示已经接受到请求， 成功建立Websocket啦！

```http
HTTP/1.1 101 Switching Protocols
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Accept: HSmrc0sMlYUkAGmm5OPpG2HaGWk=
Sec-WebSocket-Protocol: chat
```

```http
Upgrade: websocket
Connection: Upgrade
```

依然是固定的，告诉客户端即将升级的是Websocket协议,`Sec-WebSocket-Accept` 这个则是经过服务器确认，并且加密过后的 `Sec-WebSocket-Key`。服务器：**好啦好啦，知道啦，给你看我的ID CARD来证明行了吧。。**
后面的，`Sec-WebSocket-Protocol` 则是表示最终使用的协议。

至此，HTTP已经完成它所有工作了，接下来就是完全按照Websocket协议进行了。

