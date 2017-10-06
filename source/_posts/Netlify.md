---
title: Migrated from Heroku to Netlify
date: 2017-09-28 17:25:25
tags:
    - Netlify
    - migrate
    - Heroku
    - static site
categories:
    - general
---

之前学习ruby on rails的时候，开始接触的Heroku。这次部署个人博客，想着Heroku应该是个不错的选择吧，但试用了几天的Heroku之后，我决定还是转到其他地方部署我的blog，为什么呢？原因有二：

1. 免费的Heroku App，如果搁置时间长了一直无人访问的话，再次访问是需要一定时间等待激活的，就像是电脑睡眠了等着叫醒一样。付费的话太贵，对于只是搭载个人博客而言不太划算；
2. Heroku服务器是在美国和欧洲，而且一个App只能存在在某一个区域，不可变。所以对于国内的朋友，登录我的博客有时会有不小的延迟，虽然对于一个静态网站，本来也无需很快的响应速度，但配合第一条，这个时间有时真的让人无语……

<!--more-->

那之所以选择Netlify也是有两个原因：

1. 不存在Heroku的第一个问题；
2. 静态内容部署采用的是global CDN方式，这样用户登录博客的时候会根据用户所在地选取最近的节点获取信息。

除了这两点以外，Netlify的设置界面也做得十分简洁易用，相比Heroku而言，甚至连部署都更加简单，除了利用git部署，还可以直接把生成好的静态网页文件夹拖动到一个部署框内

![drag_deploy](/images/drag_deploy.png)