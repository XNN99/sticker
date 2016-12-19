---
layout: post
title: " 使用Jekyll+Markdown创建博客站点"
description: 这里介绍如何通过Jekyll和Markdown工具建立自己博客站点，文章只简要介绍博客搭建步骤。 
img: jekyll.jpg
color: BF360C
author: Carlwang
---

* some text
{: toc}

# 需要工具
- Jekyll 是一个简单的免费的Blog生成工具。
- MarkdownPad Windows下最好的写作利器。
		
# 准备环境: 
jekyll是``Ruby``的一个gem包,安装稍显复杂：

1. 下载`Ruby`。[ruby下载地址](http://www.baidu.com)
2. 安装`Ruby`，通过下面命令检测Ruby是否安装成功，并添加到环境变量中。

```python
 
ruby -v

ruby 2.2.2p95(2015-04-13 revision 50295)[x64-mingw32]
```

3. 



当``Ruby``
Fork the ``master`` branch and delete ``gh-pages`` branch in it. This is important because ``gh-pages`` branch is used here only to host the blog. You should be using the master branch as the source and create a fresh ``gh-pages`` branch.

Watch my video on instlallation
<iframe class="video" src="https://www.youtube.com/embed/T2nx6tj-ZH4?rel=0?rel=0" frameborder="0" allowfullscreen></iframe>

## How to delete old **gh-pages** branch?
After forking the repository, click on **branches**.

![delete gh-pages branch](http://blog.webjeda.com/images/delete-github-branch.png)

Delete ``gh-pages`` branch.
![delete gh-pages branch](http://blog.webjeda.com/images/delete-github-branch-2.png)

You have to create a new ``gh-pages`` branch using the master branch. Go back to the forked repository and create ``gh-pages`` branch.

![create gh-pages branch](http://blog.webjeda.com/images/create-gh-pages-branch.JPG)

Now, go to settings and check the **Github Pages** section. You should see a URL where the blog is hosted.

This process will host the theme as a **Project Page**. You can also download the files for local development. 

The default theme will look like this

![webjeda karna jekyll theme]({{site.baseurl}}/images/karna-jekyll-theme-screenshot.png){: style="box-shadow: 0 0 5px #777"}



A sample post would look like this

![webjeda karna jekyll theme sidebar]({{site.baseurl}}/images/post.png){: style="box-shadow: 0 0 5px #777"}


This theme is made especially for smartphones.

![webjeda karna responsive jekyll theme]({{site.baseurl}}/images/karna-responsive-jekyll-theme.png){: style="box-shadow: 0 0 5px #777"}
{: style="text-align:center"}

# Development
Make changes to the **master** branch and create a pull request. Do not use **gh-pages** branch as it is used to host the theme.

# License
MIT License

# Change Log

### Version 0.8
* Initial release with a responsive card-friendly layout.