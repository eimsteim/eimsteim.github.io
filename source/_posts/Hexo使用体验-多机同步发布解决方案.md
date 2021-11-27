---
title: Hexo使用体验-多机同步发布解决方案
tags:
  - Hexo
  - GitHub
categories: 技术
abbrlink: 55953
date: 2016-07-03 17:05:08
---
自从使用了Hexo作为主力博客平台，已经有半年左右的时间，虽然由于工作关系，其间没有更新多少博文，不过也没想过切换到其他平台。相比于GitHub官方推荐的Jekyll，我觉得Hexo更简单快速，符合写作的初心。
然而Hexo在多机同步发布方面，存在天然的缺陷，本文将和大家一起讨论该缺陷的解决方案。
<!--more-->
> 首先，什么是多机同步发布？

很简单，比如你在公司电脑上写了一篇文章，然而回家后，用的是另外一台电脑。怎样保证两台电脑上的博客源代码是相同的？

你一定会说：这还不简单，同步到GitHub不就行了吗？

然而很可惜，如果用GitHub发布Hexo，那么被同步的只能是源代码中`hexo g`出来的静态发布文件。GitHub会去你的工程目录中寻找名为`master`分支中的`index.html`，并将其部署到Pages服务器。

问题出现了，难道我们要随身携带U盘，时时刻刻去拷贝Hexo源代码吗？

> 坊间的几种解决方案

主要是来自[知乎的讨论帖](https://www.zhihu.com/question/21193762)：


1. 自己将源代码备份到某个独立的版本管理工具，比如：在GitHub新建一个工程，或者在coding.net上开一个私有项目，还有的朋友直接备份到Dropbox（我也是醉了）。
2. 有位高手自己搭了个服务器，在上面写了一个很牛逼的系统，大概功能是自动打包和发布，并提供了邮件接口，只要你通过发邮件的方式访问它，它就能自动把邮件内容发布到GitHub。（对这位大哥，我只想说：你搞这么麻烦，为啥不直接用WordPress？）
3. 我认为是最优秀的解决方案，也是下面即将介绍的，在GitHub上通过新建分支的方式来管理Hexo源码。

> 第三种方案的优势

* 不需要新建GitHub项目
* 不依赖第三方版本管理工具
* 不需要另行申请服务器资源

> 具体步骤

1. 在你的github.io项目中新建一个branch，名字随意（我直接取名叫hexo）
2. 将该分支clone到本地，命令参考：`git clone -b hexo https://xxx.github.com/xxx/`

3. 将本地hexo分支目录中的所有文件(除了`.git`之外)全部删除（最好使用`git rm`命令，免得麻烦），删除完毕后别忘记提交到remote
4. 将hexo源码拷贝到该目录中，同样提交到remote

这样一来，你的该项目中master依然是发布后的文件，而hexo分支则变成了hexo源码，如果你需要在多台电脑上同时写作，只需要在写作之前`git pull`一下就可以了。

不过千万不要忘记的是：在写作完成之后，要同步到GitHub分支哦！命令参考：
`git push -u origin hexo`

> 最后，就是发布了

先说个题外话，话说Hexo本身有一个命令：`hexo deploy`

我之前并没有用到这个命令，还是用比较傻的办法，先 `hexo g`，然后将生成的文件`git commit`到GitHub。

后来发现这个命令是真好用，只需要执行`hexo d`，即可将编译后的代码自动发布到master了，当然前提是你正确地配置了deploy 信息。

> 要正确配置deploy信息，需要确认两件事情：

1. 首先需要检查你的本地git私钥和公钥是否存在，如果不存在，需要重新生成一个，并配置到GitHub。这里提供一个[官方教程](https://help.github.com/articles/generating-an-ssh-key/)。
2. 修改hexo配置：根目录下`_config.yml`，找到其中的
```
deploy:
  type: git
  repo: git@github.com:eimsteim/eimsteim.github.io.git
  branch: master
```
将repo修改成类似上面的格式，一个字儿都不能差。

OK，现在你可以愉快地`hexo d`了，Enjoy Writing :-)
