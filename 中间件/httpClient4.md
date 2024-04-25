# Httpclient
尽管 java.net 包提供了通过 HTTP 访问资源的基本功能，但它并没有提供许多应用程序所需的全部功能。HttpClient填补了Java.net中在HTTP请求处理方面的一些不足，提供了更丰富、更灵活、更高效的HTTP请求处理能力。  
HttpClient 专为扩展而设计，同时为基本 HTTP 协议提供强大的支持，任何构建 HTTP 感知客户端应用程序，例如 Web 浏览器、Web 服务客户端或利用或扩展 HTTP 协议进行分布式通信的系统。  

## 基础
### 请求执行模板
HttpClient最本质的功能是执行HTTP方法。一个 HTTP 方法的执行涉及一个或多个 HTTP 请求/HTTP 响应交换，通常由 HttpClient 内部处理。用户需要提供一个要执行的请求对象，HttpClient 将请求传输到目标服务器，返回相应的响应对象，如果执行不成功，则抛出异常。  
HttpClient API 的主要入口点是定义上述协定的 HttpClient 接口。  

以下是最简单形式的请求执行过程的示例：
```
CloseableHttpClient httpclient = HttpClients.createDefault();
HttpGet httpget = new HttpGet("http://localhost/");
CloseableHttpResponse response = httpclient.execute(httpget);
try {
    <...>
} finally {
    response.close();
}
```
### HTTP请求构造
#### 概述
 HTTP 请求都有一个请求行，其中包含请求方法、请求URI和HTTP的版本协议  
 接下来还有请求体和请求头  
#### 请求行
 HttpClient通过URIBuilder的方式来构造请求  
```
 URI uri = new URIBuilder() 
        .setScheme("http") 
        .setHost("www.google.com") 
        .setPath("/search") 
        .setParameter("q", "httpclient") 
        .setParameter("btnG" , "Google search") 
        .setParameter("aq", "f") 
        .setParameter("oq", "") 
        .build(); 
HttpGet httpget = new HttpGet(uri); 
System.out.println(httpget.getURI());
```
上述实际发送的请求为http://www.google.com/search?q=httpclient&btnG=Google+Search&aq=f&oq=
也可以进行Post，Put之类的请求体的构造，其中最重要的是请求体，可以借助HttpEntity的方式来放入数据
HttpEntity是一个核心接口，它定义了HTTP实体的通用行为和属性。HttpEntity接口位于org.apache.http.HttpEntity包中，其主要作用是表示HTTP消息的内容，包括请求和响应的实体内容。
#### 请求体
HttpEntity接口的实现类包括：  
	BasicHttpEntity： 提供了基本的HttpEntity实现，适用于大多数情况。  
	BufferedHttpEntity： 是HttpEntityWrapper的一个子类，用于包装HttpEntity，并缓冲实体内容，以便多次读取。  
	ByteArrayEntity： 将实体内容表示为字节数组。  
	FileEntity： 将实体内容表示为文件。  
	StringEntity： 将实体内容表示为字符串。  
	InputStreamEntity： 将实体内容表示为输入流。  
HttpEntity接口和其实现类提供了丰富的方法，可以用于获取和操作HTTP实体的各种属性和内容，例如获取内容长度、获取内容类型、获取内容编码、获取内容流等。这些类和接口的设计使得HttpClient能够灵活处理HTTP请求和响应的实体内容。  

