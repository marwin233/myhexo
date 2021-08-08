---
title: 关于RSS
date: 2020-02-27 21:24:41
tags:
---



## 一、RSS是什么

RSS是一种信息聚合的技术，都是为了提供一种更为方便、高效的互联网信息的发布和共享，用更少的时间分享更多的信息。

简单来说RSS就是以统一的格式发布内容，订阅了某个RSS源的客户端可以接受到该源的更新推送。类似订阅一份报纸后，当天最新的报纸就会准时递到你手中。而由于RSS统一格式的优势，不拘于新闻，各种博客，网站更新，各种app讯息都能订阅，这种统一的概念正是吸引我的地方之一。另外一个吸引我的地方是RSS只会让你接收你想要接收的内容，沉浸式无广告阅读，不会有“猜你喜欢”擅自用AI技术推送消息给你的情况出现。

其实我接触RSS也才一年，这段时间我就已经抛弃了大部分的资讯类的APP。事实上使我相见恨晚的RSS并不是什么新东西，早在十几年前RSS就出现了，在各种资讯APP群魔乱舞的时代RSS始终存在于一个小圈子里。

## 二、如何使用RSS

想要用RSS来阅读内容首先要有RSS阅读软件，这些软件搜一下就能找到很多，我目前使用的是Reeder（macOS，iPadOS），RSS Reader Prime（IOS，独立开发者制作的良心应用，完全免费）。

![https://i.loli.net/2019/11/30/5V86HYL1qocpg9M.png](https://i.loli.net/2019/11/30/5V86HYL1qocpg9M.png)



拥有一个RSS阅读器后，需要找到你想要的订阅的RSS源，一般是一个链接，如少数派的RSS源地址是 https://sspai.com/feed 。在阅读器中添加该订阅源后就能获取到最新文章的推送，一般的RSS阅读器都有已读未读，分类，更换主题等功能，阅读体验比资讯APP更好。另外推荐一个RSS源的索引： https://docs.rsshub.app/ ，里面有各种订阅源可以满足你的需要。

![https://i.loli.net/2019/11/30/ujt9grOMGSQloam.png](https://i.loli.net/2019/11/30/ujt9grOMGSQloam.png)



## 三、我制作的RSS工具

出于对RSS的喜爱，我想给我自己的博客（ http://www.marwin.cn ）添加RSS功能。事实上很多使用框架搭好的博客，如WordPress都自带了RSS功能，无奈我的博客是自己写的后台加静态的前端页面。于是我写了一个定时抓取博客更新并制作成RSS源的软件，在自己设定的时间周期里，绑定的各种类型的Trigger能爬取想要的内容的更新，然后生成RSS源。目前只将我自己的博客生成了RSS源： http://api.marwin.cn/rss/blog 。以后可以爬取其他各种感兴趣的内容然后订阅。

![https://tva1.sinaimg.cn/large/006tNbRwgy1g9gb73yii9j306w04u743.jpg](https://tva1.sinaimg.cn/large/006tNbRwgy1g9gb73yii9j306w04u743.jpg)

关于这个生成RSS的软件，我上传在自己的git仓库。接下来的计划是为自己的git仓库提供一个web页面，想要了解这个软件可以在这个页面获取副本的仓库链接。



> 更新：由于现在使用hexo搭建博客，配合hexo-rss插件能够直接发布RSS，本博客RSS地址为：`https://marwincn.github.io/blog/rss2.xml`

