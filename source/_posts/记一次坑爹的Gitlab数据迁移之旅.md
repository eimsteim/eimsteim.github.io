---
title: 记一次坑爹的Gitlab数据迁移之旅
tags:
  - CentOS
  - Docker
  - Gitlab
categories: 运维
abbrlink: 20976
date: 2017-12-12 11:45:05
---
昨晚加班到两点，为了完成公司开发环境的Gitlab迁移，以便在从今天开始所有人都能在新的Gitlab上工作。迁移的原因是之前错误的估计了Gitlab的系统资源占用率，在一台比较繁忙的服务器上搭建了Gitlab，服务器配置只有单核4G内存，还运行了Jenkins和Nginx等其他服务，导致Gitlab在繁忙时经常响应缓慢或无响应，因此有了这次迁移之旅，期间踩坑无数，不过也学到了很多关于CentOS和Docker的知识。
<!--more-->
其实最初我希望使用[Gogs](https://gogs.io/)来替代Gitlab，原因是测试环境服务器资源着实有限，单核4G的虚拟机，运行Gitlab大约要占1.5-2G内存，加上CentOS本身和其他应用的内存占用，空闲内存只剩下不到100MB，很多时候系统响应非常缓慢，严重影响了团队的开发效率。而Gogs由于不会默认内置Nginx和PostgreSQL等应用，所以启动后只会占用几百兆内存，差距还是非常明显的。但是在试用之后，我觉得Gogs还是不够成熟，例如：最新版本对MySQL版本要求过高（5.7），SSH支持不友好（甚至存在bug），权限设计模型（针对单个工程设置白名单，而不像Gitlab那样有明确的角色体系），以及整体用户体验综合考虑，所以最终还是弃用了。

下面是查看Gitlab的内存占用的方法：
```sh
#其中rsz是实际内存
ps -e -o 'pid,comm,pcpu,rsz,vsz,args,stime,user,uid' | grep gitlab > ps1.txt
#然后按空格分割，求第四列的和
awk -F ' ' '{sum += $4};END {print sum}' ps1.txt
```

![mark](http://7xo8xv.com1.z0.glb.clouddn.com/blog/180118/giGbBCHbAi.png?imageslim)

最初我还尝试使用Docker来安装Gogs，为此还升级了CentOS的系统内核，因为CentOS 6.9的内核是2.6.32，而运行Docker需要Linux内核版本至少在3.1以上。内核升级完成后，再运行docker就很顺利了。

> 注意：在安装Docker的过程中，还遇到了没有对应版本的问题，因此需要先安装ELrepo，一个免费的yum源。需要注意的是CentOS 6.8和7.x之间的差异很大，在安装elrepo的时候，需要选择正确的版本，下载地址可以在这里找到：http://elrepo.org/tiki/tiki-index.php

另外，在Docker中安装Gogs遇到的主要问题有两个：
1. ssh链接无法正确授权的问题，尝试了很多方法，还是无法解决，主要原因是端口和文件系统的映射，导致ssh授权异常，最终无奈地弃用了Gogs。
2. MySQL地址无法访问，这个最后通过联合启动的方式解决了，联合启动的命令如下：

```sh
docker run --link=mysql5.7:mysql -p 10022:22 -p 10080:3000 -v /var/gogs:/data gogs/gogs
```

言归正传，要进行Gitlab的迁移，其实还是很方便的，因为Gitlab提供了自动化的工具对数据进行打包，但需要注意的是，两边的Gitlab需要保证版本是一致的，否则在导入数据的时候会报错。那么，如果像我一样已经安装了错误版本的Gitlab，怎么办呢？首先，我们需要卸载掉错误版本的Gitlab，步骤如下：

```sh
#停止gitlab
gitlab-ctl stop
#卸载gitlab（注意这里写的是gitlab-ce）
rpm -e gitlab-ce
#查看gitlab进程
ps aux | grep gitlab
#杀掉第一个进程（就是带有好多.............的进程）
kill -9 18777
#杀掉后，在ps aux | grep gitlab确认一遍，还有没有gitlab的进程
#删除所有包含gitlab文件
find / -name gitlab | xargs rm -rf
```

接下来，我们看一下源Gitlab的版本号：
```sh
#查看当前gitlab版本号：
head -1 /opt/gitlab/version-manifest.txt
```

然后，在这里可以找到比较全的Gitlab版本：https://mirrors.tuna.tsinghua.edu.cn/gitlab-ce/yum/el6/

Gitlab备份：
```sh
gitlab-rake gitlab:backup:create
```

使用以上命令会在/var/opt/gitlab/backups目录下创建一个名称类似为1481598919_gitlab_backup.tar的压缩包, 这个压缩包就是Gitlab整个的完整部分, 其中开头的1481598919是备份创建的日期 
/etc/gitlab/gitlab.rb 配置文件须备份 
/var/opt/gitlab/nginx/conf nginx配置文件 
/etc/postfix/main.cfpostfix 邮件配置备份


Gitlab恢复（需要先把上述备份文件拷贝到新的服务器上）：

```sh
# 停止相关数据连接服务
gitlab-ctl stop unicorn
gitlab-ctl stop sidekiq
# 从1481598919编号备份中恢复
gitlab-rake gitlab:backup:restore BACKUP=1481598919
# 启动Gitlab
sudo gitlab-ctl start

# 修改root密码/修改普通用户密码也适用
sudo gitlab-rails console production
user = User.find_by(email: 'admin@local.host')
user.password = 'secret_pass'
user.password_confirmation = 'secret_pass'
user.save
```

![mark](http://7xo8xv.com1.z0.glb.clouddn.com/blog/180119/LcgGI85bCJ.png?imageslim)

>  **后记** ：
> 搭建完成后，夏老师发现一个问题：Git发生了新的提交后，Jenkins构建无法发现这些更新，具体表现是在“变更记录”中一直显示No changes，解决这个问题可以尝试两种方法：
1）gitlab安装时会要求指定访问的域名，这里既可以指定域名，也可以指定IP，如果和我这次一样指定的是域名，则在Jenkins的服务器上也必须配置这样一个hosts。
2）Jenkins在版本库发生变更后的第一次构建是不会触发变更记录的，再构建一次即可。

参考资料：
[Gitlab备份与恢复、迁移与升级](http://www.xuliangwei.com/xubusi/803.html)
[gitlab之：502解决](http://www.nideyuan.com/?p=319)