例如：
```
StringEntity myEntity = new StringEntity("important message", 
   ContentType.create("text/plain", "UTF-8"));
System.out.println(myEntity.getContentType());
System.out.println(myEntity.getContentLength());
System.out.println(EntityUtils.toString(myEntity));
System.out.println(EntityUtils.toByteArray(myEntity).length);
```
输出:
```
Content-Type: text/plain; charset=utf-8
17
important message
17
```
将请求数据传入请求体，例如FileEntity  
```
File file = new File("somefile.txt");
FileEntity entity = new FileEntity(file,ContentType.create("text/plain", "UTF-8"));        

HttpPost httppost = new HttpPost("http://localhost/action.do");
httppost.setEntity(entity);
```
#### 模拟表单提交
有些应用需要表单的方式来提交数据，HttpClient提供了实体类 UrlEncodedFormEntity来进行封装这个过程。
```
List<NameValuePair> formparams = new ArrayList<NameValuePair>();
formparams.add(new BasicNameValuePair("param1", "value1"));
formparams.add(new BasicNameValuePair("param2", "value2"));
UrlEncodedFormEntity entity = new UrlEncodedFormEntity(formparams, Consts.UTF_8);
HttpPost httppost = new HttpPost("http://localhost/handler.do");
httppost.setEntity(entity);
```
#### 设置分块编码
分块编码是一种用于在HTTP传输中动态生成并发送数据的方法。它允许数据在发送过程中被分成多个块（chunks），每个块的大小可以根据需要而变化。这种方法使得在数据传输过程中不需要提前知道整个数据的大小，而是可以根据实际情况动态生成块并发送，从而提高了数据传输的效率和灵活性。
```
StringEntity entity = new StringEntity("important message",
        ContentType.create("plain/text", Consts.UTF_8));
entity.setChunked(true);
HttpPost httppost = new HttpPost("http://localhost/acrtion.do");
httppost.setEntity(entity);
```
### HTTP响应
#### 响应基本测试
响应也是类似的
```
HttpResponse response = new BasicHttpResponse(HttpVersion.HTTP_1_1, 
HttpStatus.SC_OK, "OK");

System.out.println(response.getProtocolVersion());
System.out.println(response.getStatusLine().getStatusCode());
System.out.println(response.getStatusLine().getReasonPhrase());
System.out.println(response.getStatusLine().toString
```
输出为
```
HTTP/1.1
200
OK
HTTP/1.1 200 OK
```
#### 获取响应的请求内容
响应 息可以包含许多描述消息属性的标头，例如内容长度、内容类型等。HttpClient 提供了检索、添加、删除和枚举标头的方法。
```
HttpResponse response = new BasicHttpResponse(HttpVersion.HTTP_1_1, 
    HttpStatus.SC_OK, "OK");
response.addHeader("Set-Cookie", 
    "c1=a; path=/; domain=localhost");
response.addHeader("Set-Cookie", 
    "c2=b; path=\"/\", c3=c; domain=\"localhost\"");
Header h1 = response.getFirstHeader("Set-Cookie");
System.out.println(h1);
Header h2 = response.getLastHeader("Set-Cookie");
System.out.println(h2);
Header[] hs = response.getHeaders("Set-Cookie");
System.out.println(hs.length);
```
枚举  
```
HttpResponse response = new BasicHttpResponse(HttpVersion.HTTP_1_1, 
    HttpStatus.SC_OK, "OK");
response.addHeader("Set-Cookie", 
    "c1=a; path=/; domain=localhost");
response.addHeader("Set-Cookie", 
    "c2=b; path=\"/\", c3=c; domain=\"localhost\"");

HeaderIterator it = response.headerIterator("Set-Cookie");

while (it.hasNext()) {
    System.out.println(it.next());
}
```
响应数据  
这个对用户来说是最重要的，官方推荐获取实体内容采用HttpEntity#getContent()或 HttpEntity#writeTo(OutputStream)方法。
同时官方还附带了一个工具类EntityUtils，这个类提供了更方便的从实体类中获取内容和信息的方法  

实例  
```
CloseableHttpClient httpclient = HttpClients.createDefault();
HttpGet httpget = new HttpGet("http://localhost/");
CloseableHttpResponse response = httpclient.execute(httpget);
try {
    HttpEntity entity = response.getEntity();
    if (entity != null) {
        long len = entity.getContentLength();
        if (len != -1 && len < 2048) {
            System.out.println(EntityUtils.toString(entity));
        } else {
            // Stream content out
        }
    }
} finally {
    response.close();
}
```
有时候也需要多次多去实体类，这种情形可以采用缓冲方式，实现这种最简单的是BufferedHttpEntity。这将导致原始实体的内容被读入内存缓冲区
```
CloseableHttpResponse response = <...>
HttpEntity entity = response.getEntity();
if (entity != null) {
    entity = new BufferedHttpEntity(entity);
}
```

