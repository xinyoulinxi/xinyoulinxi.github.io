---
layout: post
author: "YL"
subtitle: ""
title:  "使用jekyll搭建github page"
date:   2021-07-31 23:31:36
header-img: /imgs/page_bg.jpg
tags:
    - jekyll
    - github
    - blog
    - page
---
# 使用jekyll搭建github page
### 本地调试

```shell
bundle install # 下载插件等，通过GemFile
bundle add webrick # 用于解决require的错误堆栈
bundle exec jekyll serve # 启动本地调试，一般是在本地的4000端口，也就是http://127.0.0.1:4000/
```
### 如何在post的md中加载根目录下的图片
假设图片的目录为: resource\imgs,在post_的md文件下引用图片的时候不能和下面一样：

`![](imgs/feature_1.png)`

这样是从jekyll生成的site目录下的html文件路径下开始查找，显然找不到

而是应该使用如下路径样式，最前方加上/表示从项目根目录查询

`![](/imgs/feature_1.png)`

这样才是从根目录查询。
