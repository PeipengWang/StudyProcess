## 一、什么是SSE

服务器向浏览器推送信息，除了 WebSocket，还有一种方法：Server-Sent Events（以下简称 SSE）。

SSE（Server-Sent Events）是一种用于实现服务器主动向客户端推送数据的技术，也被称为“事件流”（Event Stream）。它基于 HTTP 协议，利用了其长连接特性，在客户端与服务器之间建立一条持久化连接，并通过这条连接实现服务器向客户端的实时数据推送。

SSE 是HTML5规范的一部分，该规范非常简单，主要由两部分组成：第一部分是服务端与浏览器端的通讯协议（Http协议），第二部分是浏览器端可供JavaScript使用的EventSource对象。

严格意义上来说，Http协议是无法做到服务器主动想浏览器发送协议，但是可以变通下，服务器向客户端发起一个声明，我下面发送的内容将是 text/event-stream 格式的，这个时候浏览器就知道了。响应文本内容是一个持续的数据流，每个数据流由不同的事件组成，并且每个事件可以有一个可选的标识符，不同事件内容之间只能通过回车符\r 和换行符\n来分隔,每个事件可以由多行组成。目前除了IE和Edge，其他浏览器均支持。

## 二、SSE技术的基本原理

客户端向服务器发送一个GET请求，带有指定的header，表示可以接收事件流类型，并禁用任何的事件缓存。

服务器返回一个响应，带有指定的header，表示事件的媒体类型和编码，以及使用分块传输编码（chunked）来流式传输动态生成的内容。

服务器在有数据更新时，向客户端发送一个或多个名称：值字段组成的事件，由单个换行符分隔。事件之间由两个换行符分隔。服务器可以发送事件数据、事件类型、事件ID和重试时间等字段。

客户端使用EventSource接口来创建一个对象，打开连接，并订阅onopen、onmessage和onerror等事件处理程序来处理连接状态和接收消息。

客户端可以使用GET查询参数来传递数据给服务器，也可以使用close方法来关闭连接。

## 三、SSE适用于场景

SSE适用场景是指服务器向客户端实时推送数据的场景，例如：

股票价格更新：服务器可以根据股市的变化，实时地将股票价格推送给客户端，让客户端能够及时了解股票的走势和行情。

新闻实时推送：服务器可以根据新闻的更新，实时地将新闻内容或标题推送给客户端，让客户端能够及时了解最新的新闻动态和信息。

在线聊天：服务器可以根据用户的发送，实时地将聊天消息推送给客户端，让客户端能够及时收到和回复消息。

实时监控：服务器可以根据设备的状态，实时地将监控数据或报警信息推送给客户端，让客户端能够及时了解设备的运行情况和异常情况。

SSE适用场景的特点是：

数据更新频繁：服务器需要不断地将最新的数据推送给客户端，保持数据的实时性和准确性。

低延迟：服务器需要尽快地将数据推送给客户端，避免数据的延迟和过期。

单向通信：服务器只需要向客户端推送数据，而不需要接收客户端的数据。

chatGPT 返回的数据 就是使用的SSE 技术

实时数据大屏 如果只是需要展示 实时的数据可以使用SSE技术 而不是非要使用webSocket

