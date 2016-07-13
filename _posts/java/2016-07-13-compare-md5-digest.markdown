---
layout: post
title:  "Java中比较不同的MD5计算方式"
date:   2016-07-13 12:37:55
categories: Java
---

在项目中经常需要使用计算文件的md5，用作一些用途，md5计算算法，通常在网络上查询时，一般给的算法是读取整个文件的字节流，然后计算文件的md5,这种方式当文件较大，且有很大并发量时，则可能导致内存打爆掉。所以如下代码提供了几种方式。并通过计算一个323M的文件的md5和大小给出了，GC的一些信息

### 代码

{% highlight java %}

/*
 * Copyright (C) 2016 Baidu, Inc. All Rights Reserved.
 */
package me.nabil.mixed;

import org.apache.commons.codec.digest.DigestUtils;
import org.apache.commons.io.FileUtils;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.io.File;
import java.io.FileInputStream;
import java.io.IOException;
import java.security.MessageDigest;
import java.security.NoSuchAlgorithmException;

/**
 * md5加密类
 */
public class Md5Util {

    private static final Logger LOGGER = LoggerFactory.getLogger(Md5Util.class);

    protected static final char[] hexDigits = {'0', '1', '2', '3', '4', '5', '6', '7', '8',
            '9', 'a', 'b', 'c', 'd', 'e', 'f'};

    /**
     * 将二进制数据进行md5加密
     *
     * @param data 文件二进制数据
     * @return md5加密码
     */
    public static String getMd5ByByte(byte[] data) {
        try {
            char[] str;
            MessageDigest mdTemp = MessageDigest.getInstance("MD5");
            mdTemp.update(data);
            byte[] md = mdTemp.digest();
            int j = md.length;
            str = new char[j * 2];
            int k = 0;
            for (int i = 0; i < j; i++) {
                byte byte0 = md[i];
                str[k++] = hexDigits[byte0 >>> 4 & 0xf];
                str[k++] = hexDigits[byte0 & 0xf];
            }

            return new String(str);
        } catch (Exception e) {
            LOGGER.error("Error occurred when making MD5 for data file", e);
            return null;
        }
    }

    /**
     * 文件对象
     *
     * @param file
     * @return
     */
    public static String getMD5ByFile(File file) {
        FileInputStream fis = null;
        try {
            MessageDigest md = MessageDigest.getInstance("MD5");
            fis = new FileInputStream(file);
            byte[] buffer = new byte[8192];
            int length = -1;
            System.out.println("开始算");
            while ((length = fis.read(buffer)) != -1) {
                md.update(buffer, 0, length);
            }
            System.out.println("算完了");
            return bytesToString(md.digest());
        } catch (IOException ex) {
            LOGGER.info(ex.getMessage(), ex);
            return null;
        } catch (NoSuchAlgorithmException e) {
            LOGGER.info(e.getMessage(), e);
            return null;
        } finally {
            try {
                fis.close();
            } catch (IOException ex) {
                LOGGER.info(ex.getMessage(), ex);
            }
        }
    }


    /**
     * bytesToString
     *
     * @param data
     * @return
     */
    public static String bytesToString(byte[] data) {
        char hexDigits[] = {'0', '1', '2', '3', '4', '5', '6', '7', '8', '9', 'a', 'b', 'c', 'd',
                'e', 'f'};
        char[] temp = new char[data.length * 2];
        for (int i = 0; i < data.length; i++) {
            byte b = data[i];
            temp[i * 2] = hexDigits[b >>> 4 & 0x0f];
            temp[i * 2 + 1] = hexDigits[b & 0x0f];
        }
        return new String(temp);
    }

    /**
     * main
     *
     * @param args
     * @throws IOException
     */
    public static void main(String[] args) throws IOException {
        String path = "/path/to/file"; // 一个323M的文件路径

        // 1. 流处理
        System.out.println(getMD5ByFile(new File(path)) + ":" + FileUtils.sizeOf(new File(path)));

        // 2. common-codec
        System.out.println(DigestUtils.md5Hex(new FileInputStream(path)) + ":" + FileUtils.sizeOf(new File(path)));

        // 3. 全部读进内存
        byte[] data = FileUtils.readFileToByteArray(new File(path));
        System.out.println(getMd5ByByte(data) + ":" + data.length);

    }
}


{% endhighlight %}

### 分析

以上三种方式，通过注释其他两种，只运行其中一种的方式来给出JVM的一些信息

运行时加入参数jvm参数：-XX:+PrintGC -XX:+PrintGCDetails -XX:+PrintGCTimeStamps -XX:+PrintGCApplicationStoppedTime

####1  8M buffer流处理

