
HTTP是一个无状态协议，如果想持续为用户服务（知道是那个用户在访问）就需要会话跟踪。会话（Session）跟踪是Web程序中常用的技术，用来跟踪用户的整个会话。常用的会话跟踪技术是`Cookie`与`Session`。

- Cookie通过在客户端记录信息确定用户身份，
- Session通过在服务器端记录信息确定用户身份。

##什么是Cookie

Cookie意为“甜饼”，是由W3C组织提出，最早由Netscape社区发展的一种机制。目前Cookie已经成为标准，所有的主流浏览器如IE、Netscape、Firefox、Opera等都支持Cookie。

由于**HTTP是一种无状态的协议**，服务器单从网络连接上无从知道客户身份。怎么办呢？就给客户端们颁发一个通行证吧，每人一个，无论谁访问都必须携带自己通行证。这样服务器就能从通行证上确认客户身份了。这就是Cookie的工作原理。

Cookie实际上是一小段的文本信息。客户端请求服务器，如果服务器需要记录该用户状态，就使用response向客户端浏览器颁发一个Cookie。客户端浏览器会把Cookie保存起来。当浏览器再请求该网站时，浏览器把请求的网址连同该Cookie一同提交给服务器。服务器检查该Cookie，以此来辨认用户状态。服务器还可以根据需要修改Cookie的内容。

![Cookie](/Pics/CookieSession/1459712229883_661590.jpg)

###会话跟踪
在程序中，**会话跟踪是很重要的事情**。

>理论上，一个用户的所有请求操作都应该属于同一个会话，而另一个用户的所有请求操作则应该属于另一个会话，二者不能混淆。例如，用户A在超市购买的任何商品都应该放在A的购物车内，不论是用户A什么时间购买的，这都是属于同一个会话的，不能放入用户B或用户C的购物车内，这不属于同一个会话。

>而Web应用程序是使用HTTP协议传输数据的。**HTTP协议是无状态的协议**。_一旦数据交换完毕，客户端与服务器端的连接就会关闭，再次交换数据需要建立新的连接_。这就意味着服务器无法从连接上跟踪会话。即用户A购买了一件商品放入购物车内，当再次购买商品时服务器已经无法判断该购买行为是属于用户A的会话还是用户B的会话了。要跟踪该会话，必须引入一种机制。

Cookie就是这样的一种机制。它可以弥补HTTP协议无状态的不足。在Session出现之前，基本上所有的网站都采用Cookie来跟踪会话。

##Cookie机制

**Cookie技术是客户端的解决方案**。

>通常，我们可以从很多网站的登录界面中看到“**请记住我**”这样的选项，如果你勾选了它之后再登录，那么在下一次访问该网站的时候就不需要进行重复而繁琐的登录动作了，而这个功能就是通过Cookie实现的。

###Cookie 工作机制
Cookie就是由服务器发给客户端的特殊信息，而这些信息以文本文件的方式存放在客户端，然后客户端每次向服务器发送请求的时候都会带上这些特殊的信息。

- 当用户使用浏览器访问一个支持Cookie的网站的时候，用户会提供包括用户名在内的个人信息并且提交至服务器；
- 接着，服务器在向客户端回传相应的超文本的同时也会发回这些个人信息，当然这些信息并不是存放在HTTP响应体（Response Body）中的，而是存放于HTTP响应头（Response Header）；
- 当客户端浏览器接收到来自服务器的响应之后，浏览器会将这些信息存放在一个统一的位置，对于`Windows`操作系统而言，我们可以从：`[系统盘]:\Documents and Settings\[用户名]\Cookies目录中找到存储的Cookie`；
- 自此，客户端再向服务器发送请求的时候，都会把相应的Cookie再次发回至服务器。而这次，Cookie信息则存放在HTTP请求头（Request Header）了。

有了Cookie这样的技术实现，服务器在接收到来自客户端浏览器的请求之后，就能够通过分析存放于请求头的Cookie得到客户端特有的信息，从而动态生成与该客户端相对应的内容。

###Cookie 生命周期和工作流程
如果你把`Cookies`看成为HTTP协议的一个扩展的话，理解起来就容易的多了，其实本质上`cookies`就是HTTP协议的一个扩展。有两个HTTP头部是专门负责设置以及发送`cookie`的,它们分别是:`Set-Cookie`以及`Cookie`。

