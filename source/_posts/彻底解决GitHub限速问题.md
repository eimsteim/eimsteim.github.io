---
title: 彻底解决GitHub限速问题
tags: GitHub
categories: 运维
abbrlink: 39560
date: 2021-11-27 21:44:37
---
众所周知，GitHub断供以后在国内访问GitHub就变得异常龟速，尤其是拉取代码时经常会报以下错误：
```
LibreSSL SSL_connect: SSL_ERROR_SYSCALL in connection to github.com:443
```
由于这个问题已经由来已久，很多热心网友发明了各式各样花里胡哨的解决方案，但其中许多都无法从根本上解决问题（包括购买昂贵的梯子）。今天，让我们来彻底解决这个问题。
<!--more-->

### 步骤一
打开https://github.com.ipaddress.com/ 如下图：

![步骤一](http://www.onefanr.com/static/blog/1519624-0f5808de571d51dc.jpeg)
把IP Address 记录下来！
把IP Address 记录下来！
把IP Address 记录下来！

### 步骤二
打开https://fastly.net.ipaddress.com/github.global.ssl.fastly.net#ipinfo 如下图：

![步骤一](http://www.onefanr.com/static/blog/1519624-a759d7f708a84c0f.png)
把IP Address 记录下来！
把IP Address 记录下来！
把IP Address 记录下来！

### 步骤三
打开https://github.com.ipaddress.com/assets-cdn.github.com 如下图：
![步骤一](http://www.onefanr.com/static/blog/1519624-34dbb48bc2ab1a27.png)
把IP Address 记录下来！
把IP Address 记录下来！
把IP Address 记录下来！

### 步骤四
修改电脑的hosts文件（Mac电脑可以使用ihosts，Win电脑可使用SwitchHost软件），把下列的东东写在最后，然后保存即可
```
140.82.113.4(图1的IP Address) github.com 
199.232.69.194(图2的IP Address) github.global.ssl.fastly.net
185.199.108.153(图3的IP Address)  assets-cdn.github.com
185.199.109.153(图3的IP Address)  assets-cdn.github.com
185.199.110.153(图3的IP Address)  assets-cdn.github.com
185.199.111.153(图3的IP Address)  assets-cdn.github.com
```

### 步骤五
在终端在输以下指令刷新DNS（需要权限）
```
sudo killall -HUP mDNSResponder;say DNS cache has been flushed
```

## 接下来就是见证奇迹的时刻！！！！！
PS：如果电脑是windows系统的话，执行与上面相同的步骤，最后通过ipconfig /flushdns命令刷新DNS缓存即可