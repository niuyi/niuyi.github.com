---
layout: post
title: "在Win7下使用Octopress打造自己的blog"
date: 2012-04-05 20:40
comments: true
categories: Tools
---

Octopress是个基于Jekyll的Blog框架，非常的简单好用。特别的是，在Octopress的网站上明确说明：“Octopress is a blogging framework for hackers”，这是给热爱折腾的人准备的，如果你不喜欢折腾，千万别碰这个东西！
<!-- more -->
<h4>Step 1：安装Ruby环境</h4>

Octopress是基于Ruby的，所以必须要在机器上安装Ruby的环境。如果是Linux或Mac，安装要容易的多，可以参考Octopress官方网站上的说明。这里着重介绍下在Win7下的安装方法：
在Win7下安装需要下载这两个东东：rubyinstaller和DevKit。Ruby的版本必须是1.9.2以上版本。需要将ruby的主目录（如D:\Ruby193）添加到Windows环境变量Path里。
安装完成后，可以在命令行里运行如下命令测试：
```
ruby --version  # Should report Ruby 1.9.2
```
<h4>Step 2: 安装Python环境</h4>
为什么还要Python？Octopress里面的代码语法高亮的插件需要Python的支持。代码语法高亮也是技术blog的最大亮点，所以一定要安装。这里有个非常tricky的地方，我的环境是Win7 64位，但是下载Python64位的安装包之后总有问题，后来换成32位的安装包：<a href="http://www.python.org/ftp/python/2.7.2/python-2.7.2.msi">python-2.7.2.msi</a>就OK了。具体可以参考这2个讨论：<a href="https://github.com/imathis/octopress/issues/262">issue 1</a>，<a href="https://github.com/github/gollum/issues/225">issue 2</a>

<h4>Step 3：Setup Octopress</h4>

这步非常简单，参照官网上的说明即可：
Setup Octopress
```
git clone git://github.com/imathis/octopress.git octopress
cd octopress
```
安装依赖
```
gem install bundler
bundle install
```
安装默认主题
```
rake install
```

<h4>Step 4: Deploy Octopress</h4>
可以使用Github的<a href="pages.github.com">Pages service</a>来host自己的blog。
现在github上创建一个新的Repository（当然，你需要在github上注册），Project Name是username.github.com（比如abc.github.com），这样你的blog默认地址就是abc.github.com。

然后在命令行里执行：
```
rake setup_github_pages
```
命令行会提示输入你repo的read/write的地址，正确输入后octopress会替你完成剩余的操作。
接着执行如下命令完成deploy：
```
rake generate
rake deploy
```
最后提交source代码
```
git add .
git commit -m 'your message'
git push origin source
```

<h4>Step 5: 开始写Blog</h4>
现在终于可以开始写blog了，在命令行里执行如下操作开始一个新的文章：
```
rake new_post["title"]
```
在source/_posts下会生成一个以markdown为后缀名的文件，这个就是你的文章的内容。可以编辑这个文件来写blog。
在写的过程中可以通过如下命令实时查看效果(http://localhost:4000/)：
```
rake generate   # Generates posts and pages into the public directory
rake watch      # Watches source/ and sass/ for changes and regenerates
rake preview
```

<h4>中文问题</h4>
在Win7下，Octopress的中文经常有问题。网上有很多解决方案，我的方案是在文本编辑器里选择UTF-8无BOM格式编码，就可以支持中文了。
<h4>侧边栏</h4>
Octopress中可以添加自定义的侧边栏，在source/includes/custom/asides/里已经有个about.html，可以编辑这个文件添加个人介绍。然后在config.yml里的default_asides:加上custom/asides/about.html就可以在侧边栏里显示出来。

<h4>评论</h4>
评论是基于disqus提供的服务。先在disqus上注册，然后在_config.yml里找到相关部分添加上内容既可：
```
# Disqus Comments
disqus_short_name: 
disqus_show_comment_count: true
```