当服务器返回给客户端一个HTTP响应信息时，其中如果包含`Set-Cookie`这个头部时，意思就是指示客户端建立一个cookie，并且在后续的HTTP请求中自动发送这个cookie到服务器端，直到这个cookie过期。

如果**cookie的生存时间是整个会话期间的话，那么浏览器会将cookie保存在内存中，浏览器关闭时就会自动清除这个cookie**。另外一种情况就是**保存在客户端的硬盘中，浏览器关闭的话，该cookie也不会被清除**，下次打开浏览器访问对应网站时，这个cookie就会自动再次发送到服务器端。一个cookie的设置以及发送过程分为以下四步：

1. 客户端发送一个HTTP请求到服务器端 
2. 服务器端发送一个HTTP响应到客户端，其中包含Set-Cookie头部
3. 客户端发送一个HTTP请求到服务器端，其中包含Cookie头部 
4. 服务器端发送一个HTTP响应到客户端

这个通讯过程也可以用以下下示意图来描述：
![Cookie流程](/Pics/CookieSession/1459712229883_577648.jpg)

在客户端的第二次请求中包含的Cookie头部中，提供给了服务器端可以用来唯一标识客户端身份的信息。这时，服务器端也就可以判断客户端是否启用了cookies。尽管，用户可能在和应用程序交互的过程中突然禁用cookies的使用，但是，这个情况基本是不太可能发生的，所以可以不考虑，这在实践中也被证明是对的。

除了cookies,客户端还可以将发送给服务器的数据包含在请求的url中，比如请求的参数或者请求的路径中。 我们来看一个常规的http get 请求例子：
```
GET /index.php?foo=bar HTTP/1.1 host: example.org
```
另外一种客户端传递数据到服务器端的方式是将数据包含在http请求的内容区域内。 这种方式需要请求的类型是POST的，看下面一个例子：
```
POST /index.php HTTP/1.1 Host: example.org Content-Type: application/x-www-form-urlencoded Content-Length: 7

foo=bar
```
在一个请求中，可以同时包含这两种形式的数据：
```
POST /index.php?myget=foo HTTP/1.1 Host: example.orgContent-Type: application/x-www-form-urlencoded Content-Length: 11

mypost=bar
```
这两种传递数据的方式，比起用cookies来传递数据更稳定，因为cookie可能被禁用，但是以GET以及POST方式传递数据时，不存在这种情况。我们可以将PHPSESSID包含在http请求的url中，就像下面的例子一样：
```
GET /index.php?PHPSESSID=12345 HTTP/1.1 Host: example.org
```

##如何查看Cookie

查看某个网站颁发的Cookie很简单。在Chrome浏览器 Console 栏输入`javascript:alert(document.cookie)`就可以了（需要有网才能查看）。JavaScript脚本会弹出一个对话框显示本网站颁发的所有Cookie的内容，如图所示。
![查看Cookie](/Pics/CookieSession/1459712229883_528036.jpg)

上图中弹出的对话框中显示的为Baidu网站的Cookie。其中BAIDUID记录的就是笔者的身份，只是Baidu使用特殊的方法将Cookie信息加密了。

>注意：**Cookie功能需要浏览器的支持。如果浏览器不支持Cookie（如大部分手机中的浏览器）或者把Cookie禁用了，Cookie功能就会失效**。不同的浏览器采用不同的方式保存Cookie。IE浏览器会在“C:\Documents and Settings\你的用户名\Cookies”文件夹下以文本文件形式保存，一个文本文件保存一个Cookie。

##Cookie 存储结构

Java中把Cookie封装成了javax.servlet.http.Cookie类。每个Cookie都是该Cookie类的对象。服务器通过操作Cookie类对象对客户端Cookie进行操作。通过request.getCookie获取客户端提交的所有Cookie（以Cookie数组形式返回），通过response.addCookie(Cookiecookie)向客户端设置Cookie。

Cookie对象使用key-value属性对的形式保存用户状态，一个Cookie对象保存一个属性对，一个request或者response同时使用多个Cookie。因为Cookie类位于包javax.servlet.http.*下面，所以JSP中不需要import该类。

![查看Cookie](/Pics/CookieSession/1459712229883_528037.jpg)

###设置Cookie的所有属性

