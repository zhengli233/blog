---
title: 关于将猴调网前端部署到coding page遇到的问题
top: false
cover: false
toc: true
mathjax: true
date: 2020-01-15 11:21:51
password:
summary:
tags:
categories:
---

## 构思
猴调网的架构是Vue前端+Flask后端+MySQL数据库。之前所有的东西全部都部署在我的aws上，使用nginx作为前端服务器，gunicorn作为后端服务器。在实际使用中，后端只对内网开放，外部是无法进行api请求的。
这样的架构其实对于一个个人小网站来说是绰绰有余的了，也完全能够正常访问。但由于aws服务器是在新加坡，这就导致国内的访问速度较慢。特别是猴调的前端有时候连github的仓库都连不上，打开猴调网有时候就是打不开。那么我作为运维，就要想办法了。
通常，人们心血来潮想做个小网站又不想花钱买服务器部署的话，都会去使用github page或者coding page来发布自己的网站。在上述地方发布是完全免费的，只需要你把网页代码静态化上传到指定仓库就可以了。具体步骤可以去看github和coding自家的手册，coding手册不是很updated，但不在这里讨论了。
显然，这样发布网站，你是不可能拥有后端的。但我有自己的服务器，我只需要前端去请求我服务器端的api就可以了，前端资源从国内的服务器下载到浏览器应该是比从新加坡来的快的。
于是我就没有更改前端代码，直接发布到了coding page上，后端gunicorn配置对公网ip开放。遇到的第一个问题就是，api地址的错误。

## 问题一：api地址错误
前端代码中，向后端发起请求是这样实现的
```js
this.axios.get('/api/material/list?name=' + this.searchName)
```
axois默认的地址前缀就是我们网页部署的域名。所以整个api请求内容就是
`http://qztejm.coding-pages.com/api/material/list?name=searchName`
这不是我想要的，我需要将地址前缀修改为我后端服务器的ip+端口。
这并不难解决，只要修改axios的默认地址前缀就好
```js
axios.defaults.baseURL = 'http://52.220.251.13:8000'
```
但是即使这样做，这个请求还是不对。
在我的后端处理请求时，接受的请求内容中是不带“/api”的，像这样
```python
@APP.route('/material/list', methods=['GET'])
```
所以真正正确的请求内容应该是
`http://52.220.251.13:8000/material/list?name=searchName`
但之前前端部署在aws上明明可以正确发起请求，甚至使用的都是默认的80端口而不是后端开放的8000端口，为什么现在就不行了呢。
这里其实是因为之前是部署在nginx服务器上，服务器的配置中有这么一条
```
location / {
    try_files $uri $uri/ @router;
    index index.html;
}

location /api {
    proxy_set_header X-real-ip $remote_addr;
    proxy_pass http://127.0.0.1:8000/;
}

location @router {
    rewrite ^.*$ /index.html last;
}
```
可以看到，对`/api`路径的访问，nginx都帮我们转到`http://127.0.0.1:8000/`去了，所以在后端接收到的请求中，没有`/api`。
那么，对于发布到coding page的版本，我们把`/api`去掉，路径也就正确了。在chrome中观察，请求确实没错了，但是产生了另一个问题。

## 问题二：CORS block
所谓CORS，就是cross-origin source sharing，就是我明明访问的是A域名，却去请求了B域名。这看似没什么问题，前后端分离不就是这样么。但这会带来一个隐患：假如一个网页对其他各种需要登录验证的网站如微博、知乎、github等发起请求，如果恰好你已经处于登录的状态，那么这个网页就能获得你在登录状态能获得的资源。所以现代的浏览器，比如我用的chrome，就会默认有一个cors block。当浏览器发现cors行为时，会预先对该请求做验证，它会询问被请求的服务器是否允许这样的行为。一般来说，后端服务器只对自己信任的域名cors或者根本不允许。此时服务器会回复浏览器它信任的域名，供浏览器选择是否拦截这个cors请求。如果像我一样，没有在后端配置信任的域名，那么浏览器的询问得不到回答，于是对后端的请求就被拦截了。(https://dev.to/nicolus/what-you-should-know-about-cors-48d6)
那么如何去配置呢？
对于Flask来说，非常容易。
```python
from flask_cors import CORS

APP = Flask(
    __name__
)
CORS(APP, resources={r'*': {'origins': r'http://qztejm.coding-pages.com/*'}})
```
`r'*'`表示对所有route配置都生效，`'origins': r'http://qztejm.coding-pages.com/*'`表示该域名是被我们允许的源域名。
这样，重新部署一下后端就可以正常访问了。

## coding page的问题
解决了上面两个问题后，其实网页就已经可以正常访问了，但是，在刚部署完成时，我的访问特别不稳定，从浏览器里也看不出什么毛病。不过在我写到这里的时候，访问已经正常了，并且速度也确实超过了从aws的访问。但愿之后不会出什么问题。
