---
layout: post
title:  "jekyll简易实操手册"
date:   2018-09-10 08:14:12 +0800
categories: 技术 
---

这篇文章简介了jekyll的搭建, 以及如何修改theme, 及利用github发布博客站点的流程.
由于作者对jekyll主题的了解还不深入，其中修改主题的部分介绍的比较浅显，期待后续完善。 

### 创建website
遵循jekyll的标准步骤进行操作
```bash
gem install bundler jekyll
jekyll new example 
```
上一步会创建给bundle使用的Gemfile， 由于Gemfile里使用的是默认的rubygems源，这一步一般会像挂起了似的无法完成。
这里可以手动ctrl c终止执行流程，手动修改Gemfile里的国外rubygems源，切换为国内的rubygems源。

* source 'https://gems.ruby-china.com'
* gem 'rails', '4.1.0'

然后继续执行
```
cd example 
bundle install
bundle exec jekyll serve
```

一旦成功执行过bundle install，其他新建的jekyll blog site就不需要重复install，
但jekyll new 的时候还是会挂起。 
你可以配置bundle的rubygems源的镜像，以避免上述问题。
* bundle config mirror.https://rubygems.org https://gems.ruby-china.com


### 默认jekyll文件结构 
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
说明这些文件夹都是临时目录。当然你也可以把Gemfie*放在.gitignore中。



如果不需要定制主题，页面布局。那么你就可以使用markdown在_posts目录下写博客了。如果你对自己的网站外观有些其他的追求，那么我们就开始做手改造jekyll的主题吧。

### 定制jekyll主题 

看下默认的_config.yml，可以看到使用的主题是theme: minima。

首先我们从github上下载minima, 这个theme的源文件。可以不覆盖外层的文件。只拷贝
_include, _layouts, _sass, assets等几个目录。

修改 assets/main.scss
增加段落缩进的css代码
p{ text-indent:2em; padding:0px; margin:0px; }


修改 _layouts/post.html
修改你的blog页面格式。

### github提交

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
