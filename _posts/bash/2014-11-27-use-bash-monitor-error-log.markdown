---
layout: post
title:  "Shell Tail监控程序日志并发送邮件"
date:   2014-11-27 23:37:55
categories: Bash
---

场景：程序在运行过程中对于某些错误日志发生时期望能够及时知晓开发人员，同时将同一请求中的所有日志一并发送邮件给开发人员，便于快速定位问题。

1.假设日志格式如下，在《LogBack-Sl4j-MDC区分不同请求日志》一文中有如下类似日志，最后一行中有ERROR日志，而最后两行是统一请求的日志，期望同时被发出来。
{% highlight bash %}
e6668b68-ee47-4cde-b673-25ed9bb74f1e 2014-04-09 16:31:14,579 [qtp14850080-27] INFO  o.c.a.f.i.AuthInterceptor - GET:/project
c3b84462-81c6-49f7-923b-d8ba48c53c07 2014-04-09 16:31:31,295 [qtp14850080-27] INFO  o.c.a.f.i.AuthInterceptor - GET:/feedbacks
b0e0f1fe-f30a-42b2-a103-e70a108561b7 2014-04-09 16:31:32,254 [qtp14850080-22] INFO  o.c.a.f.i.AuthInterceptor - GET:/feedbacks
a58ed619-c2e0-4b71-9c45-995a78c1b602 2014-04-09 16:31:42,788 [qtp14850080-29] INFO  o.c.a.f.i.AuthInterceptor - GET:/project
70de174a-5a05-41c6-a8b4-f17fcd33be5c 2014-04-09 16:31:43,537 [qtp14850080-22] INFO  o.c.a.f.i.AuthInterceptor - GET:/project
f3efb2d4-f361-4c6b-bab6-16c8476c8dd0 2014-04-09 16:31:47,157 [qtp14850080-26] INFO  o.c.a.f.i.AuthInterceptor - POST:/project
f3efb2d4-f361-4c6b-bab6-16c8476c8dd0 2014-04-09 16:31:47,161 [qtp14850080-26] ERROR  o.c.a.f.i.ProjectController - >> create() > GOT PARAMS 'Project [name=url, abstractContent=a, token=null]','(POST /project)@12571381 org.eclipse.jetty.server.Request@bfd2f5'
{% endhighlight %}

2.基于以上日志格式，实现如下Bash进行日志监控
{% highlight bash %}
#!/bin/bash
#日志路径
LOGFILE="test.log"
#错误信息关键字
ERROR_TAG="ERROR"
#邮件收件人
MAILLIST="user@sample.com"

#根据RequestId查询上下文日志，发送邮件
findByRequestIdAndSendEmail()
{
        #echo $1
        grep "$1" $LOGFILE | cat | /bin/mail -s "[LOG_MONITOR]-[ERROR]" $MAILLIST
}

#tail 命令监控每行日志
tail -f $LOGFILE | while read new_log
do
        #echo $new_log
        if echo "$new_log" | grep -q $ERROR_TAG;then
                #截取RequestId
				request_id=${new_log:0:36}
                findByRequestIdAndSendEmail $request_id
        fi
done
{% endhighlight %}

