---
title: 白嫖jsDelivr作为自己网站的cdn
top: false
cover: false
toc: true
mathjax: true
date: 2020-08-08 16:00:02
password:
summary:
tags:
categories: 网络
thumbnail:
---

jsDelivr是一个免费的，为开源代码提供内容分发服务的服务商。
官网上它自己的描述是：
![](/images/白嫖jsDelivr作为自己网站的cdn/描述.png)
所谓cdn，就是Content Delivery Network内容分发网络的缩写。它的主要功能是，将你提供的内容缓存在服务商提供的不同的服务器中，就好像物流的存储仓库一样，深圳生产的手机，原厂有库存，北京的仓库有库存，广州的顾客下订单就从深圳走，天津的顾客下订单就从北京的仓库走。好处显而易见，快的同时还分担了流量的压力。
通常来说，cdn服务商会根据流量或者缓存大小等收取费用，如果我们只是凭借自己的兴趣搭了个小网站，肯定能白嫖尽量白嫖啊。所以，免费的jsDelivr是非常合适的选择。
那么，怎么使用呢？

<!--more-->

jsDelivr不需要你上传任何内容，它天然提供对npm，github和wordpress上开源资源的内容分发。
以大家比较熟悉的github举例，假设你有一组图片需要使用jsDelivr的加速，你需要在github上创建一个public的repo，将你的图片上传。当你的网站中需要这张图片时，使用
```
https://cdn.jsdelivr.net/gh/${user}/${repo}@${branch}/${file}
```
就可以享受到加速了。其中`${user}`是你的github用户名，`${repo}`是你的项目名，`${branch}`是项目分支的名字，如果使用项目的Release包，就填Release的版本号，`${file}`就是这个repo下的文件名了。
首次使用十分便利，立竿见影。但需要注意的是，如果你想修改文件内容，虽然图片一般不会修改，但有时我们加速的不是图片，而是js代码，当代码修改，即使推送到了github上的相应分支，jsDelivr也不会即使更新它的缓存，目前，官网也没有提供清空或者更新缓存的接口，查阅网上的资料，大多都说是随缘更新。我看到有人说将分支名称改为`latest`可以做到及时更新，实测下来并不可以。如果使用的是Release包，这个问题会小些，因为改变文件内容意味着必须发布一个新的Release版本，引用相应文件的地址也必须改变。
加速npm包与加速github上的文件类似，使用
```
https://cdn.jsdelivr.net/npm/${package}@${version}/${file}
```
其中`${package}`是在npm上的包名，`${version}`是版本号，`${file}`就是包内的文件了。
发布npm包其实比创建github上的项目还要简单，首先登陆npm的官网注册一个账号，并通过邮箱验证。然后在已经安装过npm情况下，通过`npm login`登陆，使用`npm init`创建包的配置文件，包括包名，版本，描述，作者，邮箱等，最后使用`npm publish`就可以将该路径下的文件打包发布了。
作为白嫖的cdn，除了更新缓存存在缺陷外，已经挑不出什么毛病了。祝你使用愉快。