## 四 实例

 新建springboot项目，包含一个原生的index.html页面和一个http接口，页面请求接口，接口读取一个txt文件并将文件内容流式输出到页面。[点击查看github源码](https://github.com/wsliliang/sse-springboot-demo)

```
<body>
<button type="button" onclick="output()">输出文章</button>
<div id="message"></div>
<script>
    function output() {
        let source = new EventSource(
            'http://localhost:8080/article');
        let innerHTML = '';
        source.onmessage = function (e) {
            if (e.data == '[done]') {
                source.close();
            } else {
                innerHTML += e.data;
                document.getElementById("message").innerHTML = innerHTML;
            }
        };
    }
</script>
</body>
```



```
@Controller
public class SseController {
    @Autowired
    private ResourceLoader resourceLoader;
​
    @RequestMapping("/article")
    public void sendArticle(HttpServletResponse res) throws Exception {
        res.setContentType("text/event-stream;charset=UTF-8");
        Resource resource = resourceLoader.getResource("classpath:1.txt");
        InputStream inputStream = resource.getInputStream();
        BufferedReader reader = new BufferedReader(new InputStreamReader(inputStream));
        String line;
        while ((line = reader.readLine()) != null) {
            // 这里用空格来将一行内容分为多个单词输出
            String delimiter = " ";
            String[] words = line.split(delimiter);
            // 为了支持将中文句子分为单个汉子输出
            if (words.length == 1) {
                delimiter = "";
                words = line.split(delimiter);
            }
            for (int i = 0; i < words.length; i++) {
                // 每次只输出一个词,每个片段以data: 开头，以\n\n结尾
                res.getWriter().write("data: " + words[i] + delimiter + "\n\n");
                res.getWriter().flush();
                // 睡眠100ms,便于观察效果
                Thread.sleep(100);
            }
            // 由于每次读的是一行数据，因此输出一个换行
            res.getWriter().write("data: </br>\n\n");
        }
        // 也用[done]来标识结束
        res.getWriter().write("data: [done]\n\n");
        res.getWriter().flush();
    }
}
```

#### 五、Node服务端示例

EventSource这个api是一个用于接收服务器发送事件（Server-Sent Events，SSE）的Web API接口。服务器发送事件是一种让服务器端能够主动向客户端发送数据的技术，它使用HTTP协议，并且遵循一定的格式。

##### 1、协议

SSE 协议非常简单，正常的Http请求，更改请起头相关配置即可

```
Content-Type: text/event-stream,utf-8
Cache-Control: no-cache
Connection: keep-alive
```

2、格式
文本流基础格式如下，以行为单位的，以冒号分割 Field 和 Value，每行结尾为 \n，每行会Trim掉前后空字符，因此 \r\n 也可以。
每一次发送的信息，由若干个message组成，每个message之间用\n\n分隔。每个message内部由若干行组成，每一行都是如下格式。

```
field: value\n
field: value\r\n
```

Field是有5个固定的name

```
data     // 数据内容
event    // 事件
id       // 数据标识符用id字段表示，相当于每一条数据的编号
retry    // 重试,服务器可以用retry字段，指定浏览器重新发起连接的时间间隔
:        //冒号开头是比较特殊的，表示注释
```

3、事件
每个事件之间通过空行来分隔。对于每一行来说，冒号（“:”）前面表示的是该行的类型，冒号后面则是对应的值。可能的类型包括：

类型为空白，表示该行是注释，会在处理时被忽略。
类型为 data，表示该行包含的是数据。以 data 开头的行可以出现多次。所有这些行都是该事件的数据。
类型为 event，表示该行用来声明事件的类型。浏览器在收到数据时，会产生对应类型的事件。
类型为 id，表示该行用来声明事件的标识符。
类型为 retry，表示该行用来声明浏览器在连接断开之后进行再次连接之前的等待时间。

3.1、事件

事件之间用\n\n隔断，一般一个事件一行，也可以多行

```
# 一个事件一行
data: message\n\n
data: message2\n\n


# 一个事件多行
data: {\n
data: "name": "zhangsan",\n
data: "age", 25\n
data: }\n\n

# 自定义事件
event: foo\n   // 自定义事件，名称 foo，触发客户端的foo监听事件
data: a foo event\n\n // 内容

data: an unnamed event\n\n // 默认事件，未指定事件名称，触发客户端 onmessage 事件

event: bar\n   // 自定义时间，名称 bar，触发客户端bar监听事件
data: a bar event\n\n // 内容
```

###### 3.2、事件唯一标识符

每个事件可以指定一个ID,浏览器会跟踪事件ID，如果发生了重连，浏览器会把最近接收到的时间ID放到 HTTP Header Last-Event-ID 中，作为一种简单的同步机制。

```
id: eef0128b-48b9-44f7-bbc6-9cc90d32ac4f\n
data: message\n\n
```

###### 3.3、重连事件

中断连接，客户端一般会3秒重连，但是服务端也可以配置

```
retry: 10000\n
```

##### 4、具体示例

在Node.js中使用Server-Sent Events（SSE），你可以创建一个[HTTP服务器](https://so.csdn.net/so/search?q=HTTP服务器&spm=1001.2101.3001.7020)，并使用res对象的write方法来向客户端发送持久的流信息。

```
const http = require('http');

const server = http.createServer((req, res) => {
    // 对于SSE请求，需要设置正确的Content-Type和Cache-Control
    // 设置Content-Type头为text/event-stream，这是SSE所需的。
    // 设置Cache-Control头为no-cache，这将防止客户端缓存事件流。
    // 使用Connection: keep-alive，这将保持连接打开，直到服务器明确地关闭连接。
    res.writeHead(200, {
        'Content-Type': 'text/event-stream',
        'Cache-Control': 'no-cache',
        'Connection': 'keep-alive',
        'Access-Control-Allow-Credentials': true,
        'Access-Control-Allow-Origin': '*'
    });

    // 每隔一定时间发送一次消息
    // 使用res.write()方法将新的数据发送到客户端。
    // 我们在数据前添加了data:前缀，这是SSE事件流所需的。
    // 最后，我们在每个事件之间添加了一个空行\n\n，这是SSE事件流规范要求的，以便客户端可以正确解析事件流
    setInterval(() => {
        const data = `data: ${new Date().toISOString()}\n\n`;
        res.write(data);
    }, 1000);

    // 当客户端断开连接时，清理资源
    req.on('close', () => {
        clearInterval(); // 清除定时器
        res.end(); // 结束响应
    });
});

server.listen(3000, () => {
    console.log('Server is running on port 3000');
});
```

#### 五、客户端示例

##### 1、检测客户端是否支持SSE

```
function supportsSSE() {
  return !!window.EventSource;
}
```

##### 2、创建客户端连接

客户端实现就比较简单了，实例化一个EventSource对象,url 可以和服务端同域，也可以跨域，如果跨域的话，需要指定第二个参数withCredentials:true，表示发送Cookie到服务端

```
//创建一个EventSource对象，用于从sse.php页面接收事件
const evtSource = new EventSource("http://localhost:3000");
```

##### 3、事件监听

事件源连接后会发送 “open” 事件，可以通过以下两种方式监听

```
# 方式一:
source.onopen = function(event) {
  // handle open event
};

## 方式二：
source.addEventListener("open", function(event) {
  // handle open event
}, false);
```

##### 4、接收事件

接收事件同样和上面同样有两种方式。浏览器会自动把一个消息中的多个分段拼接成一个完整的字符串，因此，可以轻松地在这里使用 JSON 序列化和反序列化处理。

```
# 方式一:
source.onmessage = function(event) {
  var data = event.data;
  var lastEventId = event.lastEventId;
  // handle message
};

## 方式二：
source.addEventListener("message", function(event) { 
  var data = event.data;
  var lastEventId = event.lastEventId;
  // handle message }, false);
```

##### 5、自定义事件

默认情况下，服务器发送过来的消息，都是默认事件格式的数据，这个时候都会触发onmessage，如果后端自定义事件的话，则不会触发onmessage,这个是否我们需要添加对应的监听事件

```
source.addEventListener('foo', function (event) {
  var data = event.data;
  // handle message
}, false);
```

##### 6、错误处理

```
# 方式一:
source.onerror = function(event) {
  // handle error event
};

## 方式二：
source.addEventListener("error", function(event) {
  // handle error event
}, false);
```

##### 7、主动断开连接

```
source.close()
```

##### 8、具体示例

```
// 先判断当前浏览器是否支持EventSource事件
if (typeof(EventSource) !== "undefined") {
  // 用JavaScript创建了一个新的EventSource对象，它将从服务器端的/test/server-sent-events路由接收事件流
  const source = new EventSource('/test/server-sent-events', {
    withCredentials: true
  });
  
  每当服务器发送新的事件时，source.onmessage事件处理程序将运行，我们将事件数据解析为JSON对象
  source.onmessage = (event) => {
     const data = JSON.parse(event.data)
     console.log(data)
  }
  
  // close方法用于关闭 SSE 连接。
  source.close();
 
} else {
  console.log('不支持 EventSource')
}

```

#### 七、注意事项

##### 1、nginx配置

- 使用nginx做反向代理时需要将proxy_buffering关闭

  ```
  proxy_buffering off
  
  ```

- 或者加上响应头部x-accel-buffering，这样nginx就不会给后端响应数据加buffer

  ```
  x-accel-buffering: no
  ```

  