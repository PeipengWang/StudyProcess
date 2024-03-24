# commet与websocket
##  Comet
### 前言
Comet是一种用于web的技术，能使服务器能实时地将更新的信息传送到客户端，而无须客户端发出请求，目前有两种实现方式，长轮询和iframe流。
实现方式
#### 长轮询
长轮询是在打开一条连接以后保持，等待服务器推送来数据再关闭的方式。

#### iframe流
iframe流方式是在页面中插入一个隐藏的iframe，利用其src属性在服务器和客户端之间创建一条长链接，服务器向iframe传输数据（通常是HTML，内有负责插入信息的javascript），来实时更新页面。 iframe流方式的优点是浏览器兼容好，Google公司在一些产品中使用了iframe流，如Google Talk。
### 发展历程
#### 从http协议说起

1996年IETF  HTTP工作组发布了HTTP协议的1.0版本 ，到现在普遍使用的版本1.1，HTTP协议经历了17 年的发展。这种分布式、无状态、基于TCP的请求/响应式、在互联网盛行的今天得到广泛应用的协议，相对于互联网的迅猛发展，它似乎进步地很慢。互联网从兴起到现在，经历了门户网站盛行的web1.0时代，而后随着ajax技术的出现，发展为web应用盛行的web2.0时代，如今又朝着web3.0的方向迈进。反观http协议，从版本1.0发展到1.1，除了默认长连接之外就是缓存处理、带宽优化和安全性等方面的不痛不痒的改进。它一直保留着无状态、请求/响应模式，似乎从来没意识到这应该有所改变。
#### Ajax---脚本发送的http请求
传统的web应用要想与服务器交互，必须提交一个表单（form），服务器接收并处理传来的表单，然后返回全新的页面，因为前后两个页面的数据大部分都是相同的，这个过程传输了很多冗余的数据、浪费了带宽。于是Ajax技术便应运而生。
Ajax是Asynchronous JavaScript and XML的简称，由Jesse James Garrett 首先提出。这种技术开创性地允许浏览器脚本（JS）发送http请求。Outlook Web Access小组于98年使用，并很快成为IE4.0的一部分，但是这个技术一直很小众，直到2005年初，google在他的goole groups、gmail等交互式应用中广泛使用此种技术，才使得Ajax迅速被大家所接受。
Ajax的出现使客户端与服务器端传输数据少了很多，也快了很多，也满足了以丰富用户体验为特点的web2.0时代 初期发展的需要，但是慢慢地也暴露了他的弊端。比如无法满足即时通信等富交互式应用的实时更新数据的要求。这种浏览器端的小技术毕竟还是基于http协议，http协议要求的请求/响应的模式也是无法改变的，除非http协议本身有所改变。
#### Comet---一种hack技术

