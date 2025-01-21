---
layout: post
title:  "Spring Content Negotiation"
date:   2020-07-16 00:00:00 +0800
---

Content Negotiation 是 HTTP 协议的术语，其实任何通信协议都存在内容协商的过程

Spring Web对HTTP协议的内容支持是跟着HTTP协议标准走的，主要还是通过``Accept``和``Content-Type`` header 来判断具体应该使用哪种 ``HttpMessageConverter`` 来做数据解析，当然还有一些其他的补充决策策略，但这不是本文章的重点。你也可以自己定义``HttpMessageConverter``。

Spring 解析输入参数和返回数据时，会分别调用相应的

- 输入参数解析类 ``HandlerMethodArgumentResolver#resolveArgument``
- 返回值处理类 ``HandlerMethodReturnValueHandler#handleReturnValue`` 

对数据进行处理。