除了name与value之外，Cookie还具有其他几个常用的属性。每个属性对应一个getter方法与一个setter方法。Cookie类的所有属性如下所示：

| 类型 | 属性名| 解释|
|---|---|---|
|String| name：该Cookie的名称。|**Cookie一旦创建，名称便不可更改**。|
|Object| value：该Cookie的值。|如果值为Unicode字符，需要为字符编码。如果值为二进制数据，则需要使用BASE64编码。|
|int| maxAge：该Cookie失效的时间，单位秒。|如果为正数，则该Cookie在maxAge秒之后失效。如果为负数，该Cookie为临时Cookie，关闭浏览器即失效，浏览器也不会以任何形式保存该Cookie。如果为0，表示删除该Cookie。默认为–1。|
|boolean| secure：该Cookie是否仅被使用安全协议传输。|安全协议。安全协议有HTTPS，SSL等，在网络上传输数据之前先将数据加密。默认为false。|
|String| path：该Cookie的使用路径。|如果设置为“/sessionWeb/”，则只有contextPath为“/sessionWeb”的程序可以访问该Cookie。如果设置为“/”，则本域名下contextPath都可以访问该Cookie。注意最后一个字符必须为“/”。|
|String| domain：可以访问该Cookie的域名。|如果设置为“.google.com”，则所有以“google.com”结尾的域名都可以访问该Cookie。注意第一个字符必须为“.”。|
|String| comment：该Cookie的用处说明。|浏览器显示Cookie信息的时候显示该说明。|
|int| version：该Cookie使用的版本号。|0表示遵循Netscape的Cookie规范，1表示遵循W3C的RFC 2109规范。|

###Unicode编码：保存中文

中文与英文字符不同，中文属于Unicode字符，在内存中占4个字符，而英文属于ASCII字符，内存中只占2个字节。Cookie中使用Unicode字符时需要对Unicode字符进行编码，否则会乱码。

>提示：Cookie中保存中文只能编码。一般使用UTF-8编码即可。**不推荐使用GBK等中文编码，因为浏览器不一定支持，而且JavaScript也不支持GBK编码**。

###BASE64编码：保存二进制图片

Cookie不仅可以使用ASCII字符与Unicode字符，还可以使用二进制数据。例如在Cookie中使用数字证书，提供安全度。使用二进制数据时也需要进行编码。

>注意：本程序仅用于展示Cookie中可以存储二进制内容，并不实用。由于浏览器每次请求服务器都会携带Cookie，因此**Cookie内容不宜过多，否则影响速度**。Cookie的内容应该少而精。

###Cookie的域名

Cookie是不可跨域名的。域名颁发的Cookie不会被提交到域名www.baidu.com去。这是由Cookie的隐私安全机制决定的。隐私安全机制能够禁止网站非法获取其他网站的Cookie。

####Cookie的不可跨域名性

很多网站都会使用Cookie。例如，google会向客户端颁发Cookie，Baidu也会向客户端颁发Cookie。那浏览器访问Google会不会也携带上Baidu颁发的Cookie呢？或者Google能不能修改Baidu颁发的Cookie呢？

答案是否定的。Cookie具有不可跨域名性。根据Cookie规范，浏览器访问Google只会携带Google的Cookie，而不会携带Baidu的Cookie。Google也只能操作Google的Cookie，而不能操作Baidu的Cookie。

Cookie在客户端是由浏览器来管理的。浏览器能够保证Google只会操作Google的Cookie而不会操作Baidu的Cookie，从而保证用户的隐私安全。浏览器判断一个网站是否能操作另一个网站Cookie的依据是域名。Google与Baidu的域名不一样，因此Google不能操作Baidu的Cookie。

**需要注意的是，虽然网站images.google.com与网站同属于Google，但是域名不一样，二者同样不能互相操作彼此的Cookie。**

>注意：用户登录网站之后会发现访问images.google.com时登录信息仍然有效，而普通的Cookie是做不到的。这是因为Google做了特殊处理。本章后面也会对Cookie做类似的处理。

#### 域名的设置

正常情况下，同一个一级域名下的两个二级域名如和images.helloweenvsfei.com也不能交互使用Cookie，因为二者的域名并不严格相同。如果想所有helloweenvsfei.com名下的二级域名都可以使用该Cookie，需要设置Cookie的domain参数，例如：