以即时通信为代表的web应用程序对数据的Low Latency要求，传统的基于轮询的方式已经无法满足，而且也会带来不好的用户体验。于是一种基于http长连接的“服务器推”技术便被hack出来。这种技术被命名为Comet，这个术语由Dojo Toolkit 的项目主管Alex Russell在博文Comet: Low Latency Data for the Browser首次提出，并沿用下来。
其实，服务器推很早就存在了，在经典的client/server模型中有广泛使用，只是浏览器太懒了，并没有对这种技术提供很好的支持。但是Ajax的出现使这种技术在浏览器上实现成为可能， google的gmail和gtalk的整合首先使用了这种技术。随着一些关键问题的解决（比如 IE 的加载显示问题），很快这种技术得到了认可，目前已经有很多成熟的开源Comet框架。
以下是典型的Ajax和Comet数据传输方式的对比，区别简单明了。典型的Ajax通信方式也是http协议的经典使用方式，要想取得数据，必须首先发送请求。在Low Latency要求比较高的web应用中，只能增加服务器请求的频率。Comet则不同，客户端与服务器端保持一个长连接，只有客户端需要的数据更新时，服务器才主动将数据推送给客户端。
### Comet的实现主要有两种方式：
基于Ajax的长轮询（long-polling）方式
　　浏览器发出XMLHttpRequest 请求，服务器端接收到请求后，会阻塞请求直到有数据或者超时才返回，浏览器JS在处理请求返回信息（超时或有效数据）后再次发出请求，重新建立连接。在此期间服务器端可能已经有新的数据到达，服务器会选择把数据保存，直到重新建立连接，浏览器会把所有数据一次性取回。
　　![在这里插入图片描述](https://img-blog.csdnimg.cn/e39c419cf40d415a86a10a3d8bbc1db6.png)

2.  基于 Iframe 及 htmlfile 的流（http streaming）方式

　　Iframe是html标记，这个标记的src属性会保持对指定服务器的长连接请求，服务器端则可以不停地返回数据，相对于第一种方式，这种方式跟传统的服务器推则更接近。

在第一种方式中，浏览器在收到数据后会直接调用JS回调函数，但是这种方式该如何响应数据呢？可以通过在返回数据中嵌入JS脚本的方式，如“<script type="text/javascript">js_func(“data from server ”)</script>”，服务器端将返回的数据作为回调函数的参数，浏览器在收到数据后就会执行这段JS脚本。
![在这里插入图片描述](https://img-blog.csdnimg.cn/0b6f2465ac344f41b7857814c9668c9d.png)


但是这种方式有一个明显的不足之处：IE、Morzilla Firefox 下端的进度栏都会显示加载没有完成，而且 IE 上方的图标会不停的转动，表示加载正在进行。Google 的天才们使用一个称为“htmlfile”的 ActiveX 解决了在 IE 中的加载显示问题，并将这种方法应用到了 gmail+gtalk 产品中。
## websocket
#### Websocket---未来的解决方案
      如果说Ajax的出现是互联网发展的必然，那么Comet技术的出现则更多透露出一种无奈，仅仅作为一种hack技术，因为没有更好的解决方案。Comet解决的问题应该由谁来解决才是合理的呢？浏览器，html标准，还是http标准？主角应该是谁呢？本质上讲，这涉及到数据传输方式，http协议应首当其冲，是时候改变一下这个懒惰的协议的请求/响应模式了。
      W3C给出了答案，在新一代html标准html5中提供了一种浏览器和服务器间进行全双工通讯的网络技术Websocket。从Websocket草案得知，Websocket是一个全新的、独立的协议，基于TCP协议，与http协议兼容、却不会融入http协议，仅仅作为html5的一部分。于是乎脚本又被赋予了另一种能力：发起websocket请求。这种方式我们应该很熟悉，因为Ajax就是这么做的，所不同的是，Ajax发起的是http请求而已。
      与http协议不同的请求/响应模式不同，Websocket在建立连接之前有一个Handshake（Opening Handshake）过程，在关闭连接前也有一个Handshake（Closing Handshake）过程，建立连接之后，双方即可双向通信。

#### Opening Handshake
客户端发起连接Handshake请求
```
GET /chat HTTP/1.1
Host: server.example.com
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Key: dGhlIHNhbXBsZSBub25jZQ==
Origin: http://example.com
Sec-WebSocket-Protocol: chat, superchat
Sec-WebSocket-Version: 13
```
服务器端响应：
```
HTTP/1.1 101 Switching Protocols
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Accept: s3pPLMBiTxaQ9kYGzzhZRbK+xOo=
Sec-WebSocket-Protocol: chat
```
“Upgrade：WebSocket”表示这是一个特殊的 HTTP 请求，请求的目的就是要将客户端和服务器端的通讯协议从 HTTP 协议升级到 WebSocket 协议。
“Sec-WebSocket-Key”是一段浏览器base64加密的密钥
“Sec-WebSocket-Accept”服务器端在接收到的Sec-WebSocket-Key密钥后追加一段神奇字符串“258EAFA5-E914-47DA-95CA-C5AB0DC85B11”，并将结果进行sha-1哈希，然后再进行base64加密返回给客户端。
“Sec-WebSocket-Protocol”表示客户端请求提供的可供选择的子协议，及服务器端选中的支持的子协议，“Origin”服务器端用于区分未授权的websocket浏览器
“HTTP/1.1 101 Switching Protocols”中101为服务器返回的状态码，所有非101的状态码都表示handshake并未完成。
Data Framing
Websocket协议通过序列化的数据包传输数据。数据封包协议中定义了opcode、payload length、Payload data等字段。具体封包格式如下图所示：
![在这里插入图片描述](https://img-blog.csdnimg.cn/d5a1af04c07e4b03ab69ac29e655f593.png)

FIN: 标识是否为此消息的最后一个数据包，占 1 bit
RSV1, RSV2, RSV3:  用于扩展协议，一般为0，各占1bit
Opcode：数据包类型（frame type），占4bits
0x0：标识一个中间数据包
0x1：标识一个text类型数据包
0x2：标识一个binary类型数据包
0x3-7：保留
0x8：标识一个断开连接类型数据包
0x9：标识一个ping类型数据包
0xA：表示一个pong类型数据包
0xB-F：保留
Payload length：Payload data的长度，占7bits，如果这个值等于126，则此后16bits用于表示Payload length，如果这个值等于127，则此后64bits用于表示Payload length
Payload data：应用层数据
#### Closing Handshake
相对于Opening Handshake，Closing Handshake则简单得多，主动关闭的一方向另一方发送一个关闭类型的数据包，对方收到此数据包之后，再回复一个相同类型的数据包，关闭完成。
关闭类型数据包遵守封包协议，Opcode为0x8，Payload data可以用于携带关闭原因或消息。 
虽然现阶段websocket协议还处于草案阶段，不过浏览器早就开始开始支持了，以下是不同浏览器兼容列表:
![在这里插入图片描述](https://img-blog.csdnimg.cn/0c4786cdf4114847ae20fa9b00804130.png)

