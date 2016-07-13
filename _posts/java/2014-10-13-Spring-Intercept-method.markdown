---
layout: post
title:  "Spring AutoProxy切面监控方法性能"
date:   2014-10-13 20:37:55
categories: Java
---

### 1.继承MethodInterceptor，创建Interptor,拦截方法前后记录下时间算出时间差，以下代码仅供参考

{% highlight java %}
package me.nabil.demo.autoproxydemo.interceptor;

import org.aopalliance.intercept.MethodInterceptor;
import org.aopalliance.intercept.MethodInvocation;
import org.apache.commons.logging.Log;
import org.apache.commons.logging.LogFactory;

/**
 * 
 * 
 * @author nabil
 * @date 2014年10月13日下午2:33:40
 */
public class ServiceInterceptor implements MethodInterceptor {

    private final static Log log = LogFactory.getLog(ServiceInterceptor.class);

    public Object invoke(MethodInvocation methodInvocation) throws Throwable {
        String className = "unknowClassName";
        String methodName = "unknowMethodName";
        Object[] methodParams = {};

        try {
            className = methodInvocation.getMethod().getDeclaringClass()
                    .getName();
            methodName = methodInvocation.getMethod().getName();
            methodParams = methodInvocation.getArguments();

        } catch (Throwable e) {
            log.warn("can not get class and method names", e);
        }

        System.out.println("start " + this + "," + className + "." + methodName
                + methodInvocation.getThis());
        long startTimestamp = System.currentTimeMillis();
        try {
            Object result = methodInvocation.proceed();
            System.out.println("end " + this + "," + className + "."
                    + methodName);
            long endTimestamp = System.currentTimeMillis();
            System.out.println(className + "." + methodName + "  cost"
                    + (endTimestamp - startTimestamp));
            return result;
        } catch (Throwable t) {

            throw t;
        }
    }

}

{% endhighlight %}

### 2.业务代码

{% highlight java %}
package me.nabil.demo.autoproxydemo.service.impl;

import me.nabil.demo.autoproxydemo.service.Inner1Service;
import me.nabil.demo.autoproxydemo.service.MessageService;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

/**
 * 
 * 
 * @author nabil
 * @date 2014年10月13日下午12:13:55
 */

@Service
public class MessageServiceImpl implements MessageService {

    @Autowired
    private Inner1Service inner1Service;

    public String getMessage() {
        inner1Service.say();

        System.out.println();

        return "hello" + this;
    }

}

{% endhighlight %}

### 3.main

{% highlight java %}

package me.nabil.demo.autoproxydemo;

import me.nabil.demo.autoproxydemo.service.MessagePrinter;

import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

/**
 * @author nabil
 * @date 2014年10月13日上午11:48:13
 */
public class Application {

    public static void main(String[] args) {
        ApplicationContext context = new ClassPathXmlApplicationContext(
                "applicationContext.xml");
        MessageService service = context.getBean(MessageService.class);
        service.getMessage();
    }
}

{% endhighlight %}


### 4.XML配置
{% highlight xml %}
<?xml version="1.0" encoding="UTF-8" ?>
<beans xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns="http://www.springframework.org/schema/beans" xmlns:aop="http://www.springframework.org/schema/aop"
	xmlns:context="http://www.springframework.org/schema/context"
	xmlns:jee="http://www.springframework.org/schema/jee" xmlns:tx="http://www.springframework.org/schema/tx"
	xmlns:jdbc="http://www.springframework.org/schema/jdbc"
	xsi:schemaLocation="
    http://www.springframework.org/schema/aop
    http://www.springframework.org/schema/aop/spring-aop.xsd
    http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans.xsd
    http://www.springframework.org/schema/context
    http://www.springframework.org/schema/context/spring-context.xsd
    http://www.springframework.org/schema/jee
    http://www.springframework.org/schema/jee/spring-jee.xsd
    http://www.springframework.org/schema/jdbc
    http://www.springframework.org/schema/jdbc/spring-jdbc.xsd
    http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx.xsd">
	<context:component-scan base-package=" me.nabil.demo.autoproxydemo" />
	
	<aop:aspectj-autoproxy proxy-target-class="true"/>
	
    <bean id="serviceLogInterceptor"
        class="me.nabil.demo.autoproxydemo.interceptor.ServiceInterceptor">
    </bean>

    <bean
        class="org.springframework.aop.framework.autoproxy.BeanNameAutoProxyCreator">
        <!-- 指定对满足哪些bean name的bean自动生成业务代理 -->
        <property name="beanNames">
            <list>
                <value>*ServiceImpl</value>
            </list>
        </property>
        <!-- 下面定义切面 -->
        <property name="interceptorNames">
            <list>
                <value>serviceLogInterceptor</value>
            </list>
        </property>
    </bean>
</beans>


{% endhighlight %}

