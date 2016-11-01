---
layout: post
title:  "Java HttpClient中的三种超时设置区别"
date:   2016-11-01 12:37:55
categories: Java
---

最近项目中在使用HttpClient有三个超时(connectionRequestTimeout,connectTimeout,socketTimeout)时间理解得不是很透彻，API文档不是很理解，专门写Demo理解了一下。

API的描述如下：

{% highlight java %}

/**
     * Returns the timeout in milliseconds used when requesting a connection
     * from the connection manager. A timeout value of zero is interpreted
     * as an infinite timeout.
     * <p>
     * A timeout value of zero is interpreted as an infinite timeout.
     * A negative value is interpreted as undefined (system default).
     * </p>
     * <p>
     * Default: {@code -1}
     * </p>
     */
    public int getConnectionRequestTimeout() {
        return connectionRequestTimeout;
    }

    /**
     * Determines the timeout in milliseconds until a connection is established.
     * A timeout value of zero is interpreted as an infinite timeout.
     * <p>
     * A timeout value of zero is interpreted as an infinite timeout.
     * A negative value is interpreted as undefined (system default).
     * </p>
     * <p>
     * Default: {@code -1}
     * </p>
     */
    public int getConnectTimeout() {
        return connectTimeout;
    }

    /**
     * Defines the socket timeout ({@code SO_TIMEOUT}) in milliseconds,
     * which is the timeout for waiting for data  or, put differently,
     * a maximum period inactivity between two consecutive data packets).
     * <p>
     * A timeout value of zero is interpreted as an infinite timeout.
     * A negative value is interpreted as undefined (system default).
     * </p>
     * <p>
     * Default: {@code -1}
     * </p>
     */
    public int getSocketTimeout() {
        return socketTimeout;
    }


{% endhighlight %}


### 正确解读一下

API中不能看出正式的含义是什么，经过demo之后，终于知道了各自含义

#### 1. connectTimeOut：指建立连接的超时时间，比较容易理解

#### 2. connectionRequestTimeOut：指从连接池获取到连接的超时时间，如果是非连接池的话，该参数暂时没有发现有什么用处

#### 3. socketTimeOut：指客户端和服务进行数据交互的时间，是指两者之间如果两个数据包之间的时间大于该时间则认为超时，而不是整个交互的整体时间，比如如果设置1秒超时，如果每隔0.8秒传输一次数据，传输10次，总共8秒，这样是不超时的。而如果任意两个数据包之间的时间超过了1秒，则超时。

### 代码

首先是为这次demo写的服务代码，包含几个controller方法（Spring MVC）

{% highlight java %}

package me.nabil.demo.springbootdemo;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.EnableAutoConfiguration;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.ResponseBody;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

/**
 * 超时测试
 *
 * @author nabilzhang
 */
@Controller
@EnableAutoConfiguration
@RequestMapping(value = {"/test"}, method = {RequestMethod.GET})
public class TimeoutTestController {

    private static final Logger logger = LoggerFactory.getLogger(TimeoutTestController.class);

    /**
     * main方法
     *
     * @param args 参数数组
     */
    public static void main(String args[]) {
        SpringApplication.run(TimeoutTestController.class, args);
    }

    /**
     * 1.测试socketOutTimeout，三秒后返回数据
     *
     * @return
     * @throws InterruptedException
     */
    @RequestMapping(value = {"/socket_timeout"}, method = {RequestMethod.GET})
    @ResponseBody
    String socketTimeout() throws InterruptedException {
        logger.info("socket_timeout");
        Thread.sleep(3000);
        return "socket_timeout";
    }

    /**
     * 2.测试socketOutTimeout，每隔0.8秒返回数据
     *
     * @return
     * @throws InterruptedException
     */
    @RequestMapping(value = {"/socket_timeout_2"}, method = {RequestMethod.GET})
    void socketTimeout2(HttpServletResponse response) throws InterruptedException, IOException {
        logger.info("socket_timeout_2");
        for (int i = 0; i < 10; i++) {
            logger.info("{}", i);
            response.getWriter().println("" + i);
            response.flushBuffer();
            Thread.sleep(800);
        }
    }

