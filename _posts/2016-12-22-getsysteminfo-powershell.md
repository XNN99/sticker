---
layout: post
title: "获取系统信息之powershell"
description: 通过这篇文章我将探讨如何通过Powershell获取系统信息，体会powershell的魔法魅力。 
img: image-9.jpg
color: ff1744
author: Carlwang
---

* some text
{: toc}

## 前言
我们在开发软件时常常需要获取操作系统信息，如：``”主机名“``，``”操作系统名“``，``”操作系统版本“``,``“系统总内存”``,``“系统可用内存”``等等。在Windows下通常有以下几种实现方式：
1. 使用Powershell+systeminfo实现。
2. 调用Win32API方式实现（GetVersionEx，GlobalMemoryStatusEx，GlobalMemoryStatus）。
3. 调用WMI实现，Win32_OperatingSystem，Win32_DiskDrive，Win32_PhysicalMedia。
在这篇文章中我将探讨第一种实现方式。后续文章再探讨其它技术实现。

## 实现
{% highlight powershell %}
systeminfo.exe /FO CSV |
  Select-Object -Skip 1
{% endhighlight %}
- systeminfo.exe是windows系统命令，/FO指定输出格式，可选项："TABLE", "LIST", "CSV"。
- select-object 从一个对象或一个对象集中选择指定的属性集。也可以对象数组中选择唯一对象，也可以从开始到结束位置选择指定数量对象。
- -Skip[number] 跳过指定数量的项。



> 上面代码意思：执行systeminfo命令将结果以CSV格式输出，并将结果传给Select-Object函数，Select-Object函数除第一项外选择所有的。

结果如下：
{% highlight markdown %}
PS > systeminfo.exe /FO CSV | Select-Object -Skip 1
"WIN-RGSKMLQ9ID2","Microsoft Windows 7 Professional ","6.1.7601 Service Pack 1 Build 7601","Microsoft Corporation","Sta
ndalone Workstation","Multiprocessor Free","Windows User","","00371-177-0000061-85912","12/14/2016, 3:35:10 PM","12/22/
2016, 9:49:34 AM","VMware, Inc.","VMware Virtual Platform","x64-based PC","1 Processor(s) Installed.,[01]: Intel64 Fami
ly 6 Model 70 Stepping 1 GenuineIntel ~2194 Mhz","Phoenix Technologies LTD 6.00, 5/20/2014","C:\Windows","C:\Windows\sy
stem32","\Device\HarddiskVolume1","en-us;English (United States)","en-us;English (United States)","(UTC+08:00) Beijing,
 Chongqing, Hong Kong, Urumqi","4,095 MB","2,447 MB","8,189 MB","6,499 MB","1,690 MB","C:\pagefile.sys","WORKGROUP","\\
WIN-RGSKMLQ9ID2","2 Hotfix(s) Installed.,[01]: KB2534111,[02]: KB976902","2 NIC(s) Installed.,[01]: Intel(R) PRO/1000 M
T Network Connection,      Connection Name: Local Area Connection,      DHCP Enabled:    Yes,      DHCP Server:     172
.16.74.254,      IP address(es),      [01]: 172.16.74.143,      [02]: fe80::3591:8ee4:d1a2:a74f,[02]: Bluetooth Device
(Personal Area Network),      Connection Name: Bluetooth Network Connection,      Status:          Media disconnected"
{% endhighlight %}

上面结果太凌乱，怎么办呢？进行下面的处理。

{% highlight powershell %}
$header = 'Hostname','OSName','OSVersion','OSManufacturer','OSConfig','Buildtype',`
'RegisteredOwner','RegisteredOrganization','ProductID','InstallDate','StartTime','Manufacturer',`
'Model','Type','Processor','BIOSVersion','WindowsFolder','SystemFolder','StartDevice','Culture',`
'UICulture','TimeZone','PhysicalMemory','AvailablePhysicalMemory','MaxVirtualMemory',`
'AvailableVirtualMemory','UsedVirtualMemory','PagingFile','Domain','LogonServer','Hotfix',`
'NetworkAdapter'
systeminfo.exe /FO CSV |
  Select-Object -Skip 1 |
  ConvertFrom-CSV -Header $header
{% endhighlight %}

结果漂亮多了!
{% hightlight markdown %}
PS C:\> $header = 'HostName','OSName','OSVersion','OSManufacturer','OSConfig','Buildtype',`
>> 'RegisteredOwner','RegisteredOrganization','ProductID','InstallDate','StartTime','Manufacturer',`
>> 'Model','Type','Processor','BIOSVersion','WindowsFolder','SystemFolder','StartDevice','Culture',`
>> 'UICulture','TimeZone','PhysicalMemory','AvailablePhysicalMemory','MaxVirtualMemory',`
>> 'AvailableVirtualMemory','UsedVirtualMemory','PagingFile','Domain','LogonServer','Hotfix',`
>> 'NetworkAdapter'
>> systeminfo.exe /FO CSV |
>>   Select-Object -Skip 1 |
>>   ConvertFrom-CSV -Header $header
>>


