> 在前两天的时候，打开[www.linux.org](https://www.linux.org/)的时候突然发现，页面跟原来完全不一样了，里面各种fk，第一反应就是网站被黑了，攻击者对于该站非常不满，还有一张“菊花图”，场面异常火爆，有兴趣可看这个[报道](http://www.10tiao.com/html/739/201812/2649445233/1.html)。后来网站官方说网站被黑是一起[DNS劫持事件](https://www.linux.org/threads/linux-org-dns-hijack-incident.21073/)，对于网站被黑感到啼笑之余，也同时让我好奇DNS劫持到底是什么，攻击威力如此大。
<div align=center><img src="https://github.com/FantasyGao/FantasyGao.github.io/blob/master/imgs/20181208_1.jpg" height="40%" width="60%"/></div>

## 1. DNS是什么
DNS是Domain Name System的缩写, 我们称之域名系统。首先它是远程调用服务，本地默认占用53端口，它本身的实质上一个域名和ip的数据库服务器，他要完成的任务是帮我们把输入的域名转换成ip地址，之后通过ip寻址连接目标服务器。通过一个例子说一下我的理解，当我们上网的时候如果通过域名（如www.taobao.com）访问一个网站，流程应该是输入网址，浏览器缓存中查找www.taobao.com的ip，如果你刚刚访问过，则直接返回ip，如果没找到，则先进入本机的hosts文件找有没有这个域名，有的话返回对于ip，没有的话，进入本地DNS解析器中查找缓存，找不到的情况下则需要网络中的服务器去查找，首先查找本地DNS配置的服务器，如我们熟悉的谷歌的8.8.8.8和电信的114.114.114.114这两个(mac上的配置地址文件 /etc/resolv.conf)，都是在我们机器上事先配置好的，访问这个服务器如果在其缓存中查到对于的ip则直接访问给我们，同时本机的DNS解析器缓存该记录，如果服务器也没有找到这个域名的信息，这时候要看我们本地的配置是否需要转发，如果需要就需要本地DNS服务器一级一级向上查询，知道返回域名信息，不是转发的情况下，本地DNS服务器开始与根DNS服务器交互，当然根DNS服务器并没有我们想要的ip信息，由于全球都需要依赖它，它只会返回一些基本信息，在此时它先返回给我们.com这个顶级域名管理服务器的ip，本地DNS服务器拿到这个ip再向它寻找，当然.com的域名他也不会全存储，它会返回二级域名taobao.com的管理服务器ip地址，本地DNS服务器再次查找返回给我们www.taobao.com的ip地址，本地DNS服务器返回给客户端，只会客户端根据ip寻址，连接目标服务器。
#### 在这个过程中你可以实践这些操作。
 1. 先在浏览器打开www.taobao.com 之后在本机hosts文件中修改 www.taobao.com 指向的ip，你会发现浏览器打开淘宝还是正常，这时候你换一个没有打开过淘宝的浏览器，访问跳转至你hosts中配置的ip了。
 2. 利用tcpdump抓包观察。命令行执行 sudo tcpdump -v port 53， 你会发现你之前要是打开过www.taobao.com， 它是不会输出访问数据请求信息的。但是你访问一个没有访问过的域名的时候,会有一些访问DNS服务器的信息。如下图：
<div align=center><img src="https://github.com/FantasyGao/FantasyGao.github.io/blob/master/imgs/20181208_2.jpg" height="40%" width="60%"/></div>
