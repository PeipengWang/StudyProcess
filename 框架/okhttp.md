# okhttp
[参考](https://www.cnblogs.com/it-tsz/p/11748674.html)
https://blog.csdn.net/mynameishuangshuai/article/details/51303446
Dispatcher dispatcher: 设置请求的调度程序。
Proxy proxy: 设置代理服务器。
List<Protocol> protocols: 设置支持的协议列表。
List<ConnectionSpec> connectionSpecs: 设置连接规范。
List<Interceptor> interceptors: 设置拦截器列表，用于处理请求和响应。
List<Interceptor> networkInterceptors: 设置网络拦截器列表，用于在网络层拦截和处理请求和响应。
ProxySelector proxySelector: 设置代理服务器选择器。
CookieJar cookieJar: 设置 Cookie 管理器。
Cache cache: 设置响应缓存。
InternalCache internalCache: 设置内部缓存。
SocketFactory socketFactory: 设置 Socket 工厂。
SSLSocketFactory sslSocketFactory: 设置 SSL Socket 工厂。
HostnameVerifier hostnameVerifier: 设置主机名验证器。
CertificatePinner certificatePinner: 设置证书锁定器。
Authenticator proxyAuthenticator: 设置代理服务器的身份验证器。
Authenticator authenticator: 设置客户端的身份验证器。
ConnectionPool connectionPool: 设置连接池。
Dns dns: 设置 DNS 解析器。
boolean followSslRedirects: 设置是否自动遵循 SSL 重定向。
boolean followRedirects: 设置是否自动遵循重定向。
boolean retryOnConnectionFailure: 设置是否在连接失败时自动重试。
int connectTimeout: 设置连接超时时间。
int readTimeout: 设置读取超时时间。
int writeTimeout: 设置写入超时时间。

几个实例：
1、get方法请求
```
OkHttpClient client = new OkHttpClient();

String run(String url) throws IOException {
  Request request = new Request.Builder()
      .url(url)
      .build();

  Response response = client.newCall(request).execute();
  return response.body().string();
}
```
2、post方法请求
```
public static final MediaType JSON
    = MediaType.parse("application/json; charset=utf-8");

OkHttpClient client = new OkHttpClient();

String post(String url, String json) throws IOException {
  RequestBody body = RequestBody.create(JSON, json);
  Request request = new Request.Builder()
      .url(url)
      .post(body)
      .build();
  Response response = client.newCall(request).execute();
  return response.body().string();
}

```
提交键值对
```
OkHttpClient client = new OkHttpClient();
String post(String url, String json) throws IOException {
    RequestBody formBody = new FormEncodingBuilder()
    .add("platform", "android")
    .add("name", "bug")
    .add("subject", "XXXXXXXXXXXXXXX")
    .build();

    Request request = new Request.Builder()
      .url(url)
      .post(body)
      .build();

    Response response = client.newCall(request).execute();
    if (response.isSuccessful()) {
        return response.body().string();
    } else {
        throw new IOException("Unexpected code " + response);
    }
}
```