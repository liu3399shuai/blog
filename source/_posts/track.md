---
title: "app无痕埋点"
date: 2020-1-11 20:31:00 +0800
comments: true
tags:
- 无痕埋点
categories:
- [无痕埋点]
---

介绍下app客户端实现的无痕埋点的实现方案

<!-- more -->

###Jenkins支持设置远程节点

因为iOS打包只能使用Mac电脑

1. master Jenkins 新建 支持 slave Jenkins的 节点

![](/images/jenkins-2.png)

2. 在 slave Jenkins 上 运行脚本文件

java -jar agent.jar -jnlpUrl http://ip:8080/computer/rdmac/slave-agent.jnlp // ip 为 master jenkins 的IP

脚本运行正常后，代表 节点开启成功，可以去 master Jenkins的 节点管理里面查看 节点的状态是否 agent is connected

3. 在 master Jenkins 上 新建 item，该配置的都配置下

Restrict where this project can be run 选项，打对勾，label expression 里面 写 节点的 label

至此，远程节点配置完成



