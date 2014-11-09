---
layout: post
title:  "在JavaWeb中使用Log4j步骤"
date:   2014-03-19 20:37:55
categories: Java
---

在JavaWeb中使用Log4J指南。每次在开始写一个项目的时候都忘记Log4J如何配置。所以写个步骤，作为记录。

###第一步 下载Log4J jar包

从[Apache Logging Services site](http://logging.apache.org/log4j/ "Apache Logging Services site")下载最新的Log4J的jar包。如果是使用maven2的话，可以直接在pom.xml加上如下依赖，maven将会自动进行下载。

{% highlight xml %}
<dependency>
    <groupId>log4j</groupId>
    <artifactId>log4j</artifactId> 
    <version>1.2.15</version> 
</dependency>
{% endhighlight %}

###第二步 将jar包导入到Web项目
如果使用Maven2可以忽略这一步，因为在pom.xml中声明了这一依赖，Maven将会在build的时候自动的将jar进行导入。
普通项目需要将下载下来的jar包复制到项目的WEB-INF/lib下。

###第三步 导入Log4J xml配置文件或者properties配置文件
对于Maven项目，直接将配置文件放置在<project>/Java Resources/src.main/resources目录下。

非Maven项目，需要将配置文件放置在classpath下。

properties实例：log4j.properties
{% highlight xml %}
log4j.rootLogger=INFO,stdout,logfile

log4j.appender.stdout=org.apache.log4j.ConsoleAppender 
log4j.appender.stdout.layout=org.apache.log4j.PatternLayout 
log4j.appender.stdout.layout.ConversionPattern=%d %p [%c] - <%m>%n

log4j.appender.logfile=org.apache.log4j.RollingFileAppender 
log4j.appender.logfile.File=test.log
log4j.appender.logfile.MaxFileSize=512KB

log4j.appender.logfile.MaxBackupIndex=5 
log4j.appender.logfile.layout=org.apache.log4j.PatternLayout 
log4j.appender.logfile.layout.ConversionPattern=%d %p [%c] - %m%n

{% endhighlight %}

Xml实例: log4j.xml
{% highlight xml %}
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE log4j:configuration SYSTEM "log4j.dtd" >
<log4j:configuration>
    <appender name="STDOUT" class="org.apache.log4j.ConsoleAppender">
        <layout class="org.apache.log4j.PatternLayout">
            <param name="ConversionPattern" value="%-5p [%c] %m %n" />
        </layout>
    </appender>
    <appender name="file" class="org.apache.log4j.RollingFileAppender">
        <param name="File" value="./test.log" />
        <param name="Append" value="true" />
        <param name="MaxFileSize" value="512KB" />
        <param name="MaxBackupIndex" value="5" />
        <layout class="org.apache.log4j.PatternLayout">
            <param name="ConversionPattern" value="[%d{ISO8601}] %-5p %m%n" />
        </layout>
    </appender>

    <logger name="com.sample" additivity="false">
        <level value="trace" />
        <appender-ref ref="file" />
        <appender-ref ref="STDOUT" />
    </logger>

    <root>
        <level value="INFO" />
        <appender-ref ref="file" />
        <appender-ref ref="STDOUT" />
    </root>

</log4j:configuration>

{% endhighlight %}

###第四步 写Log代码
前面一切都配置好的情况下就可以写Log代码了。

1.先导入Package
{% highlight java %}
import org.apache.log4j.Logger;
{% endhighlight %}

2.获取Logger类成员
{% highlight java %}
static Logger log = Logger.getLogger(MyClassName.class);
{% endhighlight %}

3.打Log
{% highlight java %}
log.debug("How are you today?");
log.info("I am fine.");
log.error("I am programming.");
log.warn("I love programming.");
log.fatal("I am now dead. I should have been a movie star.");    
{% endhighlight %}

###第五步 Run
启动Web app可以查看运行到相关代码就可以打出日志了。

【附】[GIT-HUB:https://github.com/nabilzhang/startup/tree/master/java.log4jdemo](https://github.com/nabilzhang/startup/tree/master/java.log4jdemo)