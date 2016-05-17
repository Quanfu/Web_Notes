
#基于反相代理的Web缓存加速——可缓存的CMS系统设计

内容摘要：  
对于一个日访问量达到百万级的网站来说，速度很快就成为一个瓶颈。除了优化 内容发布系统的应用本身外，如果能把不需要实时更新的动态页面的输出结果转化成静态网页来发布，速度上的提升效果将是显著的，因为一个动态页面的速度往往 会比静态页面慢2－10倍，而静态网页的内容如果能被缓存 在内存里，访问[速度甚至会比原有动态网页有2－3个数量级的提高](http://www.chedong.com/tech/cache.html#test)。  

- [动态缓存和静态缓存的比较](http://www.chedong.com/tech/cache.html#compare)
- [基于反向代理加速的站点规划](http://www.chedong.com/tech/cache.html#site)  
- [基于apache mod_proxy的反向代理加速实现](http://www.chedong.com/tech/cache.html#apache)
- [基于squid的反向代理加速实现](http://www.chedong.com/tech/cache.html#squid)
- [面向缓存的页面设计](http://www.chedong.com/tech/cache.html#page)
- [应用的缓存兼容性设计](http://www.chedong.com/tech/cache.html#compatible)：  

HTTP_HOST/SERVER_NAME和REMOTE_ADDR/REMOTE_HOST需要用 HTTP_X_FORWARDED_HOST/HTTP_X_FORWARDED_SERVER代替
后台的内容管理系统的页面输出遵守可缓存的设计，这样就可以把性能问题交给前台的缓存服务器来解决了，从而大大简化CMS系统本身的复杂程度。  

### 静态缓存和动态缓存的比较

静态页面的缓存可能有2种形式：其实主要区别就是**CMS是否自己负责关联内容的缓存更新管理**。

1. 静态缓存：是在新内容发布的同时就立刻生成相应内容的静态页面，比如：2003年3月22日，管理员通过后台内容管理界面录入一 篇文章后，就立刻生成http://www.chedong.com/tech/2003/03/22/001.html这个静态页面，并同步更新相关索 引页上的链接。  
  

2. 动态缓存：是在新内容发布以后，并不预先生成相应的静态页面，直到对相应内容发出请求时，如果前台缓存服务器找不到相应缓存，就向后台内容管 理服务器发出请求，后台系统会生成相应内容的静态页面，用户第一次访问页面时可能会慢一点，但是以后就是直接访问缓存了。  
  
如果去ZDNet等国外网站会发现他们使用的基于[Vignette](http://www.vignette.com/)内容管理系统都有这样的页面名称：0,22342566,300458.html。其实这里 的0,22342566,300458就是用逗号分割开的多个参数：  
第一次访问找不到页面后，相当于会在服务器端产生一个doc_type= 0&doc_id=22342566&doc_template=300458的查询，  
而查询结果会生成的缓存的静态页面： 0,22342566,300458.html  

静态缓存的缺点：

- 复杂的触发更新机制：这两种机制在内容管理系统比较简单的时候都是非常适用的。但对于一个关系比较复杂的网站来说，页面之间的逻 辑引用关系就成为一个非常非常复杂的问题。最典型的例子就是一条新闻要同时出现在新闻首页和相关的3个新闻专题中，在静态缓存模式中，每发一篇新文章，除 了这篇新闻内容本身的页面外，还需要系统通过触发器生成多个新的相关静态页面，这些相关逻辑的触发也往往就会成为内容管理系统中最复杂的部分之一。

- 旧内容的批量更新： 通过静态缓存发布的内容，对于以前生成的静态页面的内容很难修改，这样用户访问旧页面时，新的模板根本无法生效。

在动态缓存模式中，每个动态页面只需要关心，而相关的其他页面能自动更新，从而大大减少了设计相关页面更新触发器的需要。  

以前做小型应用的时候也用过类似方式：应用首次访问以后将数据 库的查询结果在本地存成一个文件，下次请求时先检查本地缓存目录中是否有缓存文件，从而减少对后 台数据库的访问。虽然这样做也能承载比较大的负载，但这样的内容管理和缓存管理一体的系统是很难分离的，而且数据完整性也不是很好保存，内容更新时，应用需要把相应内容的的缓存文件删除。但是这样的设计在缓存文件很多的时候往往还需要将缓存目录做一定的分布，否则一个目录下的文件节点超过3000，rm *都会出错。  

这时候，系统需要再次分工，把复杂的内容管理系统分解成：内容输入和缓存这2个相对简单的系统实现。  

- 后台：内容管理系统，专心的将内容发布做好，比如：复杂的工作流管理，复杂的模板规则等……
- 前台：页面的缓存管理则可以使用缓存系统实现  

______________________             ___________________  
|Squid Software cache|             |F5 Hardware cache|  
----------------------             -------------------  
            \                    /  
             \ ________________ /  
               |ASP |JSP |PHP |  
             Content Manage System  
               ----------------

所以分工后：内容管理和缓存管理2者，无论哪一方面可选的余地都是非常大的：软件（比如前台80端口使用SQUID对后台8080的内容发布管理系 统进行缓存），缓存硬件，甚至交给[akamai](http://www.akamai.com/)这样的专业服务商。  

### 面向缓存的站点规划

一个利用SQUID对多个站点进行做WEB加速http acceleration方案：  
原先一个站点的规划可能是这样的：  

200.200.200.207 www.chedong.com   
200.200.200.208 news.chedong.com   
200.200.200.209 bbs.chedong.com   
200.200.200.205 images.chedong.com
  
面向缓存服务器的设计中：所有站点都通过外部DNS指向到同一个IP：200.200.200.200/201这2台缓存服务器上（**使用2台是为了 冗余备份**）

                          _____________________   ________  
www.chedong.com  请求  \ |       cache box     | |        |  / 192.168.0.4   www.chedong.com   
news.chedong.com 请求   -| 200.200.200.200/201 |-|firewall| -  192.168.0.4   news.chedong.com   
bbs.chedong.com  请求  / |   /etc/hosts        | |   box  |  \ 192.168.0.3   bbs.chedong.com  
                          ---------------------   --------  
工作原理：
  
外部请求过来时，设置缓存根据配置文件进行转向解析。这样，服务器请求就可以转 发到我们指定的内部地址上。  
在处理多虚拟主机转向方面：mod_proxy比squid要简单一些：可以把不同服务转向后后台多个IP的不同端口上。  
而squid只能通过禁用DNS解析，然后根据本地的/etc/hosts文件根据请求的域名进行地址转发，后台多个服务器必须使用相同的端口。  
使用反向代理加速，我们不仅可以得到性能上的提升，而且还能获得额外的安全性和配置的灵活度：  

- 配置灵活性提高：可以自己在内部服务器上控制后台服务器的DNS解析，当需要在服务器之间做迁移调整时，就不用大量修改外部DNS配置了，只 需要修改内部DNS实现服务的调整。
- 数据安全性增加：所有后台服务器可以很方便的被保护在防火墙内。
- 后台应用设计复杂程度降低：原先为了效率常常需要建立专门的图片服务器images.chedong.com和负载比较高的应用服务器 bbs.chedong.com分离，在反向代理加速模式中，所有前台请求都通过缓存服务器：实际上就都是静态页面，这样，应用设计时就不用考虑图片和应用本身分离了，也大大降低了后台内容发布系统设计的复杂程度，由于数据和应用都存放在一起，也方便了文件系统的维护和管理。  

### 基于Apache mod_proxy的反向代理缓存加速实现

Apache包含了mod_proxy模块，可以用来实现代理服务器，针对后台服务器的反向加速  
安装apache 1.3.x 编译时：
```  
--enable-shared=max --enable-module=most  
```
注：Apache 2.x中mod_proxy已经被分离成mod_proxy和mod_cache：同时mod_cache有基于文件和基于内存的不同实现  
创建/var/www/proxy，设置apache服务所用户可写  
mod_proxy配置样例：反相代理缓存＋缓存  
架设前台的www.example.com反向代理后台的www.backend.com的8080端口服务。  
修改：httpd.conf
```  
<VirtualHost *>  
ServerName www.example.com  
ServerAdmin admin@example.com  
# reverse proxy setting  
ProxyPass / http://www.backend.com:8080/  
ProxyPassReverse / http://www.backend.com:8080/  
# cache dir root  
CacheRoot "/var/www/proxy"  
# max cache storage  
CacheSize 50000000  
# hour: every 4 hour   
CacheGcInterval 4  
# max page expire time: hour  
CacheMaxExpire 240  
# Expire time = (now - last_modified) * CacheLastModifiedFactor   
CacheLastModifiedFactor 0.1  
# defalt expire tag: hour  
CacheDefaultExpire 1  
# force complete after precent of content retrived: 60-90%  
CacheForceCompletion 80  
CustomLog /usr/local/apache/logs/dev_access_log combined  
</VirtualHost>  
```
### 基于Squid的反向代理加速实现

Squid是一个更专用的代理服务器，性能和效率会比Apache的mod_proxy高很多。  
如果需要combined格式日志补丁：  
[http://www.squid-cache.org/mail-archive/squid-dev/200301/0164.html](http://www.squid-cache.org/mail-archive/squid-dev/200301/0164.html)  
squid的编译：
```  
./configure --enable-useragent-log  --enable-referer-log --enable-default-err-language=Simplify_Chinese \ --enable-err-languages="Simplify_Chinese English" --disable-internal-dns    
make  
#make install  
#cd /usr/local/squid  
make dir cache  
chown squid.squid *  
vi /usr/local/squid/etc/squid.conf  
在/etc/hosts中：加入内部的DNS解析，比如：  
192.168.0.4 www.chedong.com   
192.168.0.4 news.chedong.com  
192.168.0.3 bbs.chedong.com  
---------------------cut here----------------------------------  
# visible name  
visible_hostname cache.example.com  
# cache config: space use 1G and memory use 256M  
cache_dir ufs /usr/local/squid/cache 1024 16 256   
cache_mem 256 MB  
cache_effective_user squid  
cache_effective_group squid  
  
http_port 80  
httpd_accel_host virtual  
httpd_accel_single_host off  
httpd_accel_port 80  
httpd_accel_uses_host_header on  
httpd_accel_with_proxy on  
# accelerater my domain only  
acl acceleratedHostA dstdomain .example1.com  
acl acceleratedHostB dstdomain .example2.com  
acl acceleratedHostC dstdomain .example3.com  
# accelerater http protocol on port 80  
acl acceleratedProtocol protocol HTTP  
acl acceleratedPort port 80  
# access arc  
acl all src 0.0.0.0/0.0.0.0  
# Allow requests when they are to the accelerated machine AND to the  
# right port with right protocol  
http_access allow acceleratedProtocol acceleratedPort acceleratedHostA  
http_access allow acceleratedProtocol acceleratedPort acceleratedHostB  
http_access allow acceleratedProtocol acceleratedPort acceleratedHostC  
# logging  
emulate_httpd_log on  
cache_store_log none  
# manager  
acl manager proto cache_object  
http_access allow manager all  
cachemgr_passwd pass all  
```
----------------------cut here---------------------------------  
创建缓存目录：  
/usr/local/squid/sbin/squid -z  
启动squid  
/usr/local/squid/sbin/squid  
停止squid：  
/usr/local/squid/sbin/squid -k shutdown  
启用新配置：  
/usr/local/squid/sbin/squid -k reconfig  
通过crontab每天0点截断/轮循日志：  
0 0 * * * (/usr/local/squid/sbin/squid -k rotate)   

### 可缓存的动态页面设计

什么样的页面能够比较好的被缓存服务器缓存呢？如果返回内容的HTTP HEADER中有"Last-Modified"和"Expires"相关声明，比如：  
Last-Modified: Wed, 14 May 2003 13:06:17 GMT  
Expires: Fri, 16 Jun 2003 13:06:17 GMT  
前端缓存服务器在期间会将生成的页面缓存在本地：硬盘或者内存中，直至上述页面过期。  
因此，一个可缓存的页面：

- 页面必须包含Last-Modified: 标记  
一般纯静态页面本身都会有Last-Modified信息，动态页面需要通过函数强制加上，比如在PHP中：  
// always modified now  
header("Last-Modified: " . gmdate("D, d M Y H:i:s") . " GMT");  
  

- 必须有Expires或Cache-Control: max-age标记设置页面的过期时间：  
对于静态页面，通过apache的mod_expires根据页面的MIME类型设置缓存周期：比如图片缺省是1个月，HTML页面缺省是2天等。  
```
<IfModule mod_expires.c>   
    ExpiresActive on  
    ExpiresByType image/gif "access plus 1 month"  
    ExpiresByType text/css "now plus 2 day"  
    ExpiresDefault "now plus 1 day"  
</IfModule>  
```  
对于动态页面，则可以直接通过写入HTTP返回的头信息，比如对于新闻首页index.php可以是20分钟，而对于具体的一条新闻页面可能是1天后过 期。比如：在php中加入了1个月后过期： 
``` 
// Expires one month later  
header("Expires: " .gmdate ("D, d M Y H:i:s", time() + 3600 * 24 * 30). " GMT");  
```

- 如果服务器端有基于HTTP的认证，必须有Cache-Control: public标记，允许前台
ASP应用的缓存改造 首先在公用的包含文件中(比如include.asp)加入以下公用函数：  
```
<%  
' Set Expires Header in minutes  
Function SetExpiresHeader(ByVal minutes)   
    ' set Page Last-Modified Header:  
    ' Converts date (19991022 11:08:38) to http form (Fri, 22 Oct 1999 12:08:38 GMT)  
    Response.AddHeader "Last-Modified", DateToHTTPDate(Now())  
      
    ' The Page Expires in Minutes  
    Response.Expires = minutes  
      
    ' Set cache control to externel applications  
    Response.CacheControl = "public"  
End Function   
' Converts date (19991022 11:08:38) to http form (Fri, 22 Oct 1999 12:08:38 GMT)  
Function DateToHTTPDate(ByVal OleDATE)  
  Const GMTdiff = #08:00:00#  
  OleDATE = OleDATE - GMTdiff  
  DateToHTTPDate = engWeekDayName(OleDATE) & _  
    ", " & Right("0" & Day(OleDATE),2) & " " & engMonthName(OleDATE) & _  
    " " & Year(OleDATE) & " " & Right("0" & Hour(OleDATE),2) & _  
    ":" & Right("0" & Minute(OleDATE),2) & ":" & Right("0" & Second(OleDATE),2) & " GMT"  
End Function   
Function engWeekDayName(dt)  
    Dim Out  
    Select Case WeekDay(dt,1)  
        Case 1:Out="Sun"  
        Case 2:Out="Mon"  
        Case 3:Out="Tue"  
        Case 4:Out="Wed"  
        Case 5:Out="Thu"  
        Case 6:Out="Fri"  
        Case 7:Out="Sat"  
    End Select  
    engWeekDayName = Out  
End Function  
Function engMonthName(dt)  
    Dim Out  
    Select Case Month(dt)  
        Case 1:Out="Jan"  
        Case 2:Out="Feb"  
        Case 3:Out="Mar"  
        Case 4:Out="Apr"  
        Case 5:Out="May"  
        Case 6:Out="Jun"  
        Case 7:Out="Jul"  
        Case 8:Out="Aug"  
        Case 9:Out="Sep"  
        Case 10:Out="Oct"  
        Case 11:Out="Nov"  
        Case 12:Out="Dec"  
    End Select  
    engMonthName = Out  
End Function  
%>  
然后在具体的页面中，比如index.asp和news.asp的“最上面”加入以下代码：HTTP Header  
<!--#include file="../include.asp"-->  
<%  
'页面将被设置20分钟后过期  
SetExpiresHeader(20)  
%>  
```

### 应用的缓存兼容性设计

  
经过代理以后，由于在客户端和服务之间增加了中间层，因此服务器无法直接拿到客户端的IP，服务器端应用也无法直接通过转发请求的地址返回给客户端。但是 在转发请求的HTTD头信息中，增加了HTTP_X_FORWARDED_????信息。用以跟踪原有的客户端IP地址和原来客户端请求的服务器地址：  
下面是2个例子，用于说明缓存兼容性应用的设计原则：  
```
    '对于一个需要服务器名的地址的ASP应用：不要直接引用HTTP_HOST/SERVER_NAME，判断一下是否有HTTP_X_FORWARDED_SERVER  
    function getHostName ()  
        dim hostName as String = ""  
        hostName = Request.ServerVariables("HTTP_HOST")  
        if not isDBNull(Request.ServerVariables("HTTP_X_FORWARDED_HOST")) then  
            if len(trim(Request.ServerVariables("HTTP_X_FORWARDED_HOST"))) > 0 then  
                hostName = Request.ServerVariables("HTTP_X_FORWARDED_HOST")  
            end if  
        end if  
        return hostNmae  
    end function  
  
    //对于一个需要记录客户端IP的PHP应用：不要直接引用REMOTE_ADDR，而是要使用HTTP_X_FORWARDED_FOR，  
    function getUserIP (){  
        $user_ip = $_SERVER["REMOTE_ADDR"];  
        if ($_SERVER["HTTP_X_FORWARDED_FOR"]) {  
            $user_ip = $_SERVER["HTTP_X_FORWARDED_FOR"];  
        }  
    }   
```

注意：HTTP_X_FORWARDED_FOR如果经过了多个中间代理服务器，有何能是逗号分割的多个地址，  
比如：200.28.7.155,200.10.225.77 unknown,219.101.137.3  
因此在很多旧的数据库设计中（比如BBS）往往用来记录客户端地址的字段被设置成20个字节就显得过小了。  
经常见到类似以下的错误信息：  

Microsoft JET Database Engine 错误 '80040e57'

字段太小而不能接受所要添加的数据的数量。试着插入或粘贴较少的数据。

/inc/char.asp， 行236

原因就是在设计客户端访问地址时，相关用户IP字段大小最好要设计到50个字节以上，当然经过3层以上代理的几率也非常小。  
如何检查目前站点页面的可缓存性（Cacheablility）呢？可以参考以下2个站点上的工具：  
[http://www.ircache.net/cgi-bin/cacheability.py](http://www.ircache.net/cgi-bin/cacheability.py)  

### 附：SQUID性能测试试验

  
phpMan.php是一个基于php的man page server，每个man  
page需要调用后台的man命令和很多页面格式化工具，系统负载比较高，提供了Cache  
Friendly的URL，以下是针对同样的页面的性能测试资料：  
测试环境：Redhat 8 on Cyrix 266 / 192M Mem   
测试程序：使用apache的ab(apache benchmark)：  
测试条件：请求50次，并发50个连接  
测试项目：直接通过apache 1.3 (80端口) vs squid 2.5(8000端口：加速80端口)   
  
测试1：无CACHE的80端口动态输出：  
ab -n 100 -c 10 http://www.chedong.com:81/phpMan.php/man/kill/1  
This is ApacheBench, Version 1.3d <$Revision: 1.2 $> apache-1.3  
Copyright (c) 1996 Adam Twiss, Zeus Technology Ltd,  
http://www.zeustech.net/  
Copyright (c) 1998-2001 The Apache Group, http://www.apache.org/  
  
Benchmarking localhost (be patient).....done  
Server Software:         
Apache/1.3.23                                       
Server Hostname:        localhost  
Server  
Port:             
80  
  
Document Path:           
/phpMan.php/man/kill/1  
Document Length:        4655 bytes  
  
Concurrency Level:      5  
Time taken for tests:   63.164 seconds  
Complete requests:      50  
Failed requests:        0  
Broken pipe errors:     0  
Total transferred:      245900 bytes  
HTML transferred:       232750 bytes  
Requests per second:    0.79 [#/sec] (mean)  
Time per request:       6316.40 [ms]  
(mean)  
Time per request:       1263.28 [ms]  
(mean, across all concurrent requests)  
Transfer rate:           
3.89 [Kbytes/sec] received  
  
Connnection Times (ms)  
               
min  mean[+/-sd] median   max  
Connect:        0     
29  106.1      0   553  
Processing:  2942  6016  
1845.4   6227 10796  
  
Waiting:      
2941  5999 1850.7   6226 10795  
  
Total:        
2942  6045 1825.9   6227 10796  
  
Percentage of the requests served within a certain time (ms)  
  50%   6227  
  66%   7069  
  75%   7190  
  80%   7474  
  90%   8195  
  95%   8898  
  98%   9721  
  99%  10796  
 100%  10796 (last request)  
  
测试2：SQUID缓存输出  
/home/apache/bin/ab -n50 -c5  
"http://localhost:8000/phpMan.php/man/kill/1"  
This is ApacheBench, Version 1.3d <$Revision: 1.2 $> apache-1.3  
Copyright (c) 1996 Adam Twiss, Zeus Technology Ltd,  
http://www.zeustech.net/  
Copyright (c) 1998-2001 The Apache Group, http://www.apache.org/  
  
Benchmarking localhost (be patient).....done  
Server Software:         
Apache/1.3.23                                       
Server Hostname:        localhost  
Server  
Port:             
8000  
  
Document Path:           
/phpMan.php/man/kill/1  
Document Length:        4655 bytes  
  
Concurrency Level:      5  
Time taken for tests:   4.265 seconds  
Complete requests:      50  
Failed requests:        0  
Broken pipe errors:     0  
Total transferred:      248043 bytes  
HTML transferred:       232750 bytes  
Requests per second:    11.72 [#/sec] (mean)  
Time per request:       426.50 [ms] (mean)  
Time per request:       85.30 [ms] (mean,  
across all concurrent requests)  
Transfer rate:           
58.16 [Kbytes/sec] received  
  
Connnection Times (ms)  
               
min  mean[+/-sd] median   max  
Connect:         
0     1     
9.5      0    68  
Processing:      
7    83  537.4       
7  3808  
  
Waiting:         
5    81  529.1       
6  3748  
  
Total:           
7    84  547.0       
7  3876  
  
Percentage of the requests served within a certain time (ms)  
  50%      7  
  66%      7  
  75%      7  
  80%      7  
  90%      7  
  95%      7  
  98%      8  
  99%   3876  
 100%   3876 (last request)  
  
结论：No Cache / Cache = 6045 / 84 = 70  
结论：对于可能被缓存请求的页面，服务器速度可以有2个数量级的提高，因为SQUID是把缓存页面放在内存里的（因此几乎没有硬盘I/O操作）。  
  
小节：  

- 大访问量的网站应尽可能将动态网页生成静态页面作为缓存发布，甚至对于搜索引擎这样的动态应用来说，缓存机制也是非常非常重要的。  

- 在动态页面中利用HTTP Header定义缓存更新策略。  

- 利用缓存服务器获得额外的配置和安全性  

- 日志非常重要：SQUID日志缺省不支持COMBINED日志，但对于需要REFERER日志的这个补丁非常重要：[http://www.squid-cache.org/mail-archive/squid-dev/200301/0164.html](http://www.squid-cache.org/mail-archive/squid-dev/200301/0164.html)  

参考资料：  
[HTTP代理缓存](http://vancouver-webpages.com/proxy.html)   
http://vancouver-webpages.com/proxy.html  
[可缓存的页面设计](http://linux.oreillynet.com/pub/a/linux/2002/02/28/cachefriendly.html)  
http://linux.oreillynet.com/pub/a/linux/2002/02/28/cachefriendly.html  
[运用ASP.NET的输出缓冲来存储动态页面 -  开发者 - ZDNet China](http://www.zdnet.com.cn/developer/tech/story/0,2000081602,39110239-2,00.htm)  
http://www.zdnet.com.cn/developer/tech/story/0,2000081602,39110239-2,00.htm  
相关RFC文档：  

  

- [RFC  2616](http://www.w3.org/Protocols/rfc2616/rfc2616.html):  
  

    - [section  13](http://www.w3.org/Protocols/rfc2616/rfc2616-sec13.html#sec13) (Caching)  

    - [section  14.9](http://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.9) (Cache-Control header)  

    - [section  14.21](http://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.21) (Expires header)  

    - [section  14.32](http://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.32) (Pragma: no-cache) is important if you are interacting with  HTTP/1.0 caches  

    - [section  14.29](http://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.29) (Last-Modified) is the most common validation method  

    - [section  3.11](http://www.w3.org/Protocols/rfc2616/rfc2616-sec3.html#sec3.11) (Entity Tags) covers the extra validation method  

  
  

  
[可缓存性检查](http://www.web-caching.com/cacheability.html)  
http://www.web-caching.com/cacheability.html  
[缓存设计要素](http://vancouver-webpages.com/CacheNow/detail.html)  
http://vancouver-webpages.com/CacheNow/detail.html  
  
ZOPE上的几篇使用APACHE MOD_PROXY MOD_GZIP加速的文档  
[http://www.zope.org/Members/anser/apache_zserver/](http://www.zope.org/Members/anser/apache_zserver/)  
[http://www.zope.org/Members/softsign/ZServer_and_Apache_mod_gzip](http://www.zope.org/Members/softsign/ZServer_and_Apache_mod_gzip)  
[http://www.zope.org/Members/rbeer/caching](http://www.zope.org/Members/rbeer/caching) 作者：[车东](http://www.chedong.com/) 发表于：2003-06-06 17:06 最后更新于：2007-04-12 11:04  
[版权声明](http://creativecommons.org/licenses/by/3.0/deed.zh)：可以任意转载，转载时请务必以超链接形式标明文章[原始出处](http://www.chedong.com/tech/cache.html)和作者信息及[本声明](http://www.chedong.com/blog/archives/001249.html)。  
[http://www.chedong.com/tech/cache.html](http://www.chedong.com/tech/cache.html)