---
layout: post
title: "火焰图(FlameGraphs)（一）之基础篇"
description: 你能快速定位程序性能问题？当你的工作环境非常复杂且变化迅速，性能问题往往复杂难辨，可能会为这个问题解决数天。在这里我将介绍一个简单好用的工具火焰图(FlameGraphs)。 
img: flameGraphs-1.png
color: 4CAF50
author: Carlwang
markdown: kramdown
highlighter: rouge
---

* some text
{: toc}

## 环境准备
这里将介绍三种使用火焰图方式，三种方式需要的工具和操作步骤有所不一样，下面将针对三种方式逐一解释如何操作，本文属于基础篇，重点介绍环境搭建和基本用法。

- 方式一(java)：需要工具
``JDK1.8``
``FlameGraph``
``perf-map-agent``
- 方法二(java)：需要工具
``JDK1.8``
``JCMD``
``jfr-flame-graph``
``FlameGraph``
- 方法三(C/C++):需要工具
``perf``
``FlameGraph``

## 火焰图方式一
该方式适用于java应用。

### 安装JDK1.8
下载``JDK1.8``，安装完毕后配置``JAVA_HOME``,这一步是必须的并且必须是``JDK1.8``。

### 克隆perf-map-agent
{% highlight markdown %}
1.git clone --depth=1 https://github.com/jrudolph/perf-map-agent
2.cd perf-map-agent
3.cmake .
4.make
{% endhighlight %}

### 克隆FlameGraph
{% highlight markdown %}
1.>git clone --depth=1 https://github.com/brendangregg/FlameGraph
2.>sudo su
3.>export FLAMEGRAPH_DIR=/home/xxx/FlameGraph //这里是FlameGraph路径
4.>export PERF_RECORD_SECONDS=120    //这里是记录时间
{% endhighlight %}
到这不环境已经配置成功！接下来可以使用perf-map-agent生成性能火焰图。

### 配置JVM参数
在jvm参数中增加-XX:+PreserveFramePointer

### 生成火焰图
{% highlight markdown %}
>/home/xxx/perf-map-agent/bin/perf-java-flames 1122 -F 99 -a -g
{% endhighlight %}

- 1122    	是目标进程PID
- —F 99		是采样频率99
- -a		是所有CPU
- -g      	是采样堆栈

## 火焰图方式二
该方式适用于java应用。

### 安装JDK1.8
下载``JDK1.8``，安装完毕后配置``JAVA_HOME``,这一步是必须的并且必须是``JDK1.8``。

### 安装jfr-flame-graph
{% highlight markdown %}
1.>git clone https://github.com/chrishantha/jfr-flame-graph
2.>cd jfr-flame-graph
3.>./install-mc-jars.sh
4.>mvn clean install -U
{% endhighlight %}

### 配置JVM参数
{% highlight markdown %}
-XX:+UnlockCommercialFeatures -XX:+FlightRecorder -XX:FlightRecorderOptions=loglevel=info
{% endhighlight %}
这些参数是必须的，否则不会生成火焰图。

### 开始采样
{% highlight markdown %}
>sudo -u <java_user> -i jcmd <pid> JFR.start filename=/tmp/app.jfr duration=60s
{% endhighlight %}

<java_user> 就是java进程运行的账户。
<pid> 是目标进程PID。
filename 是输出文件生成位置。
duration 持续采样周期。

### 检查采样是否完成
{% highlight markdown %}
>sudo -u <java_user> -i jcmd <pid> JFR.check
{% endhighlight %}

### 生成火焰图
{% highlight markdown %}
>flamegraph-output.sh folded -f app.jfr -o app.txt 
>cat app.txt | FlameGraph/flamegraph.pl >app.svg
{% endhighlight %}

还有一种更简单的方式：
{% highlight markdown %}
>./create_flamegraph.sh -f app.jfr >my.svg
{% endhighlight %}

## 火焰图方式三
该方式适用于C、C++，相对java来说要简单很多。

### Perf采样
{% highlight markdown %}
> perf record -p <pid> -F 99 -a -g -- sleep 60
{% endhighlight %}
这一步会生成perf.data文件。

### 创建中间文件
{% highlight markdown %}
>perf script | ./stackcollapse-perf.pl > out.perf-folded
{% endhighlight %}
这个命令会读取perf.data文件，script是必须的。

### 生成火焰图
{% highlight markdown %}
> ./flamegraph.pl out.perf-folded > perf-kernel.svg
{% endhighlight %}

## 效果图
![display]({{site.baseurl}}/images/flamegraph-example-1.svg)
