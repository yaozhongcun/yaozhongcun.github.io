---
layout: post
title:  "jekyll简易实操手册 & git基本使用"
date:   2018-09-10 08:14:12 +0800
categories: 技术 
---

这篇文章简单介绍了如何利用jekyll搭建个人blog站点, 如何修改jekyll blog站点的theme（站点风格）, 以及如何利用github发布博客站点。
由于作者对jekyll theme的了解还不深入，其中修改theme的部分，介绍的比较浅显，期待后续完善。 

### 搭建本地jekyll blog站点
不考虑网络，代理的设置，搭建一个jekyll blog站点非常简单。首先使用gem安装bundler和jekyll，然后执行jekyll新建站点的命令。
```bash
gem install bundler jekyll
jekyll new example 
```
gem是管理ruby库和程序的命令，它使用https://rubygems.org作为源来查找，安装，升级ruby软件包。

由于一些未知原因，国内无法访问https://rubygems.org，但还好国内有该源的镜像。
截至目前（2018-10），https://gems.ruby-china.com这个镜像是可用的。
我们可以使用如下命令设置镜像源:
```bash
gem sources --remove https://rubygems.org/
gem sources --add https://gems.ruby-china.com/
gem sources --list
```

jekyll new example 这一步
会创建给bundle使用的Gemfile， 由于Gemfile里使用的也是默认的rubygems源，由于网络的原因，这一步一般会像挂起了似的无法完成。
这里可以手动ctrl c终止执行流程，手动修改Gemfile里的国外rubygems源，切换为国内的rubygems源。

* source 'https://gems.ruby-china.com'
* gem 'rails', '4.1.0'

然后继续执行
```
cd example 
bundle install
```

一旦成功执行过bundle install，再次新建jekyll blog site就不需要再执行bundle install了，
但jekyll new 的时候还是会挂起。不需要重复执行，是因为所需的软件包已经安装成功；会挂起是因为新建站点的时候，bundle还是会去rubygems源去验证程序版本。
你可以配置bundle的rubygems默认源的镜像，就是前面已经介绍过的国内镜像源，以避免手动打断jekyll new的执行过程。
* bundle config mirror.https://rubygems.org https://gems.ruby-china.com

最后在新建的站点目录，运行如下命令，启动本地的http服务。
```
bundle exec jekyll serve
```
现在你可以通过浏览器，访问http://localhost:4000查看新创建的站点。
### 默认jekyll blog站点的目录结构 
新生成的jekyll blog site只包含如下文件
```
.gitignore 404.html Gemfile Gemfile.lock _config.yml about.md index.md
```

以及一个_posts目录，这个目录是给我们存放blog文件的。

运行server后，会产生_site, .sass-cache目录。

可以看下.gitignore内容：
```
_site
.sass-cache
.jekyll-metadata
```
说明这些文件夹和文件都是临时目录。当然你也可以把Gemfie*放在.gitignore中。后面我们会讲到，如何将这些文件提交到github。

如果不需要定制站点风格，页面布局。那么现在你就可以使用markdown在_posts目录下写博客了。
如果你对自己的网站外观有些追求，那么我们就开始做手改造jekyll的站点风格吧。

### 定制jekyll站点风格

默认的，jekyll站点使用_config.yml进行配置。查看该文件，默认使用的站点风格是theme: minima。

要改造theme，
首先我们要从github上下载minima这个theme的源文件。下载下来后，你会发现minama这个包拥有和我们创建的站点一样的目录结构。
解压后，可以不覆盖外层的文件，只拷贝
_include, _layouts, _sass, assets等几个目录。

站点的css样式，是在assets/main.scss文件中定义的。
我主要增加了段落缩进的css代码
p{ text-indent:2em; padding:0px; margin:0px; }

站点的页面表现代码在_layouts中。我主要修改了
_layouts/post.html，blog的页面代码，增加了打赏图片。

有很多开源的theme提供下载，你也可以下载你喜欢的theme代码，二次开发。

### github提交

你可以使用github的页面工作，或者任何工具，将你的站点提交到yourname.github.io.git项目，
就完成了你站点的发布。完成发布后，你就可以访问http://yourname.github.io看下效果了。

下面介绍的是，使用git命令行，利用ssh的方式进行提交。 
首先配置git命令行。
```
git config --global user.name "your.name"
git config --global user.email your.email@example.com
ssh-keygen -t rsa -C your.email
```
将公钥添加到你github账户的profile中。并使用如下命令，来验证是否之前的步骤是否执行成功。
```
ssh -T git@github.com
```
如果尚未在github创建个人jekyll站点。可以在刚才新建的jekyll目录下，执行如下git命令完成创建和文件推送。

```
git init
```
确认编辑好你的.gitignore
```
git add * 
git commit -m "init"
git remote add origin git@github.com:yourname/yourname.github.io.git
git push origin master
```
现在你就可以访问yourname.github.io看下效果了。
github.io理论上支持所有的静态网页元素，并且默认支持jekyll。



如果你已经将jekyll放到了github上，你可以使用如下git命令拉去下来进行操作。
```
git clone git@github.com:yourname/yourname.github.io.git
git add .gitignore  // add commit file
git commit -m "add ignore"
git push origin master
```


