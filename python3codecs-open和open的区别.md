---
title: python3codecs.open和open的区别
author: Juntech
top: false
cover: false
toc: true
mathjax: false
copyright: true
categories: python
tags: python
keywords: python
abbrlink: 7b216b49
date: 2019-09-25 11:02:23
summary:
---

Python3直接用open。
Python2.x下用codecs.open，特别是有中文的情况，然后也可以避免踩到2.6下面io.open的坑。
如果希望代码同时兼容Python2和Python3，那么推荐用codecs.open。