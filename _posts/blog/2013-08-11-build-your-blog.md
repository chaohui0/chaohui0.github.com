---
layout: post
title: 走向Geek第一步，使用GitHub搭建blog
description: 使用GitHub是Geek必备的基本素质
category: blog
---

##一点废话

作为一个码农，如果没有一个自己的独立技术博客，好像显得不够专业。不过由于比较懒，一直只是在一些社区博客上零零碎碎贴点代码，直到几个星期前无意中发现我的同名域名没被注册，而且便宜的令人发指，果断抢注。然后寻寻觅觅，发现很多大牛都用Github写博客，于是就有了这个博客。

##还有些专业的废话

21世纪的码农如果不知道[Github][]，那真的是out了。它本身是不错的代码社区，而且还提供了诸如[Github Pages][]这些很实用的服务，本博客正是基于此搭建。

Github Pages有以下几个优点：

<ul>
    <li>轻量级的博客系统，没有麻烦的配置</li>
    <li>使用标记语言，比如<a href="http://markdown.tw">Markdown</a></li>
    <li>无需自己搭建服务器</li>
    <li>根据Github的限制，对应的每个站有300MB空间</li>
    <li>可以绑定自己的域名</li>
</ul>

当然他也有缺点：

* 使用[Jekyll][]模板系统，相当于静态页发布，适合博客，文档介绍等。
* 动态程序的部分相当局限，比如没有评论，不过还好我们有解决方案。
* 基于Git，很多东西需要动手，不像Wordpress有强大的后台

但是，能像写代码一样用vim写博客是不是很爽呢，而且Github提供了各种图形化操作工具，像博主这样的记不住各种vi，git命令的菜鸟们完全可以借助这些工具很方便的管理自己的博客。如果你有了Github的账号，并且有了自己的域名，那就开始动手搭建自己的blog吧。

##开始动手，十分钟搭建个人主页

google一下“github 搭建blog”，很多结果，但是很多介绍都是各种命令，各种开源工具，博主开始跟着这些大牛的思路，折腾了一天，结果还是个404页面。然后仔细研究下了GitHub Pages的介绍，发现完全不用各种命令，十分钟一个主页就搞定了。好吧，还是要先学走才能学飞啊，以下是菜鸟教程。

###1. 配置GitHub Pages
如果你的GitHub账户名为username，首先在Github中新建一个名为“username.github.com"的repositories，然后在Setting页面中可以看到一个“Automatic Page Generator”按钮，点击下就可以看到一个默认的介绍页面，选个模本就直接PUBLIS吧。然后在工程页面的顶部就能看到一行tips

	Your project page has been created at http://XXXX.github.com . It may take up to 10 minutes to activate. Read more at the Pages Guide.

那个就是你的blog地址拉，很简单有木有，赶紧用浏览器打开看下！

###2. 绑定个人域名
然后绑定下你的域名，也很简单，在工程中添加一个CNAME的文件，文件内容为你的域名，然后到你的域名服务商那修改下域名的CNAME跳转，等几分钟后，当你在浏览器中看到自己设计的页面是不是很有动力去学习各种git命令，哈哈。

###3. 选个钟意的模板，学下git命令
乔帮主讲过「Good artists copy; Great artist steal」，所以赶紧听从乔帮主的号召找个喜欢的blog模板copy吧。这个时候就发现学习git命令的好处了，所以还是要耐下心学习下几个最最基本的git命令，比如 git clone、git add、git rm、git commit、 git push等，画不鸟几分钟的，然后就可以像个hacker一样有模有样的在控制台窗口狂敲命令完成所有的事情了有木有，不过要先[配置ssh keys][]等。当然GitHub也是提供了客户端的，安装后使用也非常方便。

