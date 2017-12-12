title: 记一次坑爹的Gitlab数据迁移之旅
date: 2017-12-12 11:45:05
tags:
  - CentOS
  - Docker
  - Gitlab
categories: 运维
---
昨晚加班到两点，为了完成公司开发环境的Gitlab迁移，以便在从今天开始所有人都能在新的Gitlab上工作。迁移的原因是之前错误的估计了Gitlab的系统资源占用率，在一台比较繁忙的服务器上搭建了Gitlab，服务器配置只有单核4G内存，还运行了Jenkins和Nginx等其他服务，导致Gitlab在繁忙时经常响应缓慢或无响应，因此有了这次迁移之旅，期间踩坑无数，不过也学到了很多关于CentOS和Docker的知识。
<!--more-->