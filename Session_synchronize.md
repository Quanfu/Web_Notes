#Session同步方案可以采用tomcat+memcach session复制方案（基于反向代理轮询机制）

部署步骤：
##1. 安装memcache
##2. 停止tomcat
##3. 在tomcat/lib目录放置下面的包
spymemcached-2.7.3.jar
memcached-session-manager-tc7-1.6.2.jar
memcached-session-manager-1.6.2.jar
##4. 修改tomcat/conf/server.xml文件
在Context元素中增加：
```
<Manager className="de.javakaffee.web.msm.MemcachedBackupSessionManager"
        memcachedNodes="n1:192.168.1.2:58728"
        sticky="false"
        sessionBackupAsync="false"
        requestUriIgnorePattern=".*\.(ico|png|gif|jpg|css|js)$"
        transcoderFactoryClass="de.javakaffee.web.msm.serializer.kryo.KryoTranscoderFactory"
/>
```
其中memcachedNodes里面应该填写能访问的memcache实际ip
##5.  在WEB应用的maven配置中增加
```
<dependency>
    <groupId>de.javakaffee.msm</groupId>
    <artifactId>msm-kryo-serializer</artifactId>
    <version>1.6.2</version>
</dependency>
```
相当于增加了如下几个jar：
    annotations-1.3.9.jar
    jsr305-1.3.9.jar
    kryo-1.04.jar
    kryo-serializers-0.9.jar
    minlog-1.2.jar
    msm-kryo-serializer-1.6.2.jar
    reflectasm-1.01.jar
##6. 重启tomcat
参见：
https://code.google.com/p/memcached-session-manager/wiki/SetupAndConfiguration#Add_custom_serializers_to_your_webapp_(optional)

此方案是基于nginx轮询tomcat方式，如果是基于iphash等其他方式，可以修改相关配置，以获取最佳性能。