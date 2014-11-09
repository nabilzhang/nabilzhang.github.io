---
layout: post
title:  "android实现session登录保持"
date:   2012-03-19 02:13:09
categories: android
---

在最近写的一个Android中需要请求web服务器中的数据，有一个登录Activity，登录后会到MainActivity,这中间登录和MainActivity都需要请求php的jsonapi，所以要在网络请求中保持session的，研究了好半天才搞定。其实sesion在浏览器和web服务器直接是通过一个叫做name为sessionid的cookie来传递的，所以只要在每次数据请求时保持sessionid是同一个不变就可以用到web的session了，做法是第一次数据请求时就获取sessionid的值并保存在一个静态变量中，然后在第二次请求数据的时候要将这个sessionid一并放在Cookie中发给服务器，服务器则是通过这个sessionid来识别究竟是那个客户端在请求数据的，在php中这个sessionid的名字叫做PHPSESSID。下面贴下代码

{% highlight java %}
import java.io.IOException;
import java.io.UnsupportedEncodingException;
import java.util.List;

import org.apache.http.HttpEntity;
import org.apache.http.HttpResponse;
import org.apache.http.HttpStatus;
import org.apache.http.NameValuePair;
import org.apache.http.client.ClientProtocolException;
import org.apache.http.client.CookieStore;
import org.apache.http.client.entity.UrlEncodedFormEntity;
import org.apache.http.client.methods.HttpPost;
import org.apache.http.cookie.Cookie;
import org.apache.http.impl.client.DefaultHttpClient;
import org.apache.http.protocol.HTTP;
import org.apache.http.util.EntityUtils;

public class MyHttpClient implements InetConfig {
    private DefaultHttpClient httpClient;
    private HttpPost httpPost;
    private HttpEntity httpEntity;
    private HttpResponse httpResponse;
    public static String PHPSESSID = null;
    public LVHttpClient() {

    }

    public String executeRequest(String path, List<NameValuePair> params) {
        String ret = "none";
        try {
            this.httpPost = new HttpPost(BASEPATH + path);
            httpEntity = new UrlEncodedFormEntity(params, HTTP.UTF_8);
            httpPost.setEntity(httpEntity);
            //第一次一般是还未被赋值，若有值则将SessionId发给服务器
            if(null != PHPSESSID){
                httpPost.setHeader("Cookie", "PHPSESSID=" + PHPSESSID);
            }            
            httpClient = new DefaultHttpClient();
        } catch (UnsupportedEncodingException e) {
            e.printStackTrace();
        }
        try {
            httpResponse = httpClient.execute(httpPost);
            if (httpResponse.getStatusLine().getStatusCode() == HttpStatus.SC_OK) {
                HttpEntity entity = httpResponse.getEntity();
                ret = EntityUtils.toString(entity);
                CookieStore mCookieStore = httpClient.getCookieStore();
                List<Cookie> cookies = mCookieStore.getCookies();
                for (int i = 0; i < cookies.size(); i++) {
                    //这里是读取Cookie['PHPSESSID']的值存在静态变量中，保证每次都是同一个值
                    if ("PHPSESSID".equals(cookies.get(i).getName())) {
                        PHPSESSID = cookies.get(i).getValue();
                        break;
                    }

                }
            }
        } catch (ClientProtocolException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }

        return ret;
    }
}
{% endhighlight %}

其实web的原理都是一样的，基于http协议的，那么如果网站不是php做的话，那个叫做Sessionid的Cookie可能叫做别的了，就不是PHPSESSID了，而是叫做别的名字了，这个要具体情况去查。

其实不只是Android程序，其他任何程序需要这么用的时候只需要在http协议请求header里头加上发送相应的SessionId就可以了。刚刚这种方法是可以帮助理解sessionid的，其实还有一种方法如果更通用的话，就可以将刚刚所有的Cookie每次都发回到服务器端，也就可以解决session保持的问题了，只是这样可能会稍微大些网络流量开销而已。

这里看到一个SessionId的本质，顺便mark一下。