title: 使用ssh-agent管理本地多个git密钥
date: 2020-05-10 20:54:02
tags:
  - Git
  - Github
  - ssh
categories: 运维
---
使用ssh-keygen生成ssh密钥的操作相信大家都很熟悉了。但是现在市面上有很多Git仓库，除了GitHub之外，还有[CODING](http://www.coding.net)、[码云Gitee](https://gitee.com)，以及公司的私有云代码仓库。我们不可能只使用其中一种，但是如果用最简单的方式去生成id_rsa，就意味着需要同时保存多个id_rsa。通过本文，你将学会如何通过ssh-agent管理多个id_rsa密钥。
<!--more-->
其实，和最简单的方式一样，也是使用ssh-keygen命令（以GitHub为例）：
```
ssh-keygen -t rsa -b 4096 -C "eimsteim@163.com" -f ~/.ssh/github_rsa
```
** 参数解释：**
+ -t ：加密协议
+ -b ：字节数
+ -C ：ssh-key的密钥名称（就是你配在Git仓库里的那个SSH Keys）
+ -f ：文件全路径

生成rsa文件以后，还需要把这个新生成的文件添加到ssh-agent中进行管理。在终端下执行命令：
```
ssh -v git@github.com
```
最后两句会出现
```
No more authentication methods to try.  
Permission denied (publickey).
```
接下来在终端再执行以下命令：
```
ssh-agent -s
```
最后执行：
```
ssh-add ~/.ssh/github_rsa
```
此时我们需要检查下，github_rsa.pub的内容有没有配置到Git仓库中，如果已经配置了，可以通过下面的命令验证：
```
ssh -T git@github.com
```

如果提示：Hi xxx! You've successfully authenticated, but GitHub does not provide shell access. 问题就解决啦