#### 响应处理封装
如果我们对于响应的处理想要封装一层，直接得到我们想要得到的值，可以考虑采用ResponseHandler的方式，具体使用流程为
1、定义一个类实现ResponseHandler，并设置泛型响应类型
2、实现handleResponse方法，从response返回想要得到的响应类型
3、调用excute，返回定义的泛型类型
实例如下：
```

import org.apache.http.HttpResponse;
import org.apache.http.client.ResponseHandler;
import org.apache.http.util.EntityUtils;

public class MyResponseHandler<T> implements ResponseHandler<T> {
    @Override
    public T handleResponse(HttpResponse response) throws IOException {
        // 从响应中提取数据并进行处理
        // 例如，将响应内容解析为特定类型的对象
        if (response.getStatusLine().getStatusCode() == 200) {
            // 从响应中提取实体内容并转换为字符串
            String responseBody = EntityUtils.toString(response.getEntity());
            // 进行其他处理，例如将字符串转换为对象
            // 返回处理结果
            return (T) responseBody;
        } else {
            // 处理非200状态码的响应，可以抛出异常或返回默认值等
            return null;
        }
    }
}

import org.apache.http.client.HttpClient;
import org.apache.http.client.methods.HttpGet;
import org.apache.http.impl.client.HttpClients;

public class Main {
    public static void main(String[] args) throws IOException {
        // 创建HttpClient实例
        HttpClient httpClient = HttpClients.createDefault();
        // 创建HttpGet请求对象
        HttpGet httpGet = new HttpGet("http://example.com/api/resource");
        // 执行HTTP请求并处理响应
        String response = httpClient.execute(httpGet, new MyResponseHandler<>());
        // 输出处理结果
        System.out.println("Response: " + response);
    }
}

```
上述代码中T为String  

### 释放资源
当不再需要某个实例CloseableHttpClient，必须通过调用该CloseableHttpClient#close() 方法来关闭与其关联的连接管理器。  
```
CloseableHttpClient httpclient = HttpClients.createDefault();
try {
    <...>
} finally {
    httpclient.close();
}
```
同时input资源也要注意释放