```

0.151: Total time for which application threads were stopped: 0.0000535 seconds, Stopping threads took: 0.0000072 seconds
0.151: Total time for which application threads were stopped: 0.0000428 seconds, Stopping threads took: 0.0000083 seconds
Connected to the target VM, address: '127.0.0.1:63630', transport: 'socket'
0.314: Total time for which application threads were stopped: 0.0001357 seconds, Stopping threads took: 0.0000145 seconds
0.417: Total time for which application threads were stopped: 0.0001063 seconds, Stopping threads took: 0.0000169 seconds
开始算
1.442: Total time for which application threads were stopped: 0.0221453 seconds, Stopping threads took: 0.0219728 seconds
算完了
Disconnected from the target VM, address: '127.0.0.1:63630', transport: 'socket'
aa3858dfbc48daab4b891e112d5396fc:323251542
Heap
 PSYoungGen      total 38400K, used 6658K [0x0000000795580000, 0x0000000798000000, 0x00000007c0000000)
  eden space 33280K, 20% used [0x0000000795580000,0x0000000795c00b00,0x0000000797600000)
  from space 5120K, 0% used [0x0000000797b00000,0x0000000797b00000,0x0000000798000000)
  to   space 5120K, 0% used [0x0000000797600000,0x0000000797600000,0x0000000797b00000)
 ParOldGen       total 87552K, used 0K [0x0000000740000000, 0x0000000745580000, 0x0000000795580000)
  object space 87552K, 0% used [0x0000000740000000,0x0000000740000000,0x0000000745580000)
 Metaspace       used 3560K, capacity 4728K, committed 4864K, reserved 1056768K
  class space    used 386K, capacity 424K, committed 512K, reserved 1048576K

Process finished with exit code 0

```

####2  apache common-codec的结果


```

0.184: Total time for which application threads were stopped: 0.0000615 seconds, Stopping threads took: 0.0000078 seconds
0.185: Total time for which application threads were stopped: 0.0000279 seconds, Stopping threads took: 0.0000056 seconds
0.640: Total time for which application threads were stopped: 0.0001445 seconds, Stopping threads took: 0.0000329 seconds
0.891: Total time for which application threads were stopped: 0.0001546 seconds, Stopping threads took: 0.0000383 seconds
1.904: Total time for which application threads were stopped: 0.0102225 seconds, Stopping threads took: 0.0101437 seconds
4.580: Total time for which application threads were stopped: 0.0001097 seconds, Stopping threads took: 0.0000165 seconds
aa3858dfbc48daab4b891e112d5396fc:323251542
Heap
 PSYoungGen      total 38400K, used 9508K [0x0000000795580000, 0x0000000798000000, 0x00000007c0000000)
  eden space 33280K, 28% used [0x0000000795580000,0x0000000795ec92b8,0x0000000797600000)
  from space 5120K, 0% used [0x0000000797b00000,0x0000000797b00000,0x0000000798000000)
  to   space 5120K, 0% used [0x0000000797600000,0x0000000797600000,0x0000000797b00000)
 ParOldGen       total 87552K, used 0K [0x0000000740000000, 0x0000000745580000, 0x0000000795580000)
  object space 87552K, 0% used [0x0000000740000000,0x0000000740000000,0x0000000745580000)
 Metaspace       used 3616K, capacity 4792K, committed 5120K, reserved 1056768K
  class space    used 391K, capacity 424K, committed 512K, reserved 1048576K
Disconnected from the target VM, address: '127.0.0.1:63674', transport: 'socket'

```


####3  全部读进内存再计算


```
0.161: Total time for which application threads were stopped: 0.0000584 seconds, Stopping threads took: 0.0000190 seconds
0.161: Total time for which application threads were stopped: 0.0000407 seconds, Stopping threads took: 0.0000054 seconds
0.356: Total time for which application threads were stopped: 0.0001168 seconds, Stopping threads took: 0.0000177 seconds
0.860: [GC (Allocation Failure) [PSYoungGen: 21892K->4950K(38400K)] 21892K->25438K(125952K), 0.0187028 secs] [Times: user=0.01 sys=0.01, real=0.02 secs] 
0.879: Total time for which application threads were stopped: 0.0189020 seconds, Stopping threads took: 0.0000150 seconds
4.411: Total time for which application threads were stopped: 0.0445206 seconds, Stopping threads took: 0.0435764 seconds
4.694: Total time for which application threads were stopped: 0.0003998 seconds, Stopping threads took: 0.0000154 seconds
5.695: Total time for which application threads were stopped: 0.0001298 seconds, Stopping threads took: 0.0000841 seconds
aa3858dfbc48daab4b891e112d5396fc:323251542
Heap
 PSYoungGen      total 38400K, used 22181K [0x0000000795580000, 0x000000079a080000, 0x00000007c0000000)
  eden space 33280K, 51% used [0x0000000795580000,0x0000000796653bd0,0x0000000797600000)
  from space 5120K, 96% used [0x0000000797600000,0x0000000797ad58d0,0x0000000797b00000)
  to   space 5120K, 0% used [0x0000000799b80000,0x0000000799b80000,0x000000079a080000)
 ParOldGen       total 863744K, used 827683K [0x0000000740000000, 0x0000000774b80000, 0x0000000795580000)
  object space 863744K, 95% used [0x0000000740000000,0x0000000772848dd8,0x0000000774b80000)
 Metaspace       used 3663K, capacity 4792K, committed 5120K, reserved 1056768K
  class space    used 393K, capacity 424K, committed 512K, reserved 1048576K
Disconnected from the target VM, address: '127.0.0.1:63717', transport: 'socket'

Process finished with exit code 0

```



### 结论

可以看出前两种方案是可取的，第三种方式内存占用太高。
