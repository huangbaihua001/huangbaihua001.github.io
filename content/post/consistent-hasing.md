+++
author = "柏华"
title = "一致性哈希"
date = "2021-04-05"
description = "一致性哈希"
featured = true
tags = [
    "系统架构",
    "译文",
]

isCJKLanguage = true
+++
![](/images/graph/a1.webp)

最近我碰到过几次使用一致性哈希的情况。介绍这个概念的论文[《一致性哈希和随机树：分布式缓存协议，用于解决互联网应用中的热点问题。David Karger等人著》](http://citeseer.ist.psu.edu/karger97consistent.html)
出现在十年前，不过最近似乎这个概念已经悄悄地应用到从亚马逊的"Dynamo"到Last.fm提供的"Memcached"等越来越多的服务当中。那么，什么是一致性哈希，为什么要关心它？

<!--more-->

源文: [Consistent Hashing](http://tom-e-white.com/2007/11/consistent-hashing.html) (Tom White)

对一致性哈希的需求来自于多个缓存服务器缓存对象一些限制--例如，Web缓存。想象这样的场景，如果你有一个由<font color='red'>n</font>台缓存服务器(编号<font color='red'>1到n</font>)组成的集群，
那么在它们之间进行负载平衡的常见方法是将对象o使用<font color='red'>hash(o)</font>先取哈希值，然后对n取模(<font color='red'>mod n</font>)，
这样得到的值(<font color='red'>hash(o)</font> <font color='red'>mod n</font>)就是对应缓存服务器的编号，该缓存就会放在对应编号的缓存服务器上。
这工作良好，直到你添加或删除缓存服务器（无论出于什么原因），因为那时n发生了变化，每个对象都会被哈希到一个新的位置。这可能是灾难性的，因为可能缓存全部失效，所有的缓存请求会压到缓存后面的服务器（比如数据库），从而造成服务器的瘫痪。
这就好像缓存突然消失了一样。从某种意义上来说，它已经消失了。这就是为什么你应该关心一致性哈希的原因：需要一致性哈希来避免服务器的瘫痪！

如果添加一台缓存服务器时，它应该能从所有其他缓存服务器中获取它应得的对象；同样，当移除一台时，它的对象应该能被其余的服务器共享。
这正是一致性哈希所做的事情--将对象一致地映射到同一台缓存服务器上，至少在可能的情况下，是这样的。

一致性哈希算法背后的基本思想是使用相同的哈希函数对对象和缓存进行哈希。这样做的原因是将缓存映射到一个区间，这个区间将包含一些对象哈希。如果缓存被移除，那么它的区间就会被一个具有相邻区间的缓存接管。所有其他缓存保持不变。

# 证明
我们来详细看看这个问题。哈希函数实际上是将对象和缓存映射到一个数字范围。这应该是每个Java程序员都熟悉的: Object上的hashCode方法返回一个整数，它位于-2 <sup>31</sup>到2 <sup>31</sup>-1的范围内。
想象一下，把这个范围的数值环绕起来，就映射成了一个圆。下面是一张圆的图片，在它们哈希到的点上标记了一些对象（1、2、3、4）和缓存（A、B、C）（基于David Karger等人的《具有一致哈希的Web缓存》中的一张图）。

![](/images/graph/a2.png)

为了找到一个对象的缓存位置，我们顺时针绕着圆移动，直到找到一个缓存点。所以在上图中，我们看到对象1和4属于缓存A，对象2属于缓存B，而对象3属于缓存C。
考虑一下如果缓存C被删除会发生什么：对象3现在属于缓存A，而所有其他对象的映射没有变化。如果再在标记的位置增加一个缓存D，它将带走对象3和4，只留下属于A的对象1。

![](/images/graph/a3.png)

这样做的效果很好，只是分配给每个缓存的间隔大小很不理想。由于本质上是随机的，所以有可能在缓存之间的对象分布非常不均匀。
解决这个问题的方法是引入 "虚拟节点 "的概念，虚拟节点是圆内缓存点的复制。所以每当我们添加一个缓存时，我们就会为它在圆中创建一些点。

你可以在下面的图中看到这个效果，这是我用下面描述的代码模拟在10个缓存中存储10000个对象而制作的。
在x轴上是缓存点的复制数(虚拟节点)（用对数刻度）。当它很小的时候，我们看到对象在各缓存中的分布是不平衡的，因为标准差作为每个缓存的平均对象数的百分比（在y轴上，也是对数）很高。
随着复制数(虚拟节点)的增加，对象的分布变得更加平衡。这个实验表明，一两百个副本(虚拟节点)的数字可以达到一个可接受的平衡（标准偏差大致在平均值的5%到10%之间）。

![](/images/graph/a4.png)

# 实现
这里是一个简单的Java实现。为了使一致的散列有效，拥有一个能很好混合的散列函数是很重要的。
Object的hashCode的大多数实现都不能很好地混合--例如，它们通常会产生数量有限的小整数值--所以我们有一个HashFunction接口来允许使用自定义的哈希函数。这里推荐使用MD5哈希。

```java
import java.util.Collection;
import java.util.SortedMap;
import java.util.TreeMap;

public class ConsistentHash<T> {

  private final HashFunction hashFunction;
  private final int numberOfReplicas;
  private final SortedMap<Integer, T> circle =
    new TreeMap<Integer, T>();

  public ConsistentHash(HashFunction hashFunction,
    int numberOfReplicas, Collection<T> nodes) {

    this.hashFunction = hashFunction;
    this.numberOfReplicas = numberOfReplicas;

    for (T node : nodes) {
      add(node);
    }
  }

  public void add(T node) {
    for (int i = 0; i < numberOfReplicas; i++) {
      circle.put(hashFunction.hash(node.toString() + i),
        node);
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
      SortedMap<Integer, T> tailMap =
        circle.tailMap(hash);
      hash = tailMap.isEmpty() ?
             circle.firstKey() : tailMap.firstKey();
    }
    return circle.get(hash);
  } 

}
```

哈希圆用排序的整形映射来表示，整数代表映射到缓存的哈希值（这里是T类型）。
当一个ConsistentHash对象被创建后，每个节点都会被添加到圆中若干次（由numberOfReplicas控制）。每个副本(虚拟节点)的位置是通过对节点的名称和数字后缀进行哈希来选择的，节点被存储在图中的这些点上。

要找到一个对象的节点（get方法），就要用对象的哈希值在圆中寻找。大多数情况下，在这个哈希值处不会存储有节点（因为哈希值空间通常比节点数量大得多，即使是复制点也是如此），
所以通过在映射末尾寻找第一个键来找到下一个节点。如果映射末尾是空的，那么我们就通过获取圆的第一个键。

# 使用
那么如何才能使用一致的哈希呢？使用现成的库，而不是要自己编码。比如上面提到的Memcached，一个分布式内存对象缓存系统，现在已经有支持一致性哈希的客户端。
ketama算法是第一个，作者是Last.fm公司的Richard Jones；现在还有Dustin Sallings的Java实现（这启发了我上面的简化演示实现）。
有趣的是，只有客户端需要实现一致性哈希算法--Memcached服务器是不变的。其他采用一致散列的系统包括Chord，它是一个分布式散列表的实现，以及Amazon的Dynamo，它是一个键值存储（在Amazon之外无法使用）。


(全文完)