### HTTP执行上下文
最初，HTTP 被设计为无状态、面向响应请求的协议。然而，现实世界的应用程序通常需要能够通过几个逻辑相关的请求-响应交换来持久保存状态信息。为了使应用程序能够维持处理状态，HttpClient 允许在特定的执行上下文（称为 HTTP 上下文）内执行 HTTP 请求。如果在连续请求之间重用相同的上下文，则多个逻辑相关的请求可以参与逻辑会话。  
HttpContext是一个接口，它提供了在HTTP请求执行期间存储和检索信息的机制。它允许您在多个HTTP请求之间共享状态信息，例如在同一会话中保持会话状态或跟踪身份验证信息。  
HTTP执行上下文可以用于在HTTP请求之间传递自定义参数，以便在请求执行期间访问它们。它通常用于在整个请求执行过程中保持状态，并在需要时将其传递给相关的HTTP组件。  
在HttpClient中，HTTP执行上下文通常作为方法参数传递给执行HTTP请求的方法，例如execute方法。它是一个接口，允许用户自定义实现以满足特定的需求。
在 HTTP 请求执行过程中，HttpClient 将以下属性添加到执行上下文中：  
HttpConnection代表与目标服务器的实际连接的实例。  
HttpHost代表连接目标的实例。  
HttpRoute代表完整连接路由的实例    
HttpRequest代表实际 HTTP 请求的实例。执行上下文中的最终 HttpRequest 对象始终准确地表示消息 发送到目标服务器时的状态。默认情况下，HTTP/1.0 和 HTTP/1.1 使用相对请求 URI。但是，如果请求是通过代理以非隧道模式发送的，则 URI 将是绝对的。  
HttpResponse代表实际 HTTP 响应的实例。  
java.lang.Boolean表示标志的对象，该标志指示实际请求是否已完全传输到连接目标。  
RequestConfig代表实际请求配置的对象。  
java.util.List<URI>表示请求执行过程中收到的所有重定向位置的集合的对象  
```
HttpContext context = <...>
HttpClientContext clientContext = HttpClientContext.adapt(context);
HttpHost target = clientContext.getTargetHost();
HttpRequest request = clientContext.getRequest();
HttpResponse response = clientContext.getResponse();
RequestConfig config = clientContext.getRequestConfig();
```
在以下示例中，初始请求设置的请求配置将保留在执行上下文中，并传播到共享相同上下文的连续请求。RequestConfig 就是设置的上下文配置
```
CloseableHttpClient httpclient = HttpClients.createDefault();
RequestConfig requestConfig = RequestConfig.custom()
        .setSocketTimeout(1000)
        .setConnectTimeout(1000)
        .build();

HttpGet httpget1 = new HttpGet("http://localhost/1");
httpget1.setConfig(requestConfig);
CloseableHttpResponse response1 = httpclient.execute(httpget1, context);
try {
    HttpEntity entity1 = response1.getEntity();
} finally {
    response1.close();
}
HttpGet httpget2 = new HttpGet("http://localhost/2");
CloseableHttpResponse response2 = httpclient.execute(httpget2, context);
try {
    HttpEntity entity2 = response2.getEntity();
} finally {
    response2.close();
}
```
### HTTP拦截器
如果某一个Httpclient需要共用一些代码，可以添加拦截器，在响应的前后执行相关逻辑  
您可以通过调用HttpClients.custom()方法获取HttpClientBuilder实例，然后使用addInterceptorLast方法添加一个HttpRequestInterceptor对象作为拦截器，示例如下：  
```
CloseableHttpClient httpclient = HttpClients.custom()
        .addInterceptorLast(new HttpRequestInterceptor() {
            @Override
            public void process(HttpRequest request, HttpContext context) throws HttpException, IOException {
                // 在此处添加您的拦截器逻辑
            }
        })
        .build();

```
在process方法中，可以编写自定义的逻辑来处理HTTP请求。这可以用于在执行请求之前或之后修改请求或响应的内容，添加标头，记录请求和响应等操作。  
详细例子  
```
CloseableHttpClient httpclient = HttpClients.custom()
        .addInterceptorLast(new HttpRequestInterceptor() {

            public void process(
                    final HttpRequest request,
                    final HttpContext context) throws HttpException, IOException {
                AtomicInteger count = (AtomicInteger) context.getAttribute("count");
                request.addHeader("Count", Integer.toString(count.getAndIncrement()));
            }

        })
        .build();

AtomicInteger count = new AtomicInteger(1);
HttpClientContext localContext = HttpClientContext.create();
localContext.setAttribute("count", count);

HttpGet httpget = new HttpGet("http://localhost/");
for (int i = 0; i < 10; i++) {
    CloseableHttpResponse response = httpclient.execute(httpget, localContext);
    try {
        HttpEntity entity = response.getEntity();
    } finally {
        response.close();
    }
}
```
### 异常处理
HTTP 协议处理器可以抛出两种类型的异常： java.io.IOException发生 I/O 故障（例如套接字超时或套接字重置）以及HttpException发出 HTTP 故障信号（例如违反 HTTP 协议）
通常 I/O 错误被认为是非致命且可恢复的，而 HTTP 协议错误被认为是致命的且无法自动恢复。

