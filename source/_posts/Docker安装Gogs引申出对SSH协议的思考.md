title: Docker安装Gogs引申出对SSH协议的思考
tags:
  - Docker
  - Gogs
  - SSH
date: 2017-12-09 02:08:19
categories: 运维
---

因为工作需要，在服务器上搭建了Gogs，一个轻量级的Git控制台管理工具。考虑到未来很有可能还要频繁移植，因此是基于Docker来安装，并通过--link的方式与另一个安装在Docker中的MySQL进行了桥连。其间可谓一波三折，不过也引发了对SSH协议的一些思考，分述如下。