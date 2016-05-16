#Cookie HTTP Only

一般的Cookie都是从document对象中获得的，现在浏览器在设置 Cookie的时候一般都接受一个叫做HttpOnly的参数，跟domain等其他参数一样，一旦这个HttpOnly被设置，你在浏览器的 document对象中就看不到Cookie了，而浏览器在浏览的时候不受任何影响，因为Cookie会被放在浏览器头中发送出去(包括ajax的时 候)，应用程序也一般不会在js里操作这些敏感Cookie的，对于一些敏感的Cookie我们采用HttpOnly，对于一些需要在应用程序中用js操作的cookie我们就不予设置，这样就保障了Cookie信息的安全也保证了应用。


Session cookies (或者包含JSSESSIONID的cookie)是指用来管理web应用的session会话的cookies.这些cookie中保存特定使用者的session ID标识，而且相同的session ID以及session生命周期内相关的数据也在服务器端保存。在web应用中最常用的session管理方式是通过每次请求的时候将cookies传送到服务器端来进行session识别。

HttpOnly标识是一个可选的、避免利用XSS（Cross-Site Scripting)来获取session cookie的标识。XSS攻击最常见一个的目标是通过获取的session cookie来劫持受害者的session；使用HttpOnly标识是一种很有用的保护机制。

解决方案很简单。只需要在session cookie上添加HttpOnly标识就行了（最好是所有的cookie）。

下面是一个没有添加HttpOnly标识的session cookie：

Cookie: jsessionid=AS348AF929FK219CKA9FK3B79870H;

添加HttpOnlyFlag标识之后：
```
Cookie: jsessionid=AS348AF929FK219CKA9FK3B79870H; HttpOnly;
```

如果你看过上周的文章，添加了secure和HttpOnly标识的cookie：
```
Cookie: jsessionid=AS348AF929FK219CKA9FK3B79870H; HttpOnly; secure;
```

很简单。你可以人工设置这些参数，如果你在Servlet3或者更新的环境中开发，只需要在web.xml简单的配置就能实现这种效果。你要在web.xml中添加如下片段：
```
<session-config>

  <cookie-config>

    <http-only>true</http-only>

  </cookie-config>

</session-config>
```

而且如果使用了secure标识，配置应该如下
```
<session-config>

  <cookie-config>

    <http-only>true</http-only>

    <secure>true</secure>

  </cookie-config>

</session-config>
```
如上所述，解决这个问题很简单。每个人都应该解决这个问题。

###References
http://blog.mozilla.com/webappsec/2011/03/31/enabling-browser-security-in-web-applications/
http://michael-coates.blogspot.com/2011/03/enabling-browser-security-in-web.html
https://www.owasp.org/index.php/HttpOnly

##java
如果你正在使用的是兼容 Java EE 6.0 的容器，如 Tomcat 7，那么 Cookie 类已经有了 `setHttpOnly` 的方法来使用` HttpOnly` 的 Cookie 属性了。
``` java
cookie.setHttpOnly(true);
```
设置完后生成的 Cookie 就会在最后多了一个 `;HttpOnly`

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