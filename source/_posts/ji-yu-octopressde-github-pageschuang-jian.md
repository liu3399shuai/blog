---
layout: post
title: "基于octopress的github-pages创建"
date: 2016-02-15 21:56:10 +0800
comments: true
categories: 
---

具体搭建博客的步骤参考这些链接

http://sherlockyao.com/blog/2014/05/23/hello-blogging/

<!-- more -->

### 具体搭建博客的步骤参考这些链接

http://sherlockyao.com/blog/2014/05/23/hello-blogging/

http://shengmingzhiqing.com/blog/setup-octopress-with-github-pages.html/

http://www.pchou.info/web-build/2014/07/04/build-github-blog-page-08.html

http://yinkang.me/archives/194

http://sumnous.github.io/blog/2014/05/11/octopress-build-blog-on-github/

http://kejiwen.com/blog/2014/12/07/build-octopress-blog-record/

http://jekyllthemes.org/
https://github.com/imathis/octopress/wiki/3rd-Party-Octopress-Themes

http://cnbin.github.io/blog/2015/05/18/octopressbo-ke-da-jian/

http://stackoverflow.com/questions/19619280/octopress-pushing-error-to-github

### 发博文常用步骤

```bash
cd octopress
rake setup_github_pages
```
输入要链接的URL ,就是github上面仓库为XXX.github.io里面的URL,我的是https://github.com/liu3399shuai/liu3399shuai.github.io.git

![](/images/github_url.png)

新建博文

```bash
rake new_post["输入博文的title"]
```
完了后去source/_post里面就可以看到了

打开博文，使用markdown软件编辑，[macDown](http://macdown.uranusjr.com/) 或者 [Mou](http://25.io/mou/)都可以,写完后去生成网页，发布

```bash
rake generate
rake deploy
或者两个命令连城一个 rake gen_deploy
```
这个是发布的，发布成功后，隔几秒，刷新下github-page主页

`不要忘了`，保留更改的source文件

```bash
git commit -m'msg'
git push origin source
```

### 注意问题

在走到bundle install 这一步时候，经常失败，比如原因有如下这个

```bash
Don't run Bundler as root. Bundler can ask for sudo if it is needed, and
installing your bundle as root will break this application for all non-root
users on this machine.
Fetching gem metadata from https://rubygems.org/...........
Fetching version metadata from https://rubygems.org/...
Fetching dependency metadata from https://rubygems.org/..
Resolving dependencies...
Using rake 10.4.2
Using RedCloth 4.2.9
Using blankslate 2.1.2.4
Using hitimes 1.2.2
Using timers 4.0.1
Using celluloid 0.16.0
Using chunky_png 1.3.4
Using fast-stemmer 1.0.2
Using classifier-reborn 2.0.3
Using coffee-script-source 1.9.1.1
Using execjs 2.5.2
Using coffee-script 2.4.1
Using colorator 0.1
Using multi_json 1.11.2
Using sass 3.4.15
Using compass-core 1.0.3
Using compass-import-once 1.0.5
Using rb-fsevent 0.9.5
Using ffi 1.9.10

Gem::RemoteFetcher::FetchError: Errno::ECONNRESET: Connection reset by peer - SSL_connect (https://rubygems.org/gems/rb-inotify-0.9.5.gem)
An error occurred while installing rb-inotify (0.9.5), and Bundler cannot
continue.
Make sure that `gem install rb-inotify -v '0.9.5'` succeeds before bundling.
```

因为国内被墙的原因，解决方案 :在执行目录下得到一个Gemfile文件，用文本编辑器打开，修改为

```bash
source "https://ruby.taobao.org"
```

如果在使用 rake deploy rake gen_deploy 时候报出这样的错误，需要同步一下

```bash
Pushing generated _deploy website
To https://github.com/liu3399shuai/liu3399shuai.github.io.git
 ! [rejected]        master -> master (non-fast-forward)
error: failed to push some refs to 'https://github.com/liu3399shuai/liu3399shuai.github.io.git'
hint: Updates were rejected because the tip of your current branch is behind
hint: its remote counterpart. Integrate the remote changes (e.g.
hint: 'git pull ...') before pushing again.
hint: See the 'Note about fast-forwards' in 'git push --help' for details.

Github Pages deploy complete
```

同步命令

```bash
cd _deploy
git reset --hard origin/master
cd ..
```

联系我 <liu3399shuai@163.com>

或者我的网站 <https://github.com>