HostName                : WIN-RGSKMLQ9ID2
OSName                  : Microsoft Windows 7 Professional
OSVersion               : 6.1.7601 Service Pack 1 Build 7601
OSManufacturer          : Microsoft Corporation
OSConfig                : Standalone Workstation
Buildtype               : Multiprocessor Free
RegisteredOwner         : Windows User
RegisteredOrganization  :
ProductID               : 00371-177-0000061-85912
InstallDate             : 12/14/2016, 3:35:10 PM
StartTime               : 12/22/2016, 9:49:34 AM
Manufacturer            : VMware, Inc.
Model                   : VMware Virtual Platform
Type                    : x64-based PC
Processor               : 1 Processor(s) Installed.,[01]: Intel64 Family 6 Model 70 Stepping 1 GenuineIntel ~2194 Mhz
BIOSVersion             : Phoenix Technologies LTD 6.00, 5/20/2014
WindowsFolder           : C:\Windows
SystemFolder            : C:\Windows\system32
StartDevice             : \Device\HarddiskVolume1
Culture                 : en-us;English (United States)
UICulture               : en-us;English (United States)
TimeZone                : (UTC+08:00) Beijing, Chongqing, Hong Kong, Urumqi
PhysicalMemory          : 4,095 MB
AvailablePhysicalMemory : 2,482 MB
MaxVirtualMemory        : 8,189 MB
AvailableVirtualMemory  : 6,532 MB
UsedVirtualMemory       : 1,657 MB
PagingFile              : C:\pagefile.sys
Domain                  : WORKGROUP
LogonServer             : \\WIN-RGSKMLQ9ID2
Hotfix                  : 2 Hotfix(s) Installed.,[01]: KB2534111,[02]: KB976902
NetworkAdapter          : 2 NIC(s) Installed.,[01]: Intel(R) PRO/1000 MT Network Connection,      Connection Name: Loca
                          l Area Connection,      DHCP Enabled:    Yes,      DHCP Server:     172.16.74.254,      IP ad
                          dress(es),      [01]: 172.16.74.143,      [02]: fe80::3591:8ee4:d1a2:a74f,[02]: Bluetooth Dev
                          ice (Personal Area Network),      Connection Name: Bluetooth Network Connection,      Status:
                                    Media disconnected
{% endhighlight %}

到此通过Powershell+systeminfo获取操作系统信息就完成了。很简单吧，真是解放生产力的好工具。

## 领略powershell魔法
由于powershell和.Net结合非常紧密，两者关系类似于JAVA和groovy关系。接着让来体验下powershell魔力。
客到先上菜：

{% hightlight csharp %}
	String srciptText = @"$header = 'HostName','OSName','OSVersion','OSManufacturer','OSConfig','Buildtype',`
	'RegisteredOwner','RegisteredOrganization','ProductID','InstallDate','StartTime','Manufacturer',`
	'Model','Type','Processor','BIOSVersion','WindowsFolder','SystemFolder','StartDevice','Culture',`
	'UICulture','TimeZone','PhysicalMemory','AvailablePhysicalMemory','MaxVirtualMemory',`
	'AvailableVirtualMemory','UsedVirtualMemory','PagingFile','Domain','LogonServer','Hotfix',`
	'NetworkAdapter'
	systeminfo.exe /FO CSV |
	  Select-Object -Skip 1 |
	  ConvertFrom-CSV -Header $header";
    Runspace runspace = RunspaceFactory.CreateRunspace();  
    // open it  
    runspace.Open();  
    // create a pipeline and feed it the script text  
    Pipeline pipeline = runspace.CreatePipeline();
    pipeline.Commands.AddScript(srciptText);  
    //pipeline.Commands.Add("Out-String");  

    // execute the script  
    Collection<PSObject> results = pipeline.Invoke();  
    // close the runspace  
    runspace.Close();  

    // convert the script result into a single string  
    StringBuilder stringBuilder = new StringBuilder();  
    foreach (PSObject obj in results)
    {
        Console.WriteLine(obj.Members["HostName"].Value);
        Console.WriteLine(obj.Members["OSName"].Value);
        Console.WriteLine(obj.Members["OSVersion"].Value);
        Console.WriteLine(obj.Members["OSManufacturer"].Value);
    }
{% endhighlight %}
从上面代码可以看出在C#中获取每一个域的值是多麽简单，只需要：
> obj.Members["HostName"].Value
就获取出来了，完全不需要自己动手解析。这也是优于groovy的地方。