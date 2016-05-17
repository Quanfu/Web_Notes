原文（英文）地址： [http://www.mnot.net/cache_docs/](http://www.mnot.net/cache_docs/)  版权声明：[署名-非商业性使用-禁止演绎 2.0](http://creativecommons.org/licenses/by-nc-nd/2.0/deed.zh)  

这是一篇知识性的文档，主要目的是为了让Web缓存相关概念更容易被开发者理解并应用于实际的应用环境中。为了简要起见，某些实现方面的细节被简化或省略了。如果你更关心细节实现则完全不必耐心看完本文，后面参考文档和更多深入阅读部分可能是你更需要的内容。  

1. 什么是Web缓存，为什么要使用它？
2. 缓存的类型：
    1. 浏览器缓存；
    2. 代理服务器缓存；

3. Web缓存无害吗？为什么要鼓励缓存？
4. Web缓存如何工作：
5. 如何控制（控制不）缓存：
    1. HTML Meta标签 vs. HTTP头信息；
    2. Pragma HTTP头信息（为什么不起作用）；
    3. 使用Expires（过期时间）HTTP头信息控制保鲜期；
    4. Cache-Control（缓存控制） HTTP头信息；  

    5. 校验参数和校验；

6. 创建利于缓存网站的窍门；
7. 编写利于缓存的脚本；
8. 常见问题解答；
9. 缓存机制的实现：Web服务器端配置；
10. 缓存机制的实现：服务器端脚本；
11. 参考文档和深入阅读；
12. 关于本文档；

#### 什么是Web缓存，为什么要使用它？

Web缓存位于Web服务器之间（1个或多个，内容源服务器）和客户端之间（1个或多个）：缓存会根据进来的请求保存输出内容的副本，例如html页面， 图片，文件（统称为副本），然后，当下一个请求来到的时候：如果是相同的URL，缓存直接使用副本响应访问请求，而不是向源服务器再次发送请求。  
  
使用缓存主要有2大理由：  

- **减少相应延迟**：因为请求从缓存服务器（离客户端更近）而不是源服务器被相应，这个过程耗时更少，让web服务器看上去相应更快；
- **减少网络带宽消耗**：当副本被重用时会减低客户端的带宽消耗；客户可以节省带宽费用，控制带宽的需求的增长并更易于管理。

#### 缓存的类型

##### 浏览器缓存

对于新一代的Web浏览器来说（例如：IE，Firefox）：一般都能在设置对话框中发现关于缓存的设置，通过在你的电脑上僻处一块硬盘空间用于存储你已经看过的网站的副本。浏览器缓存根据非常简单的规则进行工作：在同一个会话过程中（在当前浏览器没有被关闭之前）会检查一次并确定缓存的副本足够新。这个缓存对于用户点击“后退”或者点击刚访问过的链接特别有用，如果你浏览过程中访问到同一个图片，这些图片可以从浏览器缓存中调出而即时显现。  

##### 代理服务器缓存

Web代理服务器使用同样的缓存原理，只是规模更大。代理服务器群为成百上千用户服务使用同样的机制；大公司和ISP经常在他们的防火墙上架设代理缓存或者单独的缓存设备；

由于带路服务器缓存并非客户端或者源服务器的一部分，而是位于原网络之外，请求必须路由到他们才能起作用。一个方法是手工设置你的浏览器：告诉浏览器使用 那个代理，另外一个是通过中间服务器：这个中间服务器处理所有的web请求，并将请求转发到后台网络，而用户不必配置代理，甚至不必知道代理的存在；  

代理服务器缓存：是一个共享缓存，不只为一个用户服务，经常为大量用户使用，因此在减少相应时间和带宽使用方面很有效：因为同一个副本会被重用多次。  

##### 网关缓存

也被称为反向代理缓存或间接代理缓存，网关缓存也是一个中间服务器，和内网管理员部署缓存用于节省带宽不同：网关缓存一般是网站管理员自己部署：让他们的网站更容易扩展并获得更好的性能；  
请求有几种方法被路由到网关缓存服务器上：其中典型的是让用一台或多台负载均衡服务器从客户端看上去是源服务器；  
  
网络内容发布商  (Content delivery networks CDNs)分布网关缓存到整个（或部分）互联网上，并出售缓存服务给需要的网站，[Speedera](http://www.speedera.com/)和[Akamai](http://www.akamai.com/)就是典型的网络内容发布商(下文简称CDN)。  
  
本问主要关注于浏览器和代理缓存，当然，有些信息对于网关缓存也同样有效；  

#### Web缓存无害吗？为什么要鼓励缓存？

Web缓存在互联网上最容易被误解的技术之一：网站管理员经常怕对网站失去控制，由于代理缓存会“隐藏”他们的用户，让他们感觉难以监控谁在使用他们的网站。  
不幸的是：就算不考虑Web缓存，互联网上也有很多网站使用非常多的参数以便管理员精确地跟踪用户如何使用他们的网站；如果这类问题也是你关心的，本文将告诉你如何获得精确的统计而不必将网站设计的非常缓存不友好。  
另外一个抱怨是缓存会给用户过期或失效的数据；无论如何：本文可以告诉你怎样配置你的服务器来控制你的内容将被如何缓存。  
  
CDN是另外一个有趣的方向，和其他代理缓存不同：CDN的网关缓存为希望被缓存的网站服务，没有以上顾虑。即使你使用了CDN，你也要考虑后续的代理服务器缓存和浏览器缓存问题。

另外一方面：如果良好地规划了你的网站，缓存会有助于网站服务更快，并节省服务器负载和互联网的链接请求。这个改善是显著的：一个难以缓存的网站可能需要几秒去载入页面，而对比有缓存的网站页面几乎是即时显现：用户更喜欢速度快的网站并更经常的访问；

这样想：很多大型互联网公司为全世界服务器群投入上百万资金，为的就是让用户访问尽可能快，客户端缓存也是这个目的，只不过更靠近用户一端，而且最好的一点是你甚至根本不用为此付费。

事实上，无论你是否喜欢，代理服务器和浏览器都回启用缓存。如果你没有配置网站正确的缓存，他们会按照缺省或者缓存管理员的策略进行缓存。  
  

#### 缓存如何工作

所有的缓存都用一套规则来帮助他们决定什么时候使用缓存中的副本提供服务（假设有副本可用的情况下）；一些规则在协议中有定义（HTTP协议1.0和1.1），一些规则由缓存的管理员设置（浏览器的用户或者代理服务器的管理员）；  
一般说来：遵循以下基本的规则（不必担心，你不必知道所有的细节，细节将随后说明）  

1. 如果响应头信息：告诉缓存器不要保留缓存，缓存器就不会缓存相应内容；
2. 如果请求信息是需要认证或者安全加密的，相应内容也不会被缓存；
3. 如果在回应中不存在校验器（ETag或者Last-Modified头信息），缓存服务器会认为缺乏直接的更新度信息，内容将会被认为不可缓存。
4. 一个缓存的副本如果含有以下信息：内容将会被认为是足够新的
    - 含有完整的过期时间和寿命控制头信息，并且内容仍在保鲜期内；
    - 浏览器已经使用过缓存副本，并且在一个会话中已经检查过内容的新鲜度；
    - 缓存代理服务器近期内已经使用过缓存副本，并且内容的最后更新时间在上次使用期之前；
    - 够新的副本将直接从缓存中送出，而不会向源服务器发送请求；

5. 如果缓存的副本已经太旧了，缓存服务器将向源服务器发出请求校验请求，用于确定是否可以继续使用当前拷贝继续服务；
总之：**_新鲜度_**和**校验**是确定内容是否可用的最重要途径：

 

如果副本足够新，从缓存中提取就立刻能用了；  
而经缓存器校验后发现副本的原件没有变化，系统也会避免将副本内容从源服务器整个重新传输一遍。  
  

#### 如何控制（控制不）缓存

有很多工具可以帮助设计师和网站管理员调整缓存服务器对待网站的方式，这也许需要你亲自下手对服务器的配置进行一些调整，但绝对值得；了解如何使用这些工具请参考后面的实现章节；  

##### HTML meta标签和HTTP 头信息

HTML的编写者会在文档的<HEAD>区域中加入描述文档的各种属性，这些META标签常常被用于标记文档不可以被缓存或者标记多长时间后过期；  
META标签使用很简单：但是效率并不高，因为只有几种浏览器会遵循这个标记（那些真正会“读懂”HTML的浏览器），没有一种缓存代理服务器能遵循这个 规则（因为它们几乎完全不解析文档中HTML内容）；有事会在Web页面中增加：Pragma: no-cache这个META标记，如果要让页面保持刷新，这个标签其实完全没有必要。  
如果你的网站托管在ISP机房中，并且机房可能不给你权限去控制HTTP的头信息（如：Expires和Cache-Control），大声控诉：这些机制对于你的工作来说是必须的；  
另外一方面： HTTP头信息可以让你对浏览器和代理服务器如何处理你的副本进行更多的控制。他们在HTML代码中是看不见的，一般由Web服务器自动生成。但是，根据 你使用的服务，你可以在某种程度上进行控制。在下文中：你将看到一些有趣的HTTP头信息，和如何在你的站点上应用部署这些特性。  
  
HTTP头信息发送在HTML代码之前，只有被浏览器和一些中间缓存能看到，一个典型的HTTP 1.1协议返回的头信息看上去像这样：  

HTTP/1.1 200 OK  
Date: Fri, 30 Oct 1998 13:19:41 GMT  
Server: Apache/1.3.3 (Unix)  
Cache-Control: max-age=3600, must-revalidate  
Expires: Fri, 30 Oct 1998 14:19:41 GMT  
Last-Modified: Mon, 29 Jun 1998 02:28:12 GMT  
ETag: "3e86-410-3596fbbc"  
Content-Length: 1040  
Content-Type: text/html  

  
在头信息空一行后是HTML代码的输出，关于如何设置HTTP头信息请参考实现章节；  

##### Pragma HTTP头信息 (为什么它不起作用)

很多人认为在HTTP头信息中设置了Pragma: no-cache后会让内容无法被缓存。但事实并非如此：HTTP的规范中，响应型头信息没有任何关于Pragma属性的说明，而讨论了的是请求型头信息 Pragma属性（头信息也由浏览器发送给服务器），虽然少数集中缓存服务器会遵循这个头信息，但大部分不会。用了Pragma也不起什么作用，要用就使 用下列头信息：  

##### 使用Expires（过期时间）HTTP头信息来控制保鲜期

Expires（过期时间） 属性是HTTP控制缓存的基本手段，这个属性告诉缓存器：相关副本在多长时间内是新鲜的。过了这个时间，缓存器就会向源服务器发送请求，检查文档是否被修改。几乎所有的缓存服务器都支持Expires（过期时间）属性；  
  
大部分Web服务器支持你用几种方式设置Expires属性；一般的：可以设计一个绝对时间间隔：基于客户最后查看副本的时间（最后访问时间）或者根据服务器上文档最后被修改的时间；

Expires头信息：对于设置静态图片文件（例如导航栏和图片按钮）可缓存特别有用；因为这些图片修改很少，你可以给它们设置一个特别长的过期时间，这会使你的网站对用户变得相应非常快；他们对于控制有规律改变的网页也很有用，例如：你每天早上6点更新新闻页，你可以设置副本的过期时间也是这个时间，这样缓存 服务器就知道什么时候去取一个更新版本，而不必让用户去按浏览器的“刷新”按钮。

过期时间头信息属性值**只能**是HTTP格式的日期时间，其他的都会被解析成当前时间“之前”，副本会过期，记住：HTTP的日期时间必须是格林威治时间（GMT），而不是本地时间。举例：  

Expires: Fri, 30 Oct 1998 14:19:41 GMT  

所以使用过期时间属性一定要确认你的Web服务器时间设置正确，一个途径是通过网络时间同步协议（Network Time Protocol NTP），和你的系统管理员那里你可以了解更多细节。  
虽然过期时间属性非常有用，但是它还是有些局限，首先：是牵扯到了日期，这样Web服务器的时间和缓存服务器的时间必须是同步的，如果有些不同步，要么是应该缓存的内容提前过期了，要么是过期结果没及时更新。  
还有一个过期时间设置的问题也不容忽视：如果你设置的过期时间是一个固定的时间，如果你返回内容的时候又没有连带更新下次过期的时间，那么之后所有访问请求都会被发送给源Web服务器，反而增加了负载和响应时间；  

##### Cache-Control（缓存控制） HTTP头信息

HTTP 1.1介绍了另外一组头信息属性：Cache-Control响应头信息，让网站的发布者可以更全面的控制他们的内容，并定位过期时间的限制。  
有用的 Cache-Control响应头信息包括：  

- **max-age**=[秒] — 执行缓存被认为是最新的最长时间。类似于过期时间，这个参数是基于请求时间的相对时间间隔，而不是绝对过期时间，[秒]是一个数字，单位是秒：从请求时间开始到过期时间之间的秒数。
- **s-maxage**=[秒] — 类似于max-age属性，除了他应用于共享（如：代理服务器）缓存
- **public **— 标记认证内容也可以被缓存，一般来说： 经过HTTP认证才能访问的内容，输出是自动不可以缓存的；
- **no-cache** — 强制每次请求直接发送给源服务器，而不经过本地缓存版本的校验。这对于需要确认认证应用很有用（可以和public结合使用），或者严格要求使用最新数据的应用（不惜牺牲使用缓存的所有好处）；
- **no-store** — 强制缓存在任何情况下都不要保留任何副本
- **must-revalidate** — 告诉缓存必须遵循所有你给予副本的新鲜度的，HTTP允许缓存在某些特定情况下返回过期数据，指定了这个属性，你高速缓存，你希望严格的遵循你的规则。
- **proxy-revalidate** — 和 must-revalidate类似，除了他只对缓存代理服务器起作用

举例:  

Cache-Control: max-age=3600, must-revalidate  

如果你计划试用Cache-Control属性，你应该看一下这篇HTTP文档，详见参考和深入阅读；  
  

##### 校验参数和校验

在Web缓存如何工作： 我们说过：校验是当副本已经修改后，服务器和缓存之间的通讯机制；使用这个机制：缓存服务器可以避免副本实际上仍然足够新的情况下重复下载整个原件。  
校验参数非常重要，如果1个不存在，并且没有任何信息说明保鲜期（Expires或Cache-Control）的情况下，缓存将不会存储任何副本；  
最常见的校验参数是文档的最后修改时间，通过最后Last-Modified头信息可以，当一份缓存包含Last-Modified信息，他基于此信息，通过添加一个If-Modified-Since请求参数，向服务器查询：这个副本从上次查看后是否被修改了。  
HTTP 1.1介绍了另外一个校验参数： ETag，服务器是服务器生成的唯一标识符ETag，每次副本的标签都会变化。由于服务器控制了ETag如何生成，缓存服务器可以通过If-None-Match请求的返回没变则当前副本和原件完全一致。  
所有的缓存服务器都使用Last-Modified时间来确定副本是否够新，而ETag校验正变得越来越流行；  
所有新一代的Web服务器都对静态内容（如：文件）自动生成ETag和Last-Modified头信息，而你不必做任何设置。但是，服务器对于动态内容（例如：CGI,ASP或数据库生成的网站）并不知道如何生成这些信息，参考一下编写利于缓存的脚本章节；  
  

#### 创建利于缓存网站的窍门

除了使用新鲜度信息和校验，你还有很多方法使你的网站缓存友好。  

- **保持URL稳定**： 这是缓存的金科玉律，如果你给在不同的页面上，给不同用户或者从不同的站点上提供相同的内容，应该使用相同的URL，这是使你的网站缓存友好最简单，也是 最高效的方法。例如：如果你在页面上使用 "/index.html" 做为引用，那么就一直用这个地址；
- **使用一个共用的库**存放每页都引用的图片和其他页面元素；
- **对于不经常改变的图片/页面启用缓存**，并使用Cache-Control: max-age属性设置一个较长的过期时间；
- **对于定期更新的内容**设置一个缓存服务器可识别的max-age属性或过期时间；
- **如果数据源（特别是下载文件）变更，修改名称**，这样：你可以让其很长时间不过期，并且保证服务的是正确的版本；而链接到下载文件的页面是一个需要设置较短过期时间的页面。
- **万不得已不要改变文件**，否则你会提供一个非常新的Last-Modified日期；例如：当你更新了网站，不要复制整个网站的所有文件，只上传你修改的文件。
- **只在必要的时候使用Cookie**，cookie是非常难被缓存的，而且在大多数情况下是不必要的，如果使用cookie，控制在动态网页上；
- **减少试用SSL**，加密的页面不会被任何共享缓存服务器缓存，只在必要的时候使用，并且在SSL页面上减少图片的使用；
- **使用可缓存性评估引擎**，这对于你实践本文的很多概念都很有帮助；

#### 编写利于缓存的脚本

脚本缺省不会返回校验参数（返回Last-Modified或ETag头信息）或其他新鲜度信息（Expires或Cache-Control），有些动态脚本的确是动态内容（每次相应内容都不一样），但是更多（搜索引擎，数据库引擎网站）网站还是能从缓存友好中获益的。  
一般说来，如果脚本生成的输出在未来一段时间（几分钟或者几天）都是可重复复制的，那么就是可缓存的。如果脚本输出内容只随URL变化而变化，也是可缓存的；但如果输出会根据cookie，认证信息或者其他外部条件变化，则还是不可缓存的。  

- 最利于缓存的脚本就是将内容改变时导出成静态文件，Web服务器可以将其当作另外一个网页并生成和试用校验参数，让一些都变得更简单，只需要写入文件即可，这样最后修改时间也有了；
- 另外一个让脚本可缓存的方法是对一段时间内能保持较新的内容设置一个相对寿命的头信息，虽然通过Expires头信息也可以实现，但更容易的是用Cache-Control: max-age属性，它会让首次请求后一段时间内缓存保持新鲜；
- 如果以上做法你都做不到，你可以让脚本生成一个校验属性，并对 If-Modified-Since 和/或If-None-Match请求作出反应，这些属性可以从解析HTTP头信息得到，并对符合条件的内容返回304 Not Modified（内容未改变），可惜的是，这种做法比不上前2种高效；

其他窍门：

- 尽量避免使用POST，除非万不得已，POST模式的返回内容不会被大部分缓存服务器保存，如果你发送内容通过URL和查询（通过GET模式）的内容可以缓存下来供以后使用；
- 不要在URL中加入针对每个用户的识别信息：除非内容是针对每个用户不同的；
- 不要统计一个用户来自一个地址的所有请求，因为缓存常常是一起工作的；
- 生成并返回Content-Length头信息，如果方便的话，这个属性让你的脚本在可持续链接模式时：客户端可以通过一个TCP/IP链接同时请求多个副本，而不是为每次请求单独建立链接，这样你的网站相应会快很多；
具体定义请参考实现章节。

#### 常见问题解答

##### 让网站变得可缓存的要点是什么？

好的策略是确定那些内容最热门，大量的复制（特别是图片）并针对这些内容先部署缓存。  

##### 如何让页面通过缓存达到最快相应？

缓存最好的副本是那些可以长时间保持新鲜的内容；基于校验虽然有助于加快相应，但是它不得不和源服务器联系一次去检查内容是否够新，如果缓存服务器上就知道内容是新的，内容就可以直接相应返回了。  

##### 我理解缓存是好的，但是我不得不统计多少人访问了我的网站！

如果你必须知道每次页面访问的，选择【一】个页面上的小元素，或者页面本身，通过适当的头信息让其不可缓存，例如： 可以在每个页面上部署一个1x1像素的透明图片。Referer头信息会有包含这个图片的每个页面信息；  
明确一点：这个并不会给你一个关于你用户精确度很高的统计，而且这对互联网和你的用户这都不太好，消耗了额外的带宽，强迫用户去访问无法缓存的内容。了解更多信息，参考访问统计资料。  

##### 我如何能看到HTTP头信息的内容？

很多浏览器在页面属性或类似界面中可以让你看到Expires 和Last-Modified信息；如果有的话：你会找到页面信息的菜单和页面相关的文件（如图片），并且包含他们的详细信息；  
看到完整的头信息，你可以用telnet手工连接到Web服务器；  
为此：你可能需要用一个字段指定端口（缺省是80），或者链接到www.example.com:80 或者 www.example.com 80(注意是空格)，更多设置请参考一下telnet客户端的文档；  
打开网站链接：请求一个查看链接，如果你想看到http://www.example.com/foo.html 连接到www.example.com的80端口后，键入：  

GET /foo.html HTTP/1.1 [回车]  
GET /foo.html HTTP/1.1 [return]  
Host: www.example.com [回车][回车]   
Host: www.example.com [return][return]  

在[回车]处按键盘的回车键；在最后，要按2次回车，然后，就会输出头信息及完整页面，如果只想看头信息，将GET换成HEAD。  
  

##### 我的页面是密码保护的，代理缓存服务器如何处理他们？

缺省的，网页被HTTP认证保护的都是私密内容，它们不会被任何共享缓存保留。但是，你可以通过设置Cache-Control: public让认证页面可缓存，HTTP 1.1标准兼容的缓存服务器会认出它们可缓存。  
如果你认为这些可缓存的页面，但是需要每个用户认证后才能看，可以组合使用Cache-Control: public和no-cache头信息，高速缓存必须在提供副本之前，将将新客户的认证信息提交给源服务器。设置就是这样：  

_Cache-Control: public, no-cache_  

无论如何：这是减少认证请求的最好方法，例如： 你的图片是不机密的，将它们部署在另外一个目录，并对此配置服务器不强制认证。这样，那些图片会缺省都缓存。  

##### 我们是否要担心用户通过cache访问我的站点？

代理服务器上SSL页面不会被缓存（不推荐被缓存），所以你不必为此担心。但是，由于缓存保存了非SSL请求和从他们抓取的URL，你要意识到没有安全保护的网站，可能被不道德的管理员可能搜集用户隐私，特别是通过URL。  
实际上，位于服务器和客户端之间的管理员可以搜集这类信息。特别是通过CGI脚本在通过URL传递用户名和密码的时候会有很大问题；这对泄露用户名和密码是一个很大的漏洞；  
如果你初步懂得互联网的安全机制，你不会对缓存服务器有任何。  

##### 我在寻找一个包含在Web发布系统解决方案，那些是比较有缓存意识的系统？

这很难说，一般说来系统越复杂越难缓存。最差就是全动态发布并不提供校验参数；你无发缓存任何内容。可以向系统提供商的技术人员了解一下，并参考后面的实现说明。  

##### 我的图片设置了1个月后过期，但是我现在需要现在更新。

过期时间是绕不过去的，除非缓存（浏览器或者代理服务器）空间不足才会删除副本，缓存副本在过期之间会被一直使用。  
最好的办法是改变它们的链接，这样，新的副本将会从源服务器上重新下载。记住：引用它们的页面本身也会被缓存。因此，使用静态图片和类似内容是很容易缓存的，而引用他们的HTML页面则要保持非常更新；  
如果你希望对指定的缓存服务器重新载入一个副本，你可以强制使用“刷新”（在FireFox中在reload的时候按住shift键：就会有前面提到恶Pragma: no-cache头信息发出）。或者你可以让缓存的管理员从他们的界面中删除相应内容；  

##### 我运行一个Web托管服务，如何让我的用户发布缓存友好的网页？

如果你使用apahe，可以考虑允许他们使用.htaccess文件并提供相应的文档；  
另外一方面： 你也可以考虑在各种虚拟主机上建立各种缓存策略。例如： 你可以设置一个目录 /cache-1m 专门用于存放访问1个月的访问，另外一个 /no-cache目录则被用提供不可存储副本的服务。  
无论如何：对于大量用户访问还是应该用缓存。对于大网站，这方面的节约很明显（带宽和服务器负载）；  

##### 我标记了一些网页是可缓存的，但是浏览器仍然每次发送请求给服务。如何强制他们保存副本？

缓存服务器并不会总保存副本并重用副本；他们只是在特定情况下会不保存并使用副本。所有的缓存服务器都回基于文件的大小，类型（例如：图片 页面），或者服务器空间的剩余来确定如何缓存。你的页面相比更热门或者更大的文件相比，并不值得缓存。  
所以有些缓存服务器允许管理员根据文件类型确定缓存副本的优先级，允许某些副本被永久缓存并长期有效；  

#### 缓存机制的实现 - Web服务器端配置

一般说来，应该选择最新版本的Web服务器程序来部署。不仅因为它们包含更多利于缓存的功能，新版本往往在性能和安全性方面都有很多的改善。  

##### Apache HTTP服务器

Apache有些可选的模块来包含这些头信息： 包括Expires和Cache-Control。 这些模块在1.2版本以上都支持；  
这些模块需要和apache一起编译；虽然他们已经包含在发布版本中，但缺省并没有启用。为了确定相应模块已经被启用：找到httpd程序并运行httpd -l 它会列出可用的模块，我们需要用的模块是mod_expires和mod_headers  

- 如果这些模块不可用，你需要联系管理员，重新编译并包含这些模块。这些模块有时候通过配置文件中把注释掉的配置启用，或者在编译的时候增加-enable -module=expires和-enable-module=headers选项（在apache 1.3和以上版本）。 参考Apache发布版中的INSTALL文件；

Apache一旦启用了相应的模块，你就可以在.htaccess文件或者在服务器的access.conf文件中通过mod_expires设置副本什 么时候过期。你可设置过期从访问时间或文件修改时间开始计算，并且应用到某种文件类型上或缺省设置，参考[模块的文档](http://httpd.apache.org/docs/1.3/mod/mod_expires.html)获得更多信息，或者遇到问题的时候向你身边的apache专家讨教。  
应用Cache-Control头信息，你需要使用mod_headers,它将允许你设置任意的HTTP头信息，参考[mod_headers的文档](http://httpd.apache.org/docs/1.3/mod/mod_headers.html)可以获得更多资料；  
这里有个例子说明如何使用头信息：  

- .htaccess文件允许web发布者使用命令只在配置文件中用到的命令。他影响到所在目录及其子目录；问一下你的服务器管理员确认这个功能是否启用了。   

### 启用 mod_expires  
ExpiresActive On  
### 设置 .gif 在被访问过后1个月过期。  
ExpiresByType image/gif A2592000  
### 其他文件设置为最后修改时间1天后过期  
### (用了另外的语法)  
ExpiresDefault "modification plus 1 day"  
### 在index.html文件应用 Cache-Control头属性  
<Files index.html>  
Header append Cache-Control "public, must-revalidate"  
</Files>          

- 注意： 在适当情况下mod_expires会自动计算并插入Cache-Control:max-age 头信息

Apache 2.0的配置和1.3类似，更多信息可以参考2.0的[mod_expires](http://httpd.apache.org/docs/2.2/mod/mod_expires.html)和[mod_headers文档](http://httpd.apache.org/docs/2.2/mod/mod_headers.html)；  
  

##### Microsoft IIS服务器

Microsoft的IIS可以非常容易的设置头信息，注意：这只针对IIS 4.0服务器，并且只能在NT服务器上运行。  
为网站的一个区域设置头信息，先要到管理员工具界面中，然后设置属性。选择HTTP Header选单，你会看到2个有趣的区域：启用内容过期和定制HTTP头信息。头一个设置会自动配置，第二个可以用于设置Cache-Control头信息；  
设置asp页面的头信息可以参考后面的ASP章节，也可以通过ISAPI模块设置头信息，细节请参考MSDN。  
  

##### Netscape/iPlanet企业服务器

3.6版本以后，Netscape/iPlanet已经不能设置Expires头信息了，他从3.0版本开始支持HTTP 1.1的功能。这意味着HTTP 1.1的缓存（代理服务器/浏览器）优势都可以通过你对Cache-Control设置来获得。  
使用Cache-Control头信息，在管理服务器上选择内容管理|缓存设置目录。然后：使用资源选择器，选择你希望设置头信息的目录。设置完头信息后，点击“OK”。更多信息请参考[Netscape/iPlanet企业服务器的手册](http://developer.netscape.com/docs/manuals/enterprise/admnunix/content.htm#1006282)。  
  

##### 缓存机制的实现：服务器端脚本

需要注意的一点是：也许服务器设置HTTP头信息比脚本语言更容易，但是两者你都应该使用。  
因为服务器端的脚本主要是为了动态内容，他本身不产生可缓存的文件页面，即使内容实际是可以缓存的。如果你的内容经常改变，但是不是每次页面请求都改变， 考虑设置一个Cache-Control: max-age头信息；大部分用户会在短时间内多次访问同一页面。例如： 用户点击“后退”按钮，即使没有新内容，他们仍然要再次从服务器下载内容查看。  
  

##### CGI程序

CGI脚本是生成内容最流行的方式之一，你可以很容易在发送内容之前的扩展HTTP头信息；大部分CGI实现都需要你写 Content-Type头信息，例如这个Perl脚本：  

#!/usr/bin/perl  
print "Content-type: text/html\n";  
print "Expires: Thu, 29 Oct 1998 17:04:19 GMT\n";  
print "\n";  
### 后面是内容体...  

由于都是文本，你可以很容易通过内置函数生成Expires和其他日期相关的头信息。如果你使用Cache-Control: max-age;会更简单；  

print "Cache-Control: max-age=600\n";  

这样脚本可以在被请求后缓存10分钟；这样用户如果按“后退”按钮，他们不会重新提交请求；  
CGI的规范同时也允许客户端发送头信息，每个头信息都有一个‘HTTP_’的前缀；这样如果一个客户端发送一个If-Modified-Since请求，就是这样的：  

HTTP_IF_MODIFIED_SINCE = Fri, 30 Oct 1998 14:19:41 GMT   

  
参考一下[cgi_buffer](http://www.mnot.net/cgi_buffer/)库，一个自动处理ETag的生成和校验的库，生成Content-Length属性和对内容进行gzip压缩。在Python脚本中也只需加入一行；  
  

##### 服务器端包含 Server Side Includes

SSI（经常使用.shtml扩展名）是网站发布者最早可以生成动态内容的方案。通过在页面中设置特别的标记，也成为一种嵌入HTML的脚本；  
大部分SSI的实现无法设置校验器，于是无法缓存。但是Apache可以通过对特定文件的组执行权限设置实现允许用户设置那种SSI可以被缓存；结合XbitHack调整整个目录。更多文档请参考[mod_include文档](http://httpd.apache.org/docs/1.3/mod/mod_include.html)。  

##### PHP

PHP是一个内建在web服务器中的服务器端脚本语言，当做为HTML嵌入式脚本，很像SSI，但是有更多的选项，PHP可以在各种Web服务器上设置为CGI模式运行，或者做为Apache的模块；  
缺省PHP生成副本没有设置校验器，于是也无法缓存，但是开发者可以通过Header()函数来生成HTTP的头信息；  
例如：以下代码会生成一个Cache-Control头信息，并设置为3天以后过期的Expires头信息；  

<?php  
 Header("Cache-Control: must-revalidate");  
  
 $offset = 60 * 60 * 24 * 3;  
 $ExpStr = "Expires: " . gmdate("D, d M Y H:i:s", time() + $offset) . " GMT";  
 Header($ExpStr);  
?>  

记住： Header()的输出必须先于所有其他HTML的输出；  
正如你看到的：你可以手工创建HTTP日期；PHP没有为你提供专门的函数（新版本已经让这个越来越容易了，请参考PHP的[日期相关函数文档](http://php.net/date)），当然，最简单的还是设置Cache-Control: max-age头信息，而且对于大部分情况都比较适用；  
更多信息，请参考[header相关的文档](http://www.php.net/manual/function.header.php3)；  
也请参考一下[cgi_buffer](http://www.mnot.net/cgi_buffer/)库，自动处理ETag的生成和校验，Content-Length生成和内容的gzip压缩，PHP脚本只需包含1行代码；  

##### Cold Fusion

[Cold Fusion](http://www.adobe.com/products/coldfusion/)是Macromedia的商业服务器端脚本引擎，并且支持多种Windows平台，Linux平台和多种Unix平台。Cold Fusion通过CFHEADER标记设置HTTP头信息相对容易。可惜的是：以下的Expires头信息的设置有些容易误导；  

<CFHEADER NAME="Expires" VALUE="#Now()#">  

它并不像你想像的那样工作，因为时间（本例中为请求发起的时间）并不会被转换成一个符合HTTP时间，而且打印出副本的Cold fusion的日期/时间对象，大部分客户端会忽略或者将其转换成1970年1月1日。  
但是：Cold Fusion另外提供了一套日期格式化函数， GetHttpTimeSTring. 结合DateAdd函数，就很容易设置过期时间了，这里我们设置一个Header声明副本在1个月以后过期；  

<cfheader name="Expires" value="#GetHttpTimeString(DateAdd('m', 1, Now()))#">  

你也可以使用CFHEADER标签来设置Cache-Control: max-age等其他头信息；  
记住：Web服务器也会将头信息设置转给Cold Fusion(做为CGI运行的时候)，检查你的服务器设置并确定你是否可以利用服务器设置代替Cold Fusion。   
  

##### ASP和ASP.NET

在asp中设置HTTP头信息是：确认Response方法先于HTML内容输出前被调用，或者使用 Response.Buffer暂存输出；同样的：注意某些版本的IIS缺省设置会输出Cache-Control: private 头信息，必须声明成public才能被共享缓存服务器缓存。  
IIS的ASP和其他web服务器都允许你设置HTTP头信息，例如： 设置过期时间，你可以设置Response对象的属性；  

<% Response.Expires=1440 %>  

设置请求的副本在输出的指定分钟后过期，类似的：也可以设置绝对的过期时间（确认你的HTTP日期格式正确）  

<% Response.ExpiresAbsolute=#May 31,1996 13:30:15 GMT# %>  

Cache-Control头信息可以这样设置：  

<% Response.CacheControl="public" %>  

在ASP.NET中，Response.Expires 已经不推荐使用了，正确的方法是通过Response.Cache设置Cache相关的头信息；  

Response.Cache.SetExpires ( DateTime.Now.AddMinutes ( 60 ) ) ;  
Response.Cache.SetCacheability ( HttpCacheability.Public ) ;  

参考[MSDN文档](http://msdn.microsoft.com/library/default.asp?url=/library/en-us/cpguide/html/cpconaspoutputcache.asp)可以找到更多相关新年系；  
  

#### 参考文档和深入阅读


##### [HTTP 1.1 规范定义](http://www.ietf.org/rfc/rfc2616.txt)

HTTP 1.1的规范有大量的扩展用于页面缓存，以及权威的接口实现指南，参考章节：13, 14.9, 14.21, 以及 14.25.  
  
- [RFC  2616](http://www.w3.org/Protocols/rfc2616/rfc2616.html):   
    - [section13](http://www.w3.org/Protocols/rfc2616/rfc2616-sec13.html#sec13) (Caching)  
    - [section14.9](http://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.9) (Cache-Control header)  
    - [section14.21](http://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.21) (Expires header)  
    - [section14.32](http://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.32) (Pragma: no-cache) is important if you are interacting with  HTTP/1.0 caches  
    - [section14.29](http://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.29) (Last-Modified) is the most common validation method  
    - [section3.11](http://www.w3.org/Protocols/rfc2616/rfc2616-sec3.html#sec3.11) (Entity Tags) covers the extra validation method  

##### [Web-Caching.com](http://www.web-caching.com/)

非常精彩的介绍缓存相关概念，并介绍其他在线资源。  
[  
](http://www.goldmark.org/netrants/webstats/)

##### [关于非连续性访问统计](http://www.goldmark.org/netrants/webstats/)

Jeff Goldberg内容丰富的演说告诉你为什么不应该过度依赖访问统计和计数器；  
  

##### [可缓存性检测引擎](http://www.mnot.net/cacheability/)

可缓存的引擎设计，检测网页并确定其如何与Web缓存服务器交互， 这个引擎配合这篇指南是一个很好的调试工具，  
  

##### [cgi_buffer库](http://www.mnot.net/cgi_buffer/)

包含库：用于CGI模式运行的Perl/Python/PHP脚本，自动处理ETag生成/校验，Content-Length生成和内容压缩。正确地。 Python版本也被用作其他大量的CGI脚本。  
  

#### 关于本文档

本文版权属于Mark Nottingham <[mnot@pobox.com](mailto:mnot@pobox.com)>，本作品遵循[创作共用版权](http://creativecommons.org/licenses/by-nc-nd/2.0/deed.zh)。  
如果你镜像本文，请通过以上邮件告知，这样你可以在更新时被通知；  
所有的商标属于其所有人。  
虽然作者确信内容在发布时的正确性，但不保证其应用或引申应用的正确性，如有误传，错误或其他需要澄清的问题请尽快告知作者；  
本文最新版本可以从 [http://www.mnot.net/cache_docs/](http://www.mnot.net/cache_docs/) 获得；  
翻译版本包括： [捷克语版](http://www.jakpsatweb.cz/clanky/caching-tutorial-czech-translation.html)，[法语版](http://www.mnot.net/cache_docs/index.fr.html)和[中文版](http://www.chedong.com/tech/cache_docs.html)。  
版本： 1.81 - 2007年3月16日  
[创作共用版权声明](http://creativecommons.org/licenses/by-nc-nd/2.0/deed.zh)  
翻译： [车东](http://www.chedong.com/tech/) 2007年9月6日

作者：[车东](http://www.chedong.com/) 发表于：2007-09-06 00:09 最后更新于：2007-11-07 15:11  
[版权声明](http://creativecommons.org/licenses/by/3.0/deed.zh)：可以任意转载，转载时请务必以超链接形式标明文章[原始出处](http://www.chedong.com/tech/cache_docs.html)和作者信息及[本声明](http://www.chedong.com/blog/archives/001249.html)。  
[http://www.chedong.com/tech/cache_docs.html](http://www.chedong.com/tech/cache_docs.html)

***

##Cache-Control Detail
Any valid HTTP headers can be put in these files. This provides another way to apply the Expires header, and it's a way to add the `Cache-Control` headers. The relevant `Cache-Control` headers are:

`Cache-Control` : `max-age = [delta-seconds]`
Modifies the expiration mechanism, overriding the Expires header. Max-age implies Cache-Control : public.
>`Cache-control: max-age=5` //表示当访问此网页后的5秒内再次访问不会去服务器 

`Cache-Control` : `public`  
Indicates that the object may be stored in a cache. This is the default.

`Cache-Control` : `private`   
`Cache-Control` : `private = [field-name]`   
Indicates that the object (or specified field) must not be stored in a shared cache and is intended for a single user. It may be stored in a private cache.

`Cache-Control : no-cache`  
`Cache-Control : no-cache = [field-name]`   
Indicates that the object (or specified field) may be cached, but may not be served to a client unless revalidated with the origin server.
>`Cache-Control: no-cache`：这个很容易让人产生误解，使人误以为是响应不被缓存。实际上`Cache-Control: no-cache`是会被缓存的，只不过每次在向客户端（浏览器）提供响应数据时，缓存都要向服务器评估缓存响应的有效性。

`Cache-Control : no-store`   
Indicates that the item must not be stored in nonvolatile storage, and should be removed as soon as possible from volatile storage.
>`Cache-Control: no-store`：这个才是响应不被缓存的意思。

`Cache-Control : no-transform`   
Proxies may convert data from one storage system to another. This directive indicates that (most of) the response must not be transformed. (The RFC allows for transformation of some fields, even with this header present.)
>`Cache-Control : no-transform` 不允许代理对响应结果进行变换

`Cache-Control : must-revalidate`   
`Cache-Control : proxy-revalidate`   
**Forces** the proxy to revalidate the page even if the client will accept a stale response. **Read the RFC before using these headers, there are restrictions on their use.**



**Caveats and gotchas(警告和陷阱)**

>`HTTP/1.0` has minimal cache control and only understands the `Pragma: no-cache` header. Caches using `HTTP/1.0` will **ignore** the `Expires` and `Cache-Control` headers.
>Pragma: no-cache：跟Cache-Control: no-cache相同，Pragma: no-cache兼容http 1.0 ，Cache-Control: no-cache是http 1.1提供的

>None of the Cache-Control directives ensure privacy or security of data. The directives "private" and "no-store" assist in privacy and security, but they are not intended to substitute for authentication and encryption.

This article is not a substitute for the RFC. If your are implementing the `Cache-Control` headers, **do** read the RFC for a detailed description of what each header means and what the limits are.

**Final words**

Caching is a reality of the Internet and enables efficient usage of bandwidth. Your clients probably view your pages through a cache, and sometimes multiple caches. Applying cache headers to your pages protects the page content and allows your clients to save their bandwidth.