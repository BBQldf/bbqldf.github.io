---
layout:     post
title:     springMVC基础04
subtitle:   springMVC-异步、拦截、文件传输
date:       2021-12-19
author:     ldf
header-img: img/post-bg-springMVC01.jpg
catalog: true
tags:
    - java基础
    - springMVC
    - code
---

> 下面介绍一些SpringMVC框架中的扩展功能，但是也是必须掌握的基础功能！

# 一、Ajax研究

## 1、 简介

- AJAX = Asynchronous JavaScript and XML（异步的 JavaScript 和 XML）。
- AJAX 是一种在无需重新加载整个网页的情况下，能够更新部分网页的技术。

- Ajax 不是一种新的编程语言，而是一种用于创建更好更快以及交互性更强的Web应用程序的技术。
- Google Suggest 使用 AJAX 创造出动态性极强的 web 界面：**当您在谷歌的搜索框输入关键字时，JavaScript 会把这些字符发送到服务器，然后服务器会返回一个搜索建议的列表**。
- 就和国内百度的搜索框一样!

- 传统的网页(即不用ajax技术的网页)，想要更新内容或者提交一个表单，都需要重新加载整个网页。

- 使用ajax技术的网页，通过在**后台服务器进行少量的数据交换**，就可以实现异步局部更新。

- 使用Ajax，用户可以创建接近本地桌面应用的直接、高可用、更丰富、更动态的Web用户界面。
