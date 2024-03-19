struts进行文件上传时装饰StrutsRequestWraper导致的问题参数映射失效问题分析  
问题：在表单提交时，如果设置表单的Content-Type属性为multipart/form-data时，就会导致提交表单的数据无法映射到对应的Action。  

例如，设置两个表单数据，如下：  

<s:hidden name="configColled" id="moduleId"/>
<s:hidden name="configColle" id="versionId"/>
无法映射到后端，导致后端获取configCollectForm.moduleId时报空指针异常。  

第二个地方：在上传模板文件时，csrfToken参数无法映射，导致后台校验失败，同时，上传文件  

<s:file name="xml" id="xml" maxlength="50" size="45"

也无法到后台  

影响：  

目前初步判断在请求头设置为enctype="multipart/form-data"时会无法映射数据

StrutsRequestWrapper

doFilter方法中调用了prepareDispatcherAndWrapRequest方法

包装出Struts2自己的request对象

在prepareDispatcherAndWrapRequest方法中调用Dispatcher类的wrapRequest方法

会根据请求内容的类型(提交的是文本的，还是multipart/form-data格式)，决定是使用tomcat的 HttpServletRequestWrapper类分离出请求中的数据，还是使用Struts2的MultiPartRequestWrapper来分离请求中的数据



Filter执行顺序：

按照web.xml的配置执行的，其中StrutsPrepareAndExecuteFilter.java在OMMP中倒数第二个执行

```java
tryHandleRequest(chain, request, response, uri);--》 handleRequest(chain, request, response, uri);
```

在这个方法中会获得所需的request

```java
HttpServletRequest wrappedRequest = prepare.wrapRequest(request);
```

是这样定义的：

```java
public HttpServletRequest wrapRequest(HttpServletRequest request) throws IOException {
// don't wrap more than once
if (request instanceof StrutsRequestWrapper) {
return request;
}

if (isMultipartSupportEnabled(request) && isMultipartRequest(request)) {
MultiPartRequest multiPartRequest = getMultiPartRequest();
LocaleProviderFactory localeProviderFactory = getContainer().getInstance(LocaleProviderFactory.class);

request = new MultiPartRequestWrapper(
multiPartRequest,
request,
getSaveDir(),
localeProviderFactory.createLocaleProvider(),
disableRequestAttributeValueStackLookup
);
} else {
request = new StrutsRequestWrapper(request, disableRequestAttributeValueStackLookup);
}

return request;
}
```

它检查传入的 HttpServletRequest 对象是否已经是 StrutsRequestWrapper 的实例（即已经被包装过）。如果已经被包装，就不再重复包装，而是直接返回原始的 HttpServletRequest 对象。

检查是否需要文件上传处理：代码接下来会检查是否启用了文件上传功能，这是通过 isMultipartSupportEnabled 和 isMultipartRequest 方法来判断的。如果文件上传功能已经启用，并且当前请求是一个多部分请求（multipart request），则会处理创建一个MultiPartRequestWrapper。

无论选择包装 MultiPartRequestWrapper 还是 StrutsRequestWrapper，都会返回包装后的 HttpServletRequest 对象，以便后续的请求处理可以使用这个包装后的对象来执行必要的操作。

在执行这个MultiPartRequestWrapper构造时

```java
ActionMapping mapping = prepare.findActionMapping(wrappedRequest, response, true);
execute.executeAction(wrappedRequest, response, mapping);
```

最后获取这个mapping，并且执行这个action

具体如下：

执行方法serviceAction(HttpServletRequest request, HttpServletResponse response, ActionMapping mapping)

调用actionMapper的getMapping方法对url进行解析，找出命名空间和action名等，以备后面根据配置文件调用相应的拦截器和action使用。

在action被调用之前，会首先走到fileUpload拦截器（对应的是FileUploadInterceptor类），在这个拦截器中，会先看一下request是不是 MultiPartRequestWrapper,如果不是，就说明不是上传文件用的request，fildUpload拦截器会直接将控制权交给下一个拦截器；如果是，就会把request对象强转为MultiPartRequestWrapper对象，然后调用hasErrors方法，看看有没有上传时候产生的错误，有的话，就直接加到了Action的错误（Action级别的）中了。

另外，在fileUpload拦截器中会将MultiPartRequestWrapper对象中放置的文件全取出来，把文件、文件名、文件类型取出来，放到request的parameters中，这样到了params拦截器时，就可以轻松的将这些内容注入到Action中了，这也就是为什么 fileUpload拦截器需要放在params拦截器前面的理由。在文件都放到request的parameters对象里之后，fileUpload 拦截器会继续调用其他拦截器直到Action等执行完毕，他还要做一个扫尾的工作：把临时文件夹中的文件删除（这些文件是由commons- fileupload组件上传的，供你在自己的Action中将文件copy到指定的目录下，当action执行完了后，这些临时文件当然就没用了）。