    /**
     * 3.测试connectionRequestTimeout用的服务，三秒后返回数据
     *
     * @param request
     * @return
     * @throws InterruptedException
     */
    @RequestMapping(value = {"/connection_request_timeout"}, method = {RequestMethod.GET})
    @ResponseBody
    String connectionRequestTimeout(HttpServletRequest request) throws InterruptedException {
        logger.info("{}", request.getRequestURI());
        Thread.sleep(3000);
        return "connectionRequestTimeout";
    }
}

{% endhighlight %}


如下是客户端的测试Case，下面几个Case分别测试了几个超时时间的各种情况，Case全部通过才可以

{% highlight java %}

package me.nabil.demo.springbootdemo;

import org.apache.http.client.ClientProtocolException;
import org.apache.http.client.config.RequestConfig;
import org.apache.http.client.methods.CloseableHttpResponse;
import org.apache.http.client.methods.HttpGet;
import org.apache.http.conn.ConnectTimeoutException;
import org.apache.http.impl.client.CloseableHttpClient;
import org.apache.http.impl.client.HttpClients;
import org.apache.http.impl.conn.BasicHttpClientConnectionManager;
import org.apache.http.impl.conn.PoolingHttpClientConnectionManager;
import org.apache.http.util.EntityUtils;
import org.junit.Assert;
import org.junit.Test;

import java.io.IOException;
import java.net.SocketTimeoutException;
import java.util.Date;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

/**
 * 测试HttpClient超时参数
 *
 * @author nabilzhang
 */
public class TimeoutTestControllerTest {

    /**
     * 1.connectionTimeout测试：IP无法链接，链接超时
     *
     * @throws Exception
     */
    @Test
    public void connectionTimeout() throws Exception {
        CloseableHttpClient httpclient = HttpClients.createDefault();
        HttpGet httpGet = new HttpGet("http://74.125.203.100");
        RequestConfig requestConfig = RequestConfig.custom().setConnectionRequestTimeout(1000)
                .setSocketTimeout(1000).setConnectTimeout(1000).build();
        httpGet.setConfig(requestConfig);
        try {
            httpclient.execute(httpGet);
        } catch (ConnectTimeoutException exception) {
            Assert.assertTrue(exception.getMessage().contains("connect timed out"));
        }

    }

    /**
     * 2.socketTimeout测试，服务端没有指定时间内任何响应，会超时
     *
     * @throws Exception
     */
    @Test
    public void socketTimeout() throws Exception {
        CloseableHttpClient httpclient = HttpClients.createDefault();
        HttpGet httpGet = new HttpGet("http://127.0.0.1:8080/test/socket_timeout");
        RequestConfig requestConfig = RequestConfig.custom()
                .setSocketTimeout(1000).build();
        httpGet.setConfig(requestConfig);
        try {
            httpclient.execute(httpGet);
        } catch (SocketTimeoutException exception) {
            Assert.assertEquals("Read timed out", exception.getMessage());
        }
    }

    /**
     * 3.socketTimeout测试：服务端隔800ms返回一点数据,不会超时
     *
     * @throws Exception
     */
    @Test
    public void socketTimeoutNo() {
        CloseableHttpClient httpclient = HttpClients.createDefault();
        HttpGet httpGet = new HttpGet("http://127.0.0.1:8080/test/socket_timeout_2");
        RequestConfig requestConfig = RequestConfig.custom().setConnectionRequestTimeout(1000)
                .setSocketTimeout(1000).setConnectTimeout(1000).build();
        httpGet.setConfig(requestConfig);

        try {
            httpclient.execute(httpGet);
            CloseableHttpResponse response = httpclient.execute(httpGet);
            System.out.println(String.format("socketTimeoutNo, %s", EntityUtils.toString(response.getEntity())));
        } catch (Exception e) {
            Assert.fail("服务端隔800ms返回一点数据,不会超时");
        }


    }

