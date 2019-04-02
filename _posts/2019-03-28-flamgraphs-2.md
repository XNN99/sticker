---
layout: post
title: "火焰图(FlameGraphs)（二）之读懂火焰图"
description: 你能快速定位程序性能问题？当你的工作环境非常复杂且变化迅速，性能问题往往复杂难辨，可能会为这个问题解决数天。在这里我将介绍一个简单好用的工具火焰图(FlameGraphs)。 
img: flameGraphs-1.png
color: 4CAF50
author: Carlwang
markdown: kramdown
highlighter: rouge
---

* some text
{: toc}

## perf命令
它是 Linux 系统原生提供的性能分析工具，会返回 CPU 正在执行的函数名以及调用栈（stack）。

{% highlight markdown %}
$ sudo perf record -F 99 -p 13204 -g -- sleep 30
{% endhighlight %}

上面的代码中，perf record表示记录，-F 99表示每秒99次，-p 13204是进程号，即对哪个进程进行分析，-g表示记录调用栈，sleep 30则是持续30秒。

## 火焰图的含义
火焰图是基于 perf 结果产生的 SVG 图片，用来展示 CPU 的调用栈。

![flame-example]({{site.baseurl}}/images/flameGraph-1.jpg)

y 轴表示调用栈，每一层都是一个函数。调用栈越深，火焰就越高，顶部就是正在执行的函数，下方都是它的父函数。
x 轴表示抽样数，如果一个函数在 x 轴占据的宽度越宽，就表示它被抽到的次数多，即执行的时间长。

注意：火焰图就是看顶层的哪个函数占据的宽度最大。只要有"平顶"（plateaus），就表示该函数可能存在性能问题。

### 火焰图实例
下面是简化版火焰图实例

![flame-example2]({{site.baseurl}}/images/flame-example-2.jpg)

上面图片中，最顶层的函数g()占用 CPU 时间最多。d()的宽度最大，但是它直接耗用 CPU 的部分很少。b()和c()没有直接消耗 CPU。因此，如果要调查性能问题，首先应该调查g()，其次是i()。
另外，从图中可知a()有两个分支b()和h()，这表明a()里面可能有一个条件语句，而b()分支消耗的 CPU 大大高于h()。

## 效果图
[FlameGraph]({{site.baseurl}}/images/flamegraph-example-1.svg)
![FlameGraph]({{site.baseurl}}/images/flamegraph-example-1.svg)
