---
layout: post
title: "Windows监控进程(一) C#技术实现篇"
description: 在这篇文章中将描述如何通过C#实现Windows进程活动信息监视，该方式只是多种进程监控方式种的一种，属于轻量级实现方式。 
img: image-1.jpg
color: 9E9D24
author: Carlwang
---

* some text
{: toc}

## 场景一
在客户环境中，我常遇到产品程序平凡重启动但始终无法启动成功这类问题，加之客户环境对我们是不见的。由于种种限制，要想找到RootCase困难重重。但再难的问题想尽一切办法、扣破脑袋也必须要解决。因为客户是我们的上帝。首先想到的监控进程启动停止信息，那到第一手信息最为关键。

## 场景二
涉足Windows监控领域多年，我们常常需要对用户系统中程序使用情况做监控，获取足够多的信息，以便进一步分析用户行为。然而以往经验都是在内核层Hook SSDT等内核驱动技术实现，这些技术方式从系统级实现，技术太重，要求太高，代价太大。

> 那么有没有一种轻量级的技术来实现。ETW(Event Tracing for Windows)提供了一种对用户层应用程序和内核层驱动创建的事件对象的跟踪记录机制。它提供了跟踪进程启动与停止事件。ETW 的真正优点之一是你可以用自己的操作关联系统生成的事件，当系统事件发生时，你关联的操作会收到该类事件。
> 这篇文章我将实现一个简单应用程序，它调用``ProcessMonitor``来监视系统中进程启动和停止。我们会看到它是如此的简单，代码也不超过50行。这篇采用C#语言实现。

## 实现依赖
C#实现进程启动停止行为监视需要依赖``TraceEvent``库，该库可以通过``程序包管理控制台``获取安装。
{% highlight csharp %}
PM> Install-Package Microsoft.Diagnostics.Tracing.TraceEvent
{% endhighlight %}

## 主要逻辑
{% highlight csharp %}
var sessionName = "ProessMonitorSession";
using (var session = new TraceEventSession(sessionName, null))  
{
    session.StopOnDispose = true;

    Console.CancelKeyPress += delegate(object sender, ConsoleCancelEventArgs e) 
	{ 
		session.Dispose(); 
	};

    using (var source = new ETWTraceEventSource(sessionName, TraceEventSourceType.Session))
    {
        Action<TraceEvent> action = delegate(TraceEvent data)
        {
            var taskName = data.TaskName;
            if (taskName == "ProcessStart" || taskName == "ProcessStop") 
            {
                string exe = (string) data.PayloadByName("ImageName");
                string exeName = Path.GetFileNameWithoutExtension(exe);

                int processId = (int) data.PayloadByName("ProcessID");
                if (taskName == "ProcessStart")
                {
                    int parentProcessId = (int)data.PayloadByName("ParentProcessID");
                    Console.WriteLine("{0:HH:mm:ss.fff}: {1,-12}: {2} ID: {3} ParentID: {4}", 
                        data.TimeStamp, taskName, exeName, processId, parentProcessId);
                }
                else
                {
                    int exitCode = (int) data.PayloadByName("ExitCode");
                    long cpuCycles = (long) data.PayloadByName("CPUCycleCount");
                    Console.WriteLine("{0:HH:mm:ss.fff}: {1,-12}: {2} ID: {3} EXIT: {4} CPU Cycles: {5:n0}",
                        data.TimeStamp, taskName, exeName, processId, exitCode, cpuCycles);
                }
            }
        };

        var registeredParser = new RegisteredTraceEventParser(source);
        registeredParser.All += action;

        var processProviderGuid = TraceEventSession.GetProviderByName("Microsoft-Windows-Kernel-Process");
        if (processProviderGuid == Guid.Empty)
        {
            Console.WriteLine("Error could not find Microsoft-Windows-Kernel-Process etw provider.");
            return -1;
        }

        // Using logman query providers Microsoft-Windows-Kernel-Process I get 
        //     0x0000000000000010  WINEVENT_KEYWORD_PROCESS
        //     0x0000000000000020  WINEVENT_KEYWORD_THREAD
        //     0x0000000000000040  WINEVENT_KEYWORD_IMAGE
        //     0x0000000000000080  WINEVENT_KEYWORD_CPU_PRIORITY
        //     0x0000000000000100  WINEVENT_KEYWORD_OTHER_PRIORITY
        //     0x0000000000000200  WINEVENT_KEYWORD_PROCESS_FREEZE
        //     0x8000000000000000  Microsoft-Windows-Kernel-Process/Analytic
        // So 0x10 is WINEVENT_KEYWORD_PROCESS
        session.EnableProvider(processProviderGuid, TraceEventLevel.Informational, 0x10);

        Console.WriteLine("Starting Listening for events");
        source.Process();
        Console.WriteLine();
        Console.WriteLine("Stopping Listening for events");
}

{% endhighlight %}

未完待续....