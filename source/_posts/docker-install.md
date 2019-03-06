---
title: Docker安装的问题
tags: Docker
comments: false
date: 2017-07-18 10:48:42
---
# Windows7
<!--more-->
![这里写图片描述](http://img.blog.csdn.net/20170627104103415?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvU2VyX0JhZA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
Windows7安装docker的时候，困扰了好久没有安装成功，今天终于试成功了。
之前一直报boot2docker找不到，仔细检查了报错的日志信息，其中一条就是下载boot2docker失败，但是我明确自己是下载成功的，最后直接把``boot2docker.iso`` 放到 ```C:\Users\user\.boot2docker``` 就好了，然后就可以直接运行了。

# Windows10
win10，直接在官网上下载Docker安装就好了，但是遇到一个问题，就是IDEA始终无法通过API URL：``tcp://localhost:2375`` 连上本地的docker，后来查遍论坛github才知道是下面的原因，打上勾就好了，真的是一大坑。
![这里写图片描述](http://img.blog.csdn.net/20170718104004533?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvU2VyX0JhZA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

# macOS10.10.3
macOS10.10.3以上也是一样的，直接安装[docker](https://store.docker.com/editions/community/docker-ce-desktop-mac)就好了。

# Intellij Idea
Intellij Idea使用docker的话，需要安装docker integration插件。

以上，没什么特别的内容只是自己在安装过程踩过的一个坑。
