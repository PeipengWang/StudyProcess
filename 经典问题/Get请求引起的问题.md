get指令最大长度为1304的字符，如果请求参数过多，会导致URL长度受到限制，导致指令发送失败。
解决方法：改成post方式发

Http get方法提交的数据大小长度并没有限制，Http协议规范没有对URL长度进行限制。
       目前说的get长度有限制，是特定的浏览器及服务器对它的限制。
        各种浏览器和服务器的最大处理能力如下：
        IE：对URL的最大限制为2083个字符，若超出这个数字，提交按钮没有任何反应。
        Firefox：对Firefox浏览器URL的长度限制为：65536个字符。
        Safari：URL最大长度限制为80000个字符。
        Opera：URL最大长度限制为190000个字符。
        Google(chrome)：URL最大长度限制为8182个字符。
        Apache(Server)：能接受的最大url长度为8192个字符（这个准确度待定？？？）
        Microsoft Internet Information Server(IIS)：n能接受最大url的长度为16384个字符。
