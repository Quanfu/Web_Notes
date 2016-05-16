#Cookie HTTP Only
一般的Cookie都是从document对象中获得的，现在浏览器在设置 Cookie的时候一般都接受一个叫做HttpOnly的参数，跟domain等其他参数一样，一旦这个HttpOnly被设置，你在浏览器的 document对象中就看不到Cookie了，而浏览器在浏览的时候不受任何影响，因为Cookie会被放在浏览器头中发送出去(包括ajax的时 候)，应用程序也一般不会在js里操作这些敏感Cookie的，对于一些敏感的Cookie我们采用HttpOnly，对于一些需要在应用程序中用js操作的cookie我们就不予设置，这样就保障了Cookie信息的安全也保证了应用。

##java
如果你正在使用的是兼容 Java EE 6.0 的容器，如 Tomcat 7，那么 Cookie 类已经有了 setHttpOnly 的方法来使用 HttpOnly 的 Cookie 属性了。
``` java
cookie.setHttpOnly(true);
```
设置完后生成的 Cookie 就会在最后多了一个 ;HttpOnly

另外使用 Session 的话 jsessionid 这个 Cookie 可通过在 Context 中使用 useHttpOnly 配置来启用 HttpOnly，例如：
```xml
<Context path="" 
docBase="D:/WORKDIR/oschina/webapp"
reloadable="false"
useHttpOnly="true"/>
```
也可以在 web.xml 配置如下：

```xml
<session-config>
<cookie-config>
<http-only>true</http-only>
</cookie-config>
<session-config>
```
##.net
对于 .NET 2.0 应用可以在 web.config 的 system.web/httpCookies 元素使用如下配置来启用 HttpOnly   

```xml
<httpCookies httpOnlyCookies="true"…>
```

而程序的处理方式如下：

C#:
```c#
HttpCookie myCookie = new HttpCookie("myCookie");

myCookie.HttpOnly =true;

Response.AppendCookie(myCookie);
```

VB.NET:

```vb
Dim myCookie As HttpCookie = new HttpCookie("myCookie")
myCookie.HttpOnly = True

Response.AppendCookie(myCookie)
```
.NET 1.1 只能手工处理：

``` c#
Response.Cookies[cookie].Path +=";HttpOnly";
```
##PHP 从 5.2.0 版本开始就支持 HttpOnly
``` php
session.cookie_httponly = True
```