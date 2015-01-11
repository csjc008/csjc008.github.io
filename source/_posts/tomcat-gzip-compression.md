title: Tomcat 开启gzip压缩
date: 2015-01-11
tags:
---
如果tomcat返回的相应中含有大量文本数据，我们可以考虑开启tomat的gzip压缩功能以缩短传输时间，获取更好响应。
<!--more-->
下面是我在`conf/server.xml`中的配置片段：
```xml
<Connector port="9915" protocol="HTTP/1.1"
           maxThreads="200" connectionTimeout="20000"
           enableLookups="false" compression="on"
           redirectPort="8443"
           URIEncoding="UTF-8"
           compressionMinSize="1024"
           compressableMimeType="text/html,text/xml,text/plain,text/javascript,text/csv,application/javascript,application/json,application/xml"
/>
```
请注意`compress`打头的三段配置。这里加入了XML，csv，json等格式的压缩，指定在1024以上的长度执行gzip压缩返回。

还有一种方法让返回的文本开启gzip压缩。如果你的tomcat前面有nginx的话，可以在nginx上加上如下配置：
```nginx
gzip                    on;
gzip_http_version       1.1;
gzip_buffers            256 64k;
gzip_comp_level         5;
gzip_min_length         1024;
gzip_types              text/javascript application/x-javascript text/css text/plain;
```
自己调整`gzip_types`就可以了。