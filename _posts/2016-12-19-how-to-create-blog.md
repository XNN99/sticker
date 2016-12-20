---
layout: post
title: "博客(一) Jekyll博客开发环境搭建"
description: 这里介绍搭建自己博客开发环境需要什么工具？如何搭建开发环境？文章罗列搭建步骤。 
img: jekyll.jpg
color: BF360C
author: Carlwang
---

* some text
{: toc}

# 需要工具
- Jekyll 是一个简单的免费的Blog生成工具。
- Ruby Jekyll是``Ruby``的一个gem包，因此Ruby是必选项。
- Python 如果需要安装语法高亮软件Pygments，就需要``Python``开发环境。这是可选项。
		
# 准备环境: 
jekyll是``Ruby``的一个gem包,安装稍显复杂：

## 1.安装`Ruby`
[ruby下载地址](http://www.ruby-lang.org/en/downloads/)
通过下面命令检测Ruby是否安装成功，并添加到环境变量中。
{% highlight markdown %}
>ruby -v
ruby 2.2.2p95(2015-04-13 revision 50295)[x64-mingw32]
{% endhighlight %}
## 2.安装`Python`
[Python下载地址](https://www.python.org/downloads/)
通过下面命令检测Python是否安装成功，并添加到环境变量中。
{% highlight markdown %}
>python -V
Python 3.5.2
{% endhighlight %}

## 3.安装 ``Pygments``
{% highlight markdown %}
easy_install Pygments
{% endhighlight %}
安装完成后在网站的配置文件 _config.yml 里将 highlighter 的值设置为 pygments，即可。

## 4. 安装``Jekyll``
因为``Jekyll``是`gem`包，因此需要使用`gem`命令安装。
{% highlight markdown %}
gem install jekyll
{% endhighlight %}
到此``jekyll``开发环境已经搭建完成。可以尝试建立你的博客了。

## 5. 新建一个应用
新建一个目录作为你的博客工作目录。进入目录并执行下面命令：
{% highlight markdown %}
jekyll new myblog
{% endhighlight %}
其中myblog是博客站点名字。生成站点目录结构:
![blogdir]({{site.baseurl}}/images/blog-dir.png)

## 6. 启动站点
进入站点的目录，运行``jekyll serve``站点成功启动，默认端口``4000``:
{% highlight markdown %}
E:\workspace\Projects\myblog\sticker>jekyll serve
Configuration file: E:/workspace/Projects/myblog/sticker/_config.yml
Configuration file: E:/workspace/Projects/myblog/sticker/_config.yml
            Source: E:/workspace/Projects/myblog/sticker
       Destination: E:/workspace/Projects/myblog/sticker/_site
 Incremental build: disabled. Enable with --incremental
      Generating...
                    done in 1.708 seconds.
  Please add the following to your Gemfile to avoid polling for changes:
    gem 'wdm', '>= 0.1.0' if Gem.win_platform?
 Auto-regeneration: enabled for 'E:/workspace/Projects/myblog/sticker'
Configuration file: E:/workspace/Projects/myblog/sticker/_config.yml
    Server address: http://127.0.0.1:4000/
  Server running... press ctrl-c to stop.
{% endhighlight %}
这表示服务启动成功，然后可以在浏览器中输入http://127.0.0.1:4000/，欣赏你的杰作。
<blockquote>
如果启动jekyll服务时遇到少包的话，使用``gem install xxx``安装好后再启动`jekyll`服务，即可。
</blockquote>
