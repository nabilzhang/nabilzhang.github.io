---
layout: post
title:  "php配置的error_reportting和short_open_tag"
date:   2011-12-26 20:39:09
categories: php
---
最近在做网站的迁移的时候出现了一些问题，花了好多时间来解决，写写记录下。

###1.short_open_tag

php的代码一般在php文件中是包含下面这样子的标签内的
{% highlight php %}
<?php

...
.......

?>
{% endhighlight %}

但是改的这个网站它在一些使用了没有php而是简单的标签，叫做short_open_tag
{% highlight php %}
<?  ... ?>
{% endhighlight %}

所以在运行的时候会直接将代码显示在了页面上。一直没有发现php竟然是这么写的，找了好久才查出是这个原因，解决方法是在php.ini中改了short_open_tag为打开状态（将Off改为On）。
{% highlight php %}
short_open_tag = On
{% endhighlight %}
然后网站正常了，但是里面竟然有些变量报错误。本来在以前环境正常的怎么发现在这里不对。就是下个问题了。

###2.error_reporting
php在使用未定义的变量时会报错，虽然说是可以未定义就使用变量，但是还是觉得先定义后使用会比较妥当一点。因为程序一直报这个错误，所以就想还是php配置的问题了。

于是后来找到php.ini里有error_reporting的配置然后修改了下就可以了，不让再提示错误。
{% highlight php %}
;;error_reporting = E_ALL & ~E_NOTICE
//去掉前面的分号改为下面这样
;error_reporting = 
//或者在程序网站程序的前面加上
error_reporting(0);
//就可以了
{% endhighlight %}

