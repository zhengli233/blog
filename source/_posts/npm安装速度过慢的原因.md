---
title: npm安装速度过慢的原因
top: false
cover: false
toc: true
mathjax: true
date: 2020-01-14 18:01:50
password:
summary:
tags:
categories:
---

如果普通的去搜索npm下载速度过慢之类的问题，无论百度或是谷歌，给出的答案都是你应该使用taobao的镜像源，然后列出了如何修改npm包源的方法。
但这个方法并不适合我的情况。
在尝试了各种方法后，我抱着试一试的心态用英文在谷歌上搜了下，得到了stackoverflow上的答案https://stackoverflow.com/questions/41524903/why-is-npm-install-really-slow。
实际的情况确实是我的npm版本低了。我使用的是ubuntu18.04,apt维护的npm版本为3.5.2,使用命令
`sudo npm install npm@latest -g`
升级到npm的最新版本。目前，我的版本升级到了6.13.6。
猴调博客的框架使用的是hexo，可以在网页的最底端看到。npm升级前花了若干小时都没有安装好，升级后也就是几秒钟的事。顺便我也把node升级了。
<!--more-->
其实问题本身并不复杂，但中文的搜索结果给出的答案总是千篇一律，literally千篇一律，这其实挺有问题的。我搜索一些技术上的问题，不止一次地将问题转化成英文才有靠谱的答案。CSDN和简书上你抄我我抄你，哪怕翻译点英文的回来也好啊。
笔者的技术水平也有限，猴调博客的内容也许不会那么高大上，但至少能保证每一篇都是原创，即使借鉴了别人的，也一定会有所注明。
