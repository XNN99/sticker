---
layout: post
title: "C++调用C#探讨"
description: 在平时的开发中，我们经常会遇到跨语言调用情况，方法很多，但如何选择呢？只要你手中东西越多，那么你选择的空间也会越大。在过往的经验告诉我，越简单才是越好的。这里我将介绍一种实现方式。
img: Cplusplus-1.png
color: 37474F
author: Carlwang
markdown: kramdown
highlighter: rouge
---

* some text
{: toc}

## 前言
这里我们讨论的话题，我们对于C#调用Native c++库都耳熟能详。
C++部分：
{% highlight cplusplus %}
int StaticElementNumber = 10;
extern "C" AFX_API_EXPORT int GetArrayElementNumber()
{
    ....
}

{% endhighlight %}

C#部分：
{% highlight csharp %}
[DllImport("MFCDll.dll")]
public static extern int GetArrayElementNumber();
{% endhighlight %}

就上面寥寥数笔就解决问题。

今天的话题是倒转过来，Native C++如何调用C#？

## 现有的技术
1. Native C++使用Managed C++间接调用C#。
2. Native C++直接调用C#实现COM组件。

- 对于第一种技术，优雅，易懂，不引入过多复杂环节。缺点在于Managed C++编译只能选择MD/MDd,部署需要安装运行时库，因此稍显复杂。同时也会多了Managed C++转发层。
- 对于第二种技术，采用COM技术，部署不需要额外安装运行时库。缺点在于引入COM，复杂性增加。

这里我们将讨论第二种技术。

## 准备工作
准备知识：
- 在 Microsoft.NET 框架中的 COM 互操作性。
- 编写COM客户端。

准备工具：
- Visual studio 2013

## 开始工作
