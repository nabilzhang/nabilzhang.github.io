---
layout: post
title:  "Rsync自动备份配置"
date:   2015-12-17 23:37:55
categories: Backsh
---

###1.备份服务器配置
安装Rsync
rsync工作目录：/home/work/rsync
{% highlight bash %}
mkdir /home/work/rsync
cd /home/work/rsync

mkdir run
mkdir log
{% endhighlight %}

####1.1新建rsync.conf
{% highlight bash %}
uid = nobody
gid = nobody
use chroot = no
pid file = /home/work/rsync/run/rsyncd.pid
lock file = /home/work/rsync/run/rsync.lock
log file = /home/work/rsync/log/rsyncd.log
[nfs]
ignore errors
read only = false
list = false
path = /home/work/nfs
auth users = work
secrets file = /home/work/rsync/rsyncd.secrets
{% endhighlight %}

以上配置中模块nfs是将文件备份到/home/work/nfs

####1.2新建rsyncd.secrets
{% highlight bash %}
work:workpass
{% endhighlight %}

####1.3启动备份服务器
{% highlight bash %}
rsync --daemon --config=/home/work/rsync/rsync.conf --port=8002
{% endhighlight %}

ps -ef | grep rsync
可以检查是否启动成功。
如果有core，很有可能是最前面run和log目录不存在导致。


###2客户端配置
####2.1备份密码配置
{% highlight bash %}
vim rsync.pass
#内容：
workpass
{% endhighlight %}

chmod 600 rsync.pass
因为密码文件只能让自己看见

####2.2备份文件
{% highlight bash %}
###从服务器备份到客户端
rsync -vzrtopg --delete --exclude "logs/" --exclude "rsync.pass" --progress work@x.x.x.x::nfs /home/work/nfs --password-file=/home/work/nfs/rsync.pass --port=8002

###从客户端备份到备份服务器
rsync -vzrtopg --delete --exclude "logs/" --exclude "rsync.pass" --progress /home/work/nfs work@x.x.x.x::nfs --password-file=/home/work/nfs/rsync.pass --port=8002
{% endhighlight %}


####2.3自动备份
配置crontab任务，每分钟一次备份
{% highlight bash %}
###从服务器备份到客户端
*/1 * * * *     rsync -vzrtopg --delete --exclude "logs/" --exclude "rsync.pass" --progress work@x.x.x.x::nfs /home/work/nfs --password-file=/home/work/nfs/rsync.pass --port=8002

###从客户端备份到备份服务器
*/1 * * * *     rsync -vzrtopg --delete --exclude "logs/" --exclude "rsync.pass" --progress /home/work/nfs work@x.x.x.x::nfs --password-file=/home/work/nfs/rsync.pass --port=8002
{% endhighlight %}

