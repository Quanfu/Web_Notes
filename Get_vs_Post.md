## 引言

WebAPI 现在非常火的轻量服务框架，因其使用得使用了Http协议，并且具备了可协商内容（生成不同内容格式）等优势，所以在互联网业务中被广泛使用。

那作为HTTP最常用的两个方法Get和Post 是什么，他们之间又有哪些区别？这一问题经常成为面试的考点，这里就做一个简短总结，不保证全，但保证对。

 

## 什么是HTTP？

Hypertext Transfer Protocol (HTTP)是为服务端客户和客户端通信而设计的一种请求响应（Request-Response）协议。


HTTP中最常的用两种请求方式：

- **GET** - Requests data from a specified resource （从指定的资源获取数据，资源可以简单认为就是服务器）
- **POST** - Submits data to be processed to a specified resource （向指定的资源提及要处理的数据。）

 

## Get方法


**Note that the query string (name/value pairs) is sent in the URL of a GET request:**

**注意 查询字符串（键值对）是在Get请求的URL中携带的。**

/test/demo_form.asp?name1=value1&name2=value2

- GET requests can be cached （请求可以被缓存）
- GET requests remain in the browser history （请求会保留在浏览器的历史纪录）
- GET requests can be bookmarked （请求可以作为书签）
- GET requests should never be used when dealing with sensitive data （请求不应该用于处理敏感数据）
- GET requests have length restrictions （请求有相应的长度限制，虽然[有网友说可以没有限制](http://www.cnblogs.com/nankezhishi/archive/2012/06/09/getandpost.html)，但我还是同意协议制定方的约定）
- GET requests should be used only to retrieve data（请求应该仅用于获取数据）

 

补充，关于Get Request的请求长度

> The limit length of Get Request  is dependent on both the server and the client used (and if applicable, also the proxy the server or the client is using).

> Most webservers have a limit of 8192 bytes (8KB), which is usually configureable somewhere in the server configuration. As to the client side matter, the HTTP 1.1 specification even warns about this, here's an extract of [chapter 3.2.1](http://www.w3.org/Protocols/rfc2616/rfc2616-sec3.html#sec3.2.1):

> _Note: Servers ought to be cautious about depending on URI lengths above 255 bytes, because some older client or proxy implementations might not properly support these lengths._

 

## Post方法

**Note that the query string (name/value pairs) is sent in the HTTP message body of a POST request:**

**注意 查询字符串（键值对）是通过Post请求的HTTP消息体携带的。**

POST /test/demo_form.asp HTTP/1.1
Host: w3schools.com
name1=value1&name2=value2

- POST requests are never cached （请求数据永远不会被缓存）
- POST requests do not remain in the browser history （请求数据不会保存到浏览器的历史纪录中）
- POST requests cannot be bookmarked （请求不能做浏览器书签）
- POST requests have no restrictions on data length（请求没有严格的数据长度限制）

## Get VS. Post


|  |GET|POST|
|---|---|---|
|BACK button/Reload| Harmless（没有什么坏处，不过如果你没按约定做，Get做了更新或者删除，那就是自作孽） |Data will be re-submitted (the browser should alert the user that the data are about to be re-submitted)数据会被重新提交，浏览器应该对这种重新提交的数据向用户提出警告。
|Bookmarked| Can be bookmarked| Cannot be bookmarked|
|Cached| Can be cached | Not cached|
|Encoding type | application/x-www-form-urlencoded application/x-www-form-urlencoded or multipart/form-data.| Use multipart encoding for binary data|
|History| Parameters remain in browser history| Parameters are not saved in browser history|
|Restrictions on data length| Yes, when sending data, the GET method adds the data to the URL; and the length of a URL is limited (maximum URL length is 2048 characters) |No restrictions|
|Restrictions on data type |Only ASCII characters allowed（不允许中文，中文需自行编码处理） |No restrictions. Binary data is also allowed|
|Security| GET is less secure compared to POST because data sent is part of the URL  Never use GET when sending passwords or other sensitive information!| POST is a little safer than GET because the parameters are not stored in browser history or in web server logs|
|Visibility| Data is visible to everyone in the URL |Data is not displayed in the URL|

 

### 如何选择呢？

POST和GET的差别其实是很大的。

语义上，GET是获取指定URL上的资源，是读操作，重要的一点是不论对某个资源GET多少次，它的状态是不会改变的，在这个意义上，我们说GET是安全的（不是被密码学或者数据保护意义上的安全）。因为GET是安全的，所以GET返回的内容可以被浏览器，Cache服务器缓存起来（其中还有很多细节，但不影响这里的讨论）。

而POST的语意是对指定资源“追加/添加”数据，所以是不安全的，每次提交的POST，参与的代码都会认为这个操作会修改操作对象资源的状态，于是，浏览器在你按下F5的时候会跳出确认框，缓存服务器不会缓存POST请求返回内容。

安全的是指没有明显的对用户有影响的副作用(包括修改该资源的状态)。HTTP方法里的GET和HEAD都是安全的。

幂等的是指一个方法不论多少次操作，结果都是一样。PUT(把内容放到指定URL)，DELETE(删除某个URL代表的资源)，虽然都修改了资源内容，但多次操作，结果是相同的，因此和HEAD，GET一样都是幂等的。

所以根据HTTP协议，GET是安全的，也是幂等的，而POST既不是安全的，也不是幂等的。

 

按约定我们使用Get来做读操作，使用Post来新增或者修改（资源）。

 

## HTTP的其他请求方式

|Method |Description|
|---|---|
|HEAD |Same as GET but returns only HTTP headers and no document body|
|PUT |Uploads a representation of the specified URI|
| DELETE | Deletes the specified resource|
|OPTIONS |Returns the HTTP methods that the server supports|
|CONNECT |Converts the request connection to a transparent TCP/IP tunnel|

 

 

### 参考

[HTTP Methods: GET vs. POST](http://www.w3schools.com/tags/ref_httpmethods.asp)

[GET vs. POST](http://www.diffen.com/difference/GET_(HTTP)_vs_POST_(HTTP))

[maximum length of HTTP GET request?](https://stackoverflow.com/questions/2659952/maximum-length-of-http-get-request)