```
 Cookie cookie = new Cookie("time","20080808"); // 新建Cookie
 cookie.setDomain(".helloweenvsfei.com"); // 设置域名 
 cookie.setPath("/"); // 设置路径 
 cookie.setMaxAge(Integer.MAX_VALUE); // 设置有效期 
 response.addCookie(cookie); // 输出到客户端读者可以修改本机下的hosts文件来配置多个临时域名，然后使用setCookie.jsp程序来设置跨域名Cookie验证domain属性。
```
 
>注意：domain参数必须以点(".")开始。另外，name相同但domain不同的两个Cookie是两个不同的Cookie。如果想要两个域名完全不同的网站共有Cookie，可以生成两个Cookie，domain属性分别为两个域名，输出到客户端。



###Cookie的路径

domain属性决定运行访问Cookie的域名，而path属性决定允许访问Cookie的路径（ContextPath）。例如，如果只允许/sessionWeb/下的程序使用Cookie，可以这么写：

```
Cookie cookie = new Cookie("time","20080808"); // 新建Cookie 
cookie.setPath("/session/"); // 设置路径 
response.addCookie(cookie); // 输出到客户端
设置为“/”时允许所有路径使用Cookie。path属性需要使用符号“/”结尾。name相同但domain不同的两个Cookie也是两个不同的Cookie。
```

>注意：页面只能获取它属于的Path的Cookie。例如/session/test/a.jsp不能获取到路径为/session/abc/的Cookie。使用时一定要注意。

domain表示的是cookie所在的域，默认为请求的地址，如网址为www.test.com/test/test.aspx，那么domain默认为www.test.com。而跨域访问，如域A为t1.test.com，域B为t2.test.com，那么在域A生产一个令域A和域B都能访问的cookie就要将该cookie的domain设置为.test.com；如果要在域A生产一个令域A不能访问而域B能访问的cookie就要将该cookie的domain设置为t2.test.com。

path表示cookie所在的目录，默认为/，就是根目录。在同一个服务器上有目录如下：`/test/`,`/test/cd/`,`/test/dd/`，现设一个cookie1的path为`/test/`，cookie2的path为`/test/cd/`，那么test下的所有页面都可以访问到cookie1，而`/test/`和`/test/dd/`的子页面不能访问cookie2。这是因为cookie能让其path路径下的页面访问。

浏览器会将domain和path都相同的cookie保存在一个文件里，cookie间用*隔开。

###Cookie的安全属性

HTTP协议不仅是**无状态**的，而且是**不安全**的。使用HTTP协议的数据不经过任何加密就直接在网络上传播，有被截获的可能。使用HTTP协议传输很机密的内容是一种隐患。如果不希望Cookie在HTTP等非安全协议中传输，可以设置Cookie的secure属性为true。浏览器只会在HTTPS和SSL等安全协议中传输此类Cookie。下面的代码设置secure属性为true：

```
Cookie cookie = new Cookie("time", "20080808"); // 新建Cookie 
cookie.setSecure(true); // 设置安全属性 
response.addCookie(cookie); // 输出到客户端
```

提示：secure属性并不能对Cookie内容加密，因而不能保证绝对的安全性。如果需要高安全性，需要在程序中对Cookie内容加密、解密，以防泄密。

###Cookie的有效期

Cookie的maxAge决定着Cookie的有效期，单位为秒（Second）。Cookie中通过getMaxAge方法与setMaxAge(int maxAge)方法来读写maxAge属性。如果maxAge属性为正数，则表示该Cookie会在maxAge秒之后自动失效。浏览器会将maxAge为正数的Cookie持久化，即写到对应的Cookie文件中。无论客户关闭了浏览器还是电脑，只要还在maxAge秒之前，登录网站时该Cookie仍然有效。下面代码中的Cookie信息将永远有效。

```
Cookie cookie = new Cookie("username","helloweenvsfei"); // 新建Cookie 
cookie.setMaxAge(Integer.MAX_VALUE); // 设置生命周期为MAX_VALUE 
response.addCookie(cookie); // 输出到客户端

```

