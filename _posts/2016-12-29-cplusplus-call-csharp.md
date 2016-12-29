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

----------

C++部分：
{% highlight cplusplus %}
int StaticElementNumber = 10;
extern "C" AFX_API_EXPORT int GetArrayElementNumber()
{
    ....
}

{% endhighlight %}

----------

C#部分：
{% highlight csharp %}
[DllImport("MFCDll.dll")]
public static extern int GetArrayElementNumber();
{% endhighlight %}

----------

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

### 编写Managed DLL


- 通过visual studio 2013创建托管类库项目(取名：order)。如下图:

![VSProject]({{site.baseurl}}/images/vs-create-managed-proj.png)

- 首先申明Public接口和方法，这是必须的，Native C++客户端需要使用这个接口。

{% highlight csharp %}
	// Interface declaration.
	public interface IOrder
	{
	    int Count();
	};
{% endhighlight %}

- 在申明接口后，接下来需要实现该接口：

{% highlight csharp %}
	// Interface implementation.
	public class FruitOrder:IOrder
	{
	    public int Count()
	    {
	        return 12;
	    }
	}
{% endhighlight %}

- 为工程添加com互相操作性

只有添加了COM互操作性，这个程序集合才能在Native C++端被调用:

![com]({{site.baseurl}}/images/register-com.png)


- 强名称工具有助于使用强名称对程序集进行签名。

在目前位置工程生成的弱命名程序集，该类程序集只能进行私有部署(即与应用程序同一目录)，如果需要部署在全局缓存集GAC(Global Assembly Cache)，则需要提供强命名程序集。弱命名程序集和强命名程序集将在后面探讨。

如何产生强命名程序集？这里使用sn.exe工具。

1.下面的命令创建一个新的随机密钥对并将其存储在 keyPair.snk 中。

{% highlight markdown %}
sn -k keyPair.snk
{% endhighlight %}

2.修改AessmbleInfo.cs

{% highlight csharp %}
[assembly: AssemblyDelaySign(false)] 
[assembly: AssemblyKeyFile("..\\..\\keyPair.SNK")]
{% endhighlight %}

- 编译工程生成托管库order.dll


### 注册COM库

托管库生成后，它是一个COM组件，因此还不能被Native C++使用，需要将其注册到系统中。

{% highlight markdown %}
RegAsm.exe order.dll /tlb:order.tlb /codebase
{% endhighlight %}

- /codebase

在注册表中创建一个 Codebase 项。Codebase 项指定未安装到全局程序集缓存中的程序集的文件路径。如果随后要安装正在注册到全局程序集缓存中的程序集，则不应指定此选项。用 /codebase 选项指定的 assemblyFile 参数必须是具有强名称的程序集。

- /tlb

生成COM类型库文件，这个文件包含了接口相关信息。需要使用COM类的模块需要使用#import xxx.tlb,编译器将生成相应的头文件。

### 从Native C++调用C#

- 创建Native C++工程(取名：client)

![client]({{site.baseurl}}/images/create-client-cpp.png)

- 在工程中导入类型库order.tlb

打开client.cpp,添加下来代码：

{% highlight cplusplus %}

	// Import the type library.
 	#import "..\order.tlb" raw_interfaces_only
	using namespace ManagedDLL;

{% endhighlight %}

- 调用托管库

{% highlight cplusplus %}
	// Initialize COM.
	HRESULT hr = CoInitialize(NULL);
	
	// Create the interface pointer.
	IOrderPtr pIorder(__uuidof(FruitOrder));
	
	long lResult = 0;
	
	// Call the count method.
	pIorder->count(&lResult);
	
	wprintf(L"The result is %d", lResult);
	
	// Uninitialize COM.
	CoUninitialize();
	return 0;
{% endhighlight %}

编译生成文件，运行，可以看到结果。

> 到此位置Native C++如何调用Managed库就将到这里。什么是弱命名程序集和什么是强命名程序集，将在以后博客中介绍。