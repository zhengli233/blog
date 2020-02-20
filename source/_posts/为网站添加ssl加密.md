---
title: 为网站添加ssl加密
top: false
cover: false
toc: true
mathjax: true
date: 2020-01-21 15:40:22
password:
summary:
tags:
categories: 网络
thumbnail: /images/为网站添加ssl加密/http-to-https.png
---

使用浏览器浏览网页的时候，在地址栏的左边会有一个符号
![](/images/为网站添加ssl加密/加密.png)
![](/images/为网站添加ssl加密/未加密.png)
加锁的表示启用了ssl，查看完整url的话可以看到是以`https://`开头的，而显示`not secure`的则是以`http：//`开头的。
它们的区别在于，`https`传输的内容是加密的，而`http`则是以明文传输的，也就是说，如果有人在你的传输线路上抓包，走`http`包的内容别人可以一目了然，像`username`和`password`这种敏感信息，以及会暴露xp的搜索内容，也许就会被有心之人所利用。有兴趣的可以搜索一下中间人攻击，在这里就不展开讲了。
现代的浏览器对于不加密的网址，往往都会给出提示。像firefox会弹出悬窗提示你该网站安全等级较低，chrome做的更绝，直接把页面block，除非你选择高级选项继续访问[(https://juejin.im/post/5b57d58d6fb9a04fda4e1d82)](https://juejin.im/post/5b57d58d6fb9a04fda4e1d82)。
对于搜索引擎排名优化而言，`https`的排名应该更有优势，至少谷歌是这样的。百度在早些年对`https`并不是很友好，甚至建议站长们提供`http`页面供它爬取[(http://www.chinaz.com/web/2014/0916/367881.shtml)](http://www.chinaz.com/web/2014/0916/367881.shtml)。
运营商劫持。
<!--more-->
加密的好处大概就是以上这些，坏处就是加大了客户端和服务端的工作量，客户端会花更多的时间打开网页，服务端则会花费更多的资源去加解密。而且虽然加密了，但却不是完全无懈可击的，如果根证书就错了，那信任链整个就错了[(https://segmentfault.com/a/1190000013075736)](https://segmentfault.com/a/1190000013075736)。

说了这么多，到底怎么给自己网站启用`https`呢。下面我说说我们的猴调网是怎么启用的。
最开始，猴调网使用的是`http`，并且连域名都没有，访问是靠直接输ip的。在服务器端，nginx也只设置了默认80端口的服务器。要想使用ssl加密，就必须拥有一个证书，这个证书是受信任的机构颁发给你的。网上有很多攻略，比如[从let's encrypt获取免费的证书](https://diamondfsd.com/lets-encrytp-hand-https/)。但是，使用它的前提是，你有一个域名，不能一个裸的ip。
那么，买个域名吧。在杭州的老马，深圳的老马和北京的老李之间，我最终选择了给深圳的老马一个坐公交车的机会。腾讯云的`.club`域名一块钱一年，跟白送差不多。另外，域名证书也能免费提供，所以直接用它的就行了。
在下载它的证书之前，要先讲一下猴调网哪里需要用到这个证书。
现在的猴调网的前端，不再是放在aws的服务器上了，我托管给了coding page，因为国内访问新加坡的服务器实在是又慢又不稳定。coding page上的域名可以绑定为`www.houtiao.club`以及`houtiao.club`这个主域名，让腾讯云解析到coding page原来的域名就可以了。并且，可以在coding page上直接设置为强制https访问，这是不需要我提供证书的，因为coding自己有。此时直接打开猴调网主页，就已经可以看到是ssl加密过的了。
但是，当猴调网去调用后端api时，问题来了，后端并没有做ssl加密。这种情况下会产生两种后果，一是请求后端api的子页面不再被认定为是加密的了，另一种是chrome阻止了未加密的请求。这两种情况我都遇到了，无论那种，都不是我们想要的。所以，后端的api请求也必须使用`https`。
之前没有申请域名走`http`的时候，我将后端的api接口直接暴露在公网上，前端只需要访问公网ip和端口就行了，这个后端是使用gunicorn部署的。
这里，我改变了之前的做法。我没有再使用暴露在公网的api接口，而是使用了Nginx的反向代理，将公网api请求转发给内网的gunicorn端口，这样就将ssl加密的工作交给了Nginx。我们只需要在Nginx的服务器配置文件中配置监听443端口(https请求的默认端口)，启用ssl加密，并指定证书的路径。
```
server {

  listen 443 ssl;

  server_name api.houtiao.club;

  ssl_certificate /home/ubuntu/upload/cert/Nginx/1_api.houtiao.club_bundle.crt;
  ssl_certificate_key /home/ubuntu/upload/cert/Nginx/2_api.houtiao.club.key;
  ssl_session_cache shared:SSL:1m;
  ssl_session_timeout 5m;
  ssl_ciphers HIGH:!aNULL:!MD5;
  ssl_prefer_server_ciphers on;

  location / {
    proxy_pass http://127.0.0.1:8000/;
  }
}
```
这样，后端的代码就不需要做更改，前端的请求也使用了ssl加密。
至此，猴调网就全面启用了`https`。