- 如果maxAge为负数，则表示该Cookie仅在本浏览器窗口以及本窗口打开的子窗口内有效，关闭窗口后该Cookie即失效。maxAge为负数的Cookie，为临时性Cookie，不会被持久化，不会被写到Cookie文件中。Cookie信息保存在浏览器内存中，因此关闭浏览器该Cookie就消失了。Cookie默认的maxAge值为–1。

- 如果maxAge为0，则表示删除该Cookie。Cookie机制没有提供删除Cookie的方法，因此通过设置该Cookie即时失效实现删除Cookie的效果。失效的Cookie会被浏览器从Cookie文件或者内存中删除：

```
Cookie cookie = new Cookie("username","helloweenvsfei"); // 新建Cookie 
cookie.setMaxAge(0); // 设置生命周期为0，不能为负数 
response.addCookie(cookie); // 必须执行这一句
```

response对象提供的Cookie操作方法只有一个添加操作add(Cookie cookie)。要想修改Cookie只能使用一个同名的Cookie来覆盖原来的Cookie，达到修改的目的。删除时只需要把maxAge修改为0即可。

>注意：从客户端读取Cookie时，包括maxAge在内的其他属性都是不可读的，也不会被提交。浏览器提交Cookie时只会提交name与value属性。maxAge属性只被浏览器用来判断Cookie是否过期。

***

##Cookie 操作

###Cookie的修改、删除

Cookie并**不提供修改、删除操作**。如果要修改某个Cookie，只需要新建一个同名的Cookie，添加到response中**覆盖**原来的Cookie。如果要删除某个Cookie，只需要新建一个同名的Cookie，并将maxAge设置为0，并添加到response中覆盖原来的Cookie。注意是0而不是负数。负数代表其他的意义。读者可以通过上例的程序进行验证，设置不同的属性。

>注意：修改、删除Cookie时，新建的Cookie除value、maxAge之外的所有属性，例如name、path、domain等，都要与原Cookie完全一样。否则，浏览器将视为两个不同的Cookie不予覆盖，导致修改、删除失败。

###JavaScript操作Cookie

Cookie是保存在浏览器端的，因此**浏览器具有操作Cookie的先决条件**。浏览器可以使用脚本程序如JavaScript或者VBScript等操作Cookie。这里以JavaScript为例介绍常用的Cookie操作。例如下面的代码会输出本页面所有的Cookie。

```
<script>document.write(document.cookie);</script> 
```

由于JavaScript能够任意地读写Cookie，有些好事者便想使用JavaScript程序去窥探用户在其他网站的Cookie。不过这是徒劳的，W3C组织早就意识到JavaScript对Cookie的读写所带来的安全隐患并加以防备了，**W3C标准的浏览器会阻止JavaScript读写任何不属于自己网站的Cookie**。换句话说，A网站的JavaScript程序读写B网站的Cookie不会有任何结果。















##Session机制

**Session机制是一种服务器端的机制**，服务器使用一种类似于散列表的结构（也可能就是使用散列表）来保存信息。

session机制采用的是一种在服务器端保持状态的解决方案。同时我们也看到，由于采用服务器端保持状态的方案在客户端也需要保存一个标识，所以session机制可能需要借助于cookie机制来达到保存标识的目的。而session提供了方便管理全局变量的方式 。

session是针对每一个用户的，变量的值保存在服务器上，用一个sessionID来区分是哪个用户session变量,这个值是通过用户的浏览器在访问的时候返回给服务器，当客户禁用cookie时，这个值也可能设置为由get来返回给服务器。

就安全性来说：当你访问一个使用session 的站点，同时在自己机子上建立一个cookie，建议在服务器端的session机制更安全些，因为它不会任意读取客户存储的信息。


当程序需要为某个客户端的请求创建一个session时，服务器首先检查这个客户端的请求里是否已包含了一个session标识（称为session id），如果已包含则说明以前已经为此客户端创建过session，服务器就按照session id把这个session检索出来使用（检索不到，会新建一个），如果客户端请求不包含session id，则为此客户端创建一个session并且生成一个与此session相关联的session id，session id的值应该是一个既不会重复，又不容易被找到规律以仿造的字符串，这个session id将被在本次响应中返回给客户端保存。

保存这个session id的方式可以采用cookie，这样在交互过程中浏览器可以自动的按照规则把这个标识发挥给服务器。一般这个cookie的名字都是类似于SEEESIONID。但cookie可以被人为的禁止，则必须有其他机制以便在cookie被禁止时仍然能够把session id传递回服务器。