### 幂等性
默认情况下，出于兼容性原因，HttpClient 假定仅非实体封闭方法（例如 GET和HEAD）具有幂等性，而实体封闭方法（例如POST和 ）PUT则不是幂等的。
### 请求重试
官方定义了HttpRequestRetryHandler接口来实现重试代码
```
HttpRequestRetryHandler myRetryHandler = new HttpRequestRetryHandler() {

    public boolean retryRequest(
            IOException exception,
            int executionCount,
            HttpContext context) {
        if (executionCount >= 5) {
            // Do not retry if over max retry count
            return false;
        }
        if (exception instanceof InterruptedIOException) {
            // Timeout
            return false;
        }
        if (exception instanceof UnknownHostException) {
            // Unknown host
            return false;
        }
        if (exception instanceof ConnectTimeoutException) {
            // Connection refused
            return false;
        }
        if (exception instanceof SSLException) {
            // SSL handshake exception
            return false;
        }
        HttpClientContext clientContext = HttpClientContext.adapt(context);
        HttpRequest request = clientContext.getRequest();
        boolean idempotent = !(request instanceof HttpEntityEnclosingRequest);
        if (idempotent) {
            // Retry if the request is considered idempotent
            return true;
        }
        return false;
    }

};
CloseableHttpClient httpclient = HttpClients.custom()
        .setRetryHandler(myRetryHandler)
        .build();
```
这段代码创建了一个自定义的HttpRequestRetryHandler对象，并将其设置为CloseableHttpClient的重试处理程序。在这个重试处理程序中，它定义了一系列条件来确定是否应该重试请求。具体来说：
如果执行次数超过了5次，则不再重试。
如果发生了超时异常（InterruptedIOException），则不重试。
如果发生了未知主机异常（UnknownHostException），则不重试。
如果发生了连接超时异常（ConnectTimeoutException），则不重试。
如果发生了SSL握手异常（SSLException），则不重试。
如果请求是幂等的（即不包含实体的请求），则重试。
这样的设置可以根据应用程序的需求灵活调整重试策略，并在遇到特定类型的异常或请求特性时自定义重试行为。

### 终止请求
在某些情况下，由于目标服务器上的负载过高或客户端发出的并发请求过多，HTTP 请求执行无法在预期时间范围内完成。在这种情况下，可能需要提前终止请求并解除对 I/O 操作中阻塞的执行线程的阻塞。HttpClient 正在执行的 HTTP 请求可以通过调用方法在执行的任何阶段中止 HttpUriRequest#abort()。该方法是线程安全的，可以从任何线程调用。当 HTTP 请求中止时，其执行线程（即使当前在 I/O 操作中被阻塞）也可以通过抛出 InterruptedIOException
###  重定向处理
HttpClient 自动处理所有类型的重定向，但 HTTP 规范明确禁止的需要用户干预的重定向除外。See Other（状态代码 303）重定向POST并将 PUT请求转换为GETHTTP 规范要求的请求。可以使用自定义重定向策略来放宽 HTTP 规范对 POST 方法自动重定向的限制。
```
LaxRedirectStrategy redirectStrategy = new LaxRedirectStrategy(); 
CloseableHttpClient httpclient = HttpClients.custom() 
        .setRedirectStrategy(redirectStrategy) 
        .build();
```

HttpClient在执行过程中经常需要重写请求消息。默认情况下，HTTP/1.0 和 HTTP/1.1 通常使用相对请求 URI。同样，原始请求可能会多次从一个位置重定向到另一个位置。最终解释的绝对 HTTP 位置可以使用原始请求和上下文来构建。该实用程序方法URIUtils#resolve可用于构建用于生成最终请求的解释绝对 URI。此方法包括来自重定向请求或原始请求的最后一个片段标识符。
```
CloseableHttpClient httpclient = HttpClients.createDefault();
HttpClientContext context = HttpClientContext.create();
HttpGet httpget = new HttpGet("http://localhost:8080/");
CloseableHttpResponse response = httpclient.execute(httpget, context);
try {
    HttpHost target = context.getTargetHost();
    List<URI> redirectLocations = context.getRedirectLocations();
    URI location = URIUtils.resolve(httpget.getURI(), target, redirectLocations);
    System.out.println("Final HTTP location: " + location.toASCIIString());
    // Expected to be an absolute URI
} finally {
    response.close();
}

```