    /**
     * 4.connectionRequestTimeout测试:指从连接管理器(例如连接池)中拿到连接的超时时间
     *
     * @throws Exception
     */
    @Test
    public void connectionRequestTimeoutWithPoolingConnectionManager() throws Exception {
        PoolingHttpClientConnectionManager conMgr = new PoolingHttpClientConnectionManager();
        conMgr.setMaxTotal(2);

        final CloseableHttpClient httpclient = HttpClients.custom().setConnectionManager(conMgr).build();
        final HttpGet httpGet = new HttpGet("http://127.0.0.1:8080/test/connection_request_timeout");

        RequestConfig requestConfig = RequestConfig.custom().setConnectTimeout(1000)
                .setConnectionRequestTimeout(1000).setSocketTimeout(1000).build();
        httpGet.setConfig(requestConfig);

        // 如下多线程占满连接池
        ExecutorService executorService = Executors.newFixedThreadPool(10);
        for (int i = 0; i < 10; i++) {
            executorService.submit(new Runnable() {
                @Override
                public void run() {
                    try {
                        CloseableHttpResponse response = httpclient.execute(httpGet);
                        System.out.println(String.format("connectionRequestTimeoutTest: %s",
                                EntityUtils.toString(response.getEntity())));
                    } catch (SocketTimeoutException exception) {
                        System.out.println(exception.getMessage());
                    } catch (ClientProtocolException e) {
                        e.printStackTrace();
                    } catch (IOException e) {
                        e.printStackTrace();
                    }
                }
            });
        }

        // 在连接池占满的情况下，拿不到就会抛异常
        try {
            CloseableHttpResponse response = httpclient.execute(httpGet);
            System.out.println(String.format("connectionRequestTimeoutTest: %s",
                    EntityUtils.toString(response.getEntity())));
            Assert.fail();
        } catch (Exception exception) {
            // 异常是从连接池拿到连接超时
            Assert.assertEquals("Timeout waiting for connection from pool", exception.getMessage());
            System.out.println(exception.getMessage());
        }

    }

    /**
     * 5.connectionRequestTimeout测试，指从连接管理器中拿到连接的超时时间，由于使用基本的连接管理器，链接被占用时，直接无法分配链接
     * connectionRequestTimeout并未生效，目前看来该参数只在连接池奏效.
     * 该链接管理器(BasicHttpClientConnectionManager)是单线程情况下可以使用，多线程情况下不要使用。
     *
     * @throws Exception
     */
    @Test
    public void connectionRequestTimeoutWithBasicConnectionManager() throws Exception {

        BasicHttpClientConnectionManager connManager = new BasicHttpClientConnectionManager();
        final CloseableHttpClient httpclient = HttpClients.custom()
                .setConnectionManager(connManager).setMaxConnPerRoute(1).build();
        final HttpGet httpGet = new HttpGet("http://127.0.0.1:8080/test/connection_request_timeout");

        RequestConfig requestConfig = RequestConfig.custom().setConnectTimeout(100000)
                .setConnectionRequestTimeout(1000000).setSocketTimeout(1000000).build();
        httpGet.setConfig(requestConfig);

        // 如下多线程占满连接
        ExecutorService executorService = Executors.newFixedThreadPool(10);
        for (int i = 0; i < 10; i++) {
            executorService.submit(new Runnable() {
                @Override
                public void run() {
                    CloseableHttpResponse response = null;
                    try {
                        response = httpclient.execute(httpGet);
                        System.out.println(String.format("connectionRequestTimeoutTest: %s",
                                EntityUtils.toString(response.getEntity())));
                    } catch (Exception exception) {
                        exception.printStackTrace();
                    } finally {
                        if (response != null) {
                            try {
                                response.close();
                                httpclient.close();
                            } catch (IOException e) {
                                e.printStackTrace();
                            }
                        }

                    }
                }
            });
        }
        System.out.println(new Date());

        // 在连接池占满的情况下，拿不到就会抛异常
        try {
            CloseableHttpResponse response = httpclient.execute(httpGet);
            System.out.println(String.format("connectionRequestTimeoutTest: %s",
                    EntityUtils.toString(response.getEntity())));
            Assert.fail();
        } catch (Exception exception) {
            System.out.println(new Date());
            exception.printStackTrace();
            // 异常是从连接池拿到连接超时
            Assert.assertEquals("Connection is still allocated", exception.getMessage());
            System.out.println(exception.getMessage());
        }

    }

}

{% endhighlight %}

注：上面Case是使用httpclient版本4.5.2测试所得。

{% highlight xml %}

		<dependency>
            <groupId>org.apache.httpcomponents</groupId>
            <artifactId>httpclient</artifactId>
            <version>4.5.2</version>
            <type>jar</type>
        </dependency>

{% endhighlight %}

更深理解可以参照:<a href="http://www.baeldung.com/httpclient-connection-management">http://www.baeldung.com/httpclient-connection-management</a>
