---
layout: post
title:  "理解一致性hash算法"
date:   2014-10-09 21:37:55
categories: distributed
---

###应用场景###
分布式缓存通常有一种这样一种场景：用4台Cache服务器缓存所有Object。
那么我将如何把一个Object映射至对应的Cache服务器呢？最简单的方法设置缓存规则是hashCode除以节点数取余：object.hashCode() % 4。

这种方法看起来可行，但是当考虑业务缩减或者业务增长需要移出节点或者增加节点时，按照其他节点数取余余数将会全部发生变化，那之前所有的缓存都将失效，所以大规模部署中很少会使用这种方法。这种方式难以满足单调性。

###单调性
Hash 算法的一个衡量指标是单调性（ Monotonicity ），定义如下：

单调性是指如果已经有一些内容通过哈希分派到了相应的缓冲中，又有新的缓冲加入到系统中。哈希的结果应能够保证原有已分配的内容可以被映射到新的缓冲中去，而不会被映射到旧的缓冲集合中的其他缓冲区。

###Consistent Hashing 算法的原理
consistent hashing 是一种 hash 算法，简单的说，在移除 / 添加一个 cache 时，它能够尽可能小的改变已存在 key 映射关系，尽可能的满足单调性的要求。

一致性哈希算法采用一种新的方式来解决问题，不再仅仅依赖object.hashCode()本身，而且将Cache的配置也进行哈希运算。具体步骤如下：

1.首先求出每个Cache的哈希值，并将其配置到一个0~2^32的圆环区间上。

2.使用同样的方法求出需要存储对象的哈希值，也将其配置到这个圆环上。

3.从数据映射到的位置开始顺时针查找，将数据保存到找到的第一个Cache节点上。如果超过2^32仍然找不到Cache节点，就会保存到第一个Cache节点上。
![Alt text](/post_img/Consistent-Hashing/1.gif "consistent hashing 算法的原理")

####新增Cache节点
假设在这个环形哈希空间中，Cache5被映射在Cache3和Cache4之间，那么受影响的将仅是沿Cache5逆时针遍历直到下一个Cache（Cache3）之间的对象（它们本来映射到Cache4上）。
![Alt text](/post_img/Consistent-Hashing/2.gif "新增Cache节点受到的影响")

####移除Cache节点
假设在这个环形哈希空间中，Cache3被移除，那么受影响的将仅是沿Cache3逆时针遍历直到下一个Cache（Cache2）之间的对象（它们本来映射到Cache3上）。
![Alt text](/post_img/Consistent-Hashing/3.gif "移除Cache节点受到的影响")

####代码示例

{% highlight java %}
import java.util.Collection;
import java.util.SortedMap;
import java.util.TreeMap;

public class ConsistentHash<T> {

 private final HashFunction hashFunction;
 private final SortedMap<Integer, T> circle = new TreeMap<Integer, T>();

 public ConsistentHash(HashFunction hashFunction, int numberOfReplicas,
     Collection<T> nodes) {
   this.hashFunction = hashFunction;

   for (T node : nodes) {
     add(node);
   }
 }

 public void add(T node) {
    circle.put(hashFunction.hash(node.toString()), node);
 }

 public void remove(T node) {
    circle.remove(hashFunction.hash(node.toString()));
 }

 public T get(Object key) {
   if (circle.isEmpty()) {
     return null;
   }
   int hash = hashFunction.hash(key);
   if (!circle.containsKey(hash)) {
        SortedMap<Integer, T> tailMap = circle.tailMap(hash);
        hash = tailMap.isEmpty() ? circle.firstKey() : tailMap.firstKey();
   }
   return circle.get(hash);
 }

}
{% endhighlight %}

###平衡性问题
Hash算法并不一定能保证平衡，尤其Cache较少的话，对象并不能被均匀的映射到 Cache上，这样节点的负载会很不均衡，系统也无法稳定。

###虚拟Cache节点
正是由于平衡性的问题，所以需要考虑缓解不平衡的问题。Consistent Hashing引入了“虚拟节点”的概念： “虚拟节点”是实际节点在环形空间的复制品，一个实际节点对应了若干个“虚拟节点”，这个对应个数也成为“复制个数”，“虚拟节点”在哈希空间中以哈希值排列。仍以4台Cache服务器为例，在下图中看到，引入虚拟节点，并设置“复制个数”为2后，共有8个“虚拟节点”分部在环形区域上，缓解了映射不均的情况。这里只是缓解了不平衡情况，完全的平衡没有做到。
![Alt text](/post_img/Consistent-Hashing/4.gif "虚拟节点")

引入了“虚拟节点”后，映射关系就从【对象--->Cache服务器】转换成了【对象--->虚拟节点---> Cache服务器】。查询对象所在Cache服务器的映射关系如下图所示。
![Alt text](/post_img/Consistent-Hashing/5.gif "对象--->虚拟节点---> Cache服务器")

####引入平衡性考虑后的代码示例
{% highlight java %}
import java.util.Collection;
import java.util.SortedMap;
import java.util.TreeMap;

public class ConsistentHash<T> {

 private final HashFunction hashFunction;
 private final int numberOfReplicas;//虚拟节点与真实节点数的倍数
 private final SortedMap<Integer, T> circle = new TreeMap<Integer, T>();

 public ConsistentHash(HashFunction hashFunction, int numberOfReplicas,
     Collection<T> nodes) {
   this.hashFunction = hashFunction;//hash算法可以自行实现
   this.numberOfReplicas = numberOfReplicas;

   for (T node : nodes) {
     add(node);
   }
 }

 public void add(T node) {
   for (int i = 0; i < numberOfReplicas; i++) {
     circle.put(hashFunction.hash(node.toString() + i), node);
   }
 }

 public void remove(T node) {
   for (int i = 0; i < numberOfReplicas; i++) {
     circle.remove(hashFunction.hash(node.toString() + i));
   }
 }

 public T get(Object key) {
   if (circle.isEmpty()) {
     return null;
   }
   int hash = hashFunction.hash(key);
   if (!circle.containsKey(hash)) {
     SortedMap<Integer, T> tailMap = circle.tailMap(hash);
     hash = tailMap.isEmpty() ? circle.firstKey() : tailMap.firstKey();
   }
   return circle.get(hash);
 }

}
{% endhighlight %}

###Hash算法的选择
[consistent-hashing](https://weblogs.java.net/blog/2007/11/27/consistent-hashing)
中推荐了Md5哈希算法。


###参考
[http://blog.csdn.net/x15594/article/details/6270242](http://blog.csdn.net/x15594/article/details/6270242)
[https://weblogs.java.net/blog/2007/11/27/consistent-hashing](https://weblogs.java.net/blog/2007/11/27/consistent-hashing)
[http://blog.csdn.net/sparkliang/article/details/5279393](http://blog.csdn.net/sparkliang/article/details/5279393)
