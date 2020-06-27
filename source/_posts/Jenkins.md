---
title: "Jenkins平台介绍"
date: 2018-10-20 9:21:03 +0800
comments: true
tags:
- Jenkins
categories:
- [Jenkins]
---

介绍下Jenkins平台，平时经常都会用到

<!-- more -->

###Jenkins介绍

一款由Java编写的持续集成工具

<https://jenkins.io/zh/>

<https://zh.wikipedia.org/wiki/Jenkins_(%E8%BD%AF%E4%BB%B6)>

###Jenkins安装

当前用户下，用war方式安装

下载war包文件 (https://jenkins.io/zh/doc/book/installing/  -> WAR文件 -> http://mirrors.jenkins.io/war-stable/latest/jenkins.war)

！！注意，千万不要用dmg、pkg等图形化方式安装，这会导致生成一个Jenkins用户，和当前用户并列，这个Jenkins用户权限很小，很多文件访问不了，其内部也是通过 java -jar jenkins.war —httpPort=9000方式开启的Jenkins服务,可参考library/Application Support/Jenkins/jenkins-runner.sh，这种安装方式坑太多

###Jenkins服务启动

java -jar /Users/liu/desktop/jenkins.war --httpPort=9000

###Jenkins配置、使用

![](/images/jenkins-1.png)

配置过程中，支持execute shell脚本、支持文件夹分组、支持参数化构建

###Jenkins支持设置远程节点

因为iOS打包只能使用Mac电脑，所以一般电脑都是放公司，在内网下才能访问，没有固定IP，也没有对外域名，很不方便，导致Jenkins经常访问不了，所以需要 通过反向代理方式，将此Mac电脑的Jenkins服务可以通过外网的Jenkins访问到

一个远程可以外网访问的Jenkins，一个联网但只有内网IP的Mac Jenkins，需要把后者 通过节点的形式 连到 外网Jenkins上，也就是 内网的Jenkins作为 外网Jenkins的 slave，外网Jenkins 是 内网Jenkins的 agent

1. master Jenkins 新建 支持 slave Jenkins的 节点

![](/images/jenkins-2.png)

2. 在 slave Jenkins 上 运行脚本文件

java -jar agent.jar -jnlpUrl http://ip:8080/computer/rdmac/slave-agent.jnlp // ip 为 master jenkins 的IP

脚本运行正常后，代表 节点开启成功，可以去 master Jenkins的 节点管理里面查看 节点的状态是否 agent is connected

3. 在 master Jenkins 上 新建 item，该配置的都配置下

Restrict where this project can be run 选项，打对勾，label expression 里面 写 节点的 label

至此，远程节点配置完成