###4. 改出自己的style，了解Jekyll
敲完clone命令，当然还要改出自己的风格才行，所以首先要了解下Jeklly模板系统。Jekyll的核心其实就是一个文本的转换引擎，用你最喜欢的标记语言写文档，可以是Markdown、Textile或者HTML等等，再通过`layout`将文档拼装起来，根据你设置的URL规则来展现，这些都是通过严格的配置文件来定义，最终的产出就是web页面。

基本的Jekyll结构如下：

    |-- _config.yml
    |-- _includes
    |-- _layouts
    |   |-- default.html
    |   `-- post.html
    |-- _posts
    |   |-- 2007-10-29-why-every-programmer-should-play-nethack.textile
    |   `-- 2009-04-26-barcamp-boston-4-roundup.textile
    |-- _site
    `-- index.html


简单介绍一下他们的作用：
####_config.yml
配置文件，用来定义你想要的效果，设置之后就不用关心了。

####_includes
可以用来存放一些小的可复用的模块，方便通过`{ % include file.ext %}`（去掉前两个{中或者{与%中的空格，下同）灵活的调用。这条命令会调用_includes/file.ext文件。

####_layouts
这是模板文件存放的位置。模板需要通过[YAML front matter][9]来定义，后面会讲到，`{ { content }}`标记用来将数据插入到这些模板中来。

####_posts
你的动态内容，一般来说就是你的博客正文存放的文件夹。他的命名有严格的规定，必须是`2012-02-22-artical-title.MARKUP`这样的形式，MARKUP是你所使用标记语言的文件后缀名，根据_config.yml中设定的链接规则，可以根据你的文件名灵活调整，文章的日期和标记语言后缀与文章的标题的独立的。

####_site
这个是Jekyll生成的最终的文档，不用去关心。最好把他放在你的`.gitignore`文件中忽略它。

####其他文件夹
你可以创建任何的文件夹，在根目录下面也可以创建任何文件，假设你创建了`project`文件夹，下面有一个`github-pages.md`的文件，那么你就可以通过`yoursite.com/project/github-pages`访问的到，如果你是使用一级域名的话。文件后缀可以是`.html`或者`markdown`或者`textile`。这里还有很多的例子：[https://github.com/mojombo/jekyll/wiki/Sites](https://github.com/mojombo/jekyll/wiki/Sites)

具体懂点css和js的就在原blog的基础上慢慢雕琢修改吧。

###5. 添加评论系统

Jekyll只是个静态页面的发布系统，想做到关爽场倒是很容易，如果想要评论呢？也很简单。现在专做评论模块的产品有很多，比如[Disqus][]，还有国产的[多说][]。Disqus主要是用facebook等河蟹账号，所以国内用户使用并不方便，推荐多说，不过现在还没用，等后面有空了再研究下。

###6.一些问题

在具体使用的时候，可能会发生更新了日志，但是网页却没有更新的情况，这个一般是由于
语法问题造成jekyll编译不通过导致的,检查下最新文章中是否使用了一些非法字符，或者本地编译调试，具体方法参考链接：[https://help.github.com/articles/using-jekyll-with-pages#troubleshooting](https://help.github.com/articles/using-jekyll-with-pages#troubleshooting)

每次提交都要输入用户名密码很麻烦，在mac下git push的时候保存用户名密码：
	git config --global credential.helper osxkeychain

##结语

用vim写博客真的很有feel有木有，还有很多nb的诸如octopress的开源工具，大家有兴趣也可以研究下，本博首篇博文完结，我要git push啦，有任何问题请留言哦。


[Github]:   http://github.com "Github"
[Github Pages]: http://pages.github.com/ "Github Pages"
[Jekyll]:   https://github.com/mojombo/jekyll "Jekyll"
[我的blog]:  "我的blog"
[配置ssh keys]: http://www.google.com.hk/#bav=on.2,or.&fp=7b10337b43d4ef46&newwindow=1&q=github+ssh+key&safe=strict "配置ssh keys"
[Disqus]: http://disqus.com/
[多说]: http://duoshuo.com/