经常被使用的一种技术叫做URL重写，就是把session id直接附加在URL路径的后面。还有一种技术叫做表单隐藏字段。就是服务器会自动修改表单，添加一个隐藏字段，以便在表单提交时能够把session id传递回服务器。

  * * * *
  
Cookie与Session都能够进行会话跟踪，但是完成的原理不太一样。普通状况下二者均能够满足需求，但有时分不能够运用Cookie，有时分不能够运用Session。下面经过比拟阐明二者的特性以及适用的场所。

 1、存取方式的不同

Cookie中只能保管ASCII字符串，假如需求存取Unicode字符或者二进制数据，需求先进行编码。Cookie中也不能直接存取Java对象。若要存储略微复杂的信息，运用Cookie是比拟艰难的。

而Session中能够存取任何类型的数据，包括而不限于String、Integer、List、Map等。Session中也能够直接保管Java Bean乃至任何Java类，对象等，运用起来十分便当。能够把Session看做是一个Java容器类。

2、隐私策略的不同

Cookie存储在客户端阅读器中，对客户端是可见的，客户端的一些程序可能会窥探、复制以至修正Cookie中的内容。而Session存储在服务器上，对客户端是透明的，不存在敏感信息泄露的风险。

假如选用Cookie，比较好的方法是，敏感的信息如账号密码等尽量不要写到Cookie中。最好是像Google、Baidu那样将Cookie信息加密，提交到服务器后再进行解密，保证Cookie中的信息只要本人能读得懂。而假如选择Session就省事多了，反正是放在服务器上，Session里任何隐私都能够有效的保护。

 3、有效期上的不同

使用过Google的人都晓得，假如登录过Google，则Google的登录信息长期有效。用户不用每次访问都重新登录，Google会持久地记载该用户的登录信息。要到达这种效果，运用Cookie会是比较好的选择。只需要设置Cookie的过期时间属性为一个很大很大的数字。

由于Session依赖于名为JSESSIONID的Cookie，而Cookie JSESSIONID的过期时间默许为–1，只需关闭了阅读器该Session就会失效，因而Session不能完成信息永世有效的效果。运用URL地址重写也不能完成。而且假如设置Session的超时时间过长，服务器累计的Session就会越多，越容易招致内存溢出。

 4、服务器压力的不同

Session是保管在服务器端的，每个用户都会产生一个Session。假如并发访问的用户十分多，会产生十分多的Session，耗费大量的内存。因而像Google、Baidu、Sina这样并发访问量极高的网站，是不太可能运用Session来追踪客户会话的。

而Cookie保管在客户端，不占用服务器资源。假如并发阅读的用户十分多，Cookie是很好的选择。关于Google、Baidu、Sina来说，Cookie或许是唯一的选择。

 5、浏览器支持的不同

Cookie是需要客户端浏览器支持的。假如客户端禁用了Cookie，或者不支持Cookie，则会话跟踪会失效。关于WAP上的应用，常规的Cookie就派不上用场了。

假如客户端浏览器不支持Cookie，需要运用Session以及URL地址重写。需要注意的是一切的用到Session程序的URL都要进行URL地址重写，否则Session会话跟踪还会失效。关于WAP应用来说，Session+URL地址重写或许是它唯一的选择。

假如客户端支持Cookie，则Cookie既能够设为本浏览器窗口以及子窗口内有效（把过期时间设为–1），也能够设为一切阅读器窗口内有效（把过期时间设为某个大于0的整数）。但Session只能在本阅读器窗口以及其子窗口内有效。假如两个浏览器窗口互不相干，它们将运用两个不同的Session。（IE8下不同窗口Session相干）

6、跨域支持上的不同

Cookie支持跨域名访问，例如将domain属性设置为“.biaodianfu.com”，则以“.biaodianfu.com”为后缀的一切域名均能够访问该Cookie。跨域名Cookie如今被普遍用在网络中，例如Google、Baidu、Sina等。而Session则不会支持跨域名访问。Session仅在他所在的域名内有效。

仅运用Cookie或者仅运用Session可能完成不了理想的效果。这时应该尝试一下同时运用Cookie与Session。Cookie与Session的搭配运用在实践项目中会完成很多意想不到的效果。