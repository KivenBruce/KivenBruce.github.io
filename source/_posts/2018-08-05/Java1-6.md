---
title: Java进阶系列(一)HashMap
date: 2018-08-05 21:50:51
tags: java
---

HashMap这个我们日常开发中经常使用的一个数据结构,你真的了解其中的奥秘吗?

HashMap中的数据结构是:数组+链表的形式来存储节点的,每个节点以键值对<Node<K,V>>的形式存储

你可以想象成这样的结构:Node<K,V>[] table;那么,Node到底是个什么东西呢?

<!--more-->

```java
static class Node<K,V> implements Map.Entry<K,V>{
    final int hash;
    final K key;
    V value;
    //指向下一个节点的应用
    Node<K,V> next;
    Node(int hash,K key,V value,Node<K,V> next){
        this.hash = hash;
        this.K = K;
        this.V = V;
        this.next = next;
    }
    ......
}
```

我们往HashMap中put元素的时候,实际上在内存中是什么样子的呢?

```
 Map<String, Integer> map = new HashMap();
        map.put("小明", 66);
        map.put("小李", 77);
        map.put("小红", 88);
        map.put("小刚", 89);
        map.put("小力", 90);
        map.put("小王", 91);
        ......
```

![å¾çæè¿°](https://img.mukewang.com/5b24f9c30001b41510340163.png)

![å¾çæè¿°](https://img.mukewang.com/5b24f9c90001adf404310800.png)

即:数组+链表;可以看出,数组并不是顺序往里面存的,中间有很多空的"桶"(英文中称之为"bin"),之所以会出现这样的结果,是由于他的put方法导致的;

put方法往map中增加一个元素,如果map中已经存在该key的映射，则旧的值将会被替换,返回该key映射的旧值，如果该key的映射不存在的话则返回null。

往HashMap中添加元素的逻辑:

对key的hashCode()做一次散列(hash函数),然后根据这个散列值,计算index(i=(n-1)&hash),n:table.length,hash:hash(key),

多说一句,hash()涉及到一个"扰动函数"的概念,扰动函数的目的就是为了扩大高位的影响,使得计算出来的数字包含了高16位和低16位的特性,让hash值更加深不可测从而降低碰撞的概率!这里(n-1)是一个很聪明的办法,因为HashMap中的table大小是2的整数次幂,也就是说,肯定不是质数,那么如果使用一般的散列,即取余处理,那么在取余的过程中,偶数的映射范围就少了一半,n-1,n是table的大小，默认是16，二进制即为10000，n - 1 对应的二进制则为1111，这样再与hash值做“与”操作时，就变成了掩码，除了最后四位全部被置为0，而最后四位的范围肯定会落在（0~n-1）之间，正好是数组的大小范围，散列函数的妙处就在于此了。

如果没有发生碰撞(哈希冲突),则直接放入桶中,如果发生碰撞,则以链表的形式挂在桶后

```java
if ((p = tab[i = (n - 1) & hash]) == null)
            //不存在，则新建节点
            tab[i] = newNode(hash, key, value, null);
else {
            Node<K,V> e; K k;
            //先找到对应的node
            if (p.hash == hash &&
                    ((k = p.key) == key || (key != null && key.equals(k))))
                e = p;
            else if (p instanceof TreeNode)
                //如果是树节点，则调用相应的putVal方法
                //todo putTreeVal
                e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
            else {
                //如果是链表则之间遍历查找
                for (int binCount = 0; ; ++binCount) {
                    if ((e = p.next) == null) {
                        //如果没有找到则在该链表新建一个节点挂在最后
                        p.next = newNode(hash, key, value, null);
                        if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                            //如果链表长度达到树化的最大长度，则进行树化
                            //todo treeifyBin
                            treeifyBin(tab, hash);
                        break;
                    }
                    if (e.hash == hash &&
                            ((k = e.key) == key || (key != null && key.equals(k))))
                        break;
                    p = e;
                }
            }
```

举例说明:

小明的hash值是756692，转换为二进制为10111000101111010100，table的大小是32，n-1=31，对应的二进制为：11111，做“与”运算之后，得到的结果是10100，即为20。  
　　小李的hash值是757012，转换为二进制为10111000110100010100，与11111做与运算后，得到的结果也是10100，即20，于是就与小明发生了冲突，但还是要先来后到，于是小李就挂在了小明后面。  

如果因为碰撞导致链表过长(大于等于TREEIFY_THRESHOLD)，就把链表转换成红黑树；

如果桶满了(超过负载因子*当前容量)，就要resize（重新调整大小并重新散列）。

负载因子表示table最大装满程度，默认是0.75，即当容量被用掉75%后将会触发扩容，因为当table中的元素足够多时，发生冲突的概率就会大大增加，冲突的增多会导致每个桶中的元素个数变多，这样的话会使得查找元素效率变得低下，当同一个桶中元素个数达到8时，桶中的元素结构将转换为红黑树。

那么，问题来了，为什么是8，而不是6或者7,10呢？？？

最佳实践,哈哈!HashMap中源码中注释的解释,大概意思是:

理想情况下，HashCode随机分布，当负载因子设置成0.75时，那么在桶中元素个数的概率大致符合0.5的[泊松分布](https://baike.baidu.com/item/%E6%B3%8A%E6%9D%BE%E5%88%86%E5%B8%83/1442110?fr=aladdin)，桶中元素个数达到8的概率小于千万分之一，因为转化为红黑树还是比较耗时耗力的操作，自然不希望经常进行，但如果设置得过大，将失去设置该值的意义。

总结:

1.HashMap的结构是什么？

　　HashMap是数组+链表的存储形式，默认的初始容量是16，默认的加载因子是0.75，当链表长度达到8时将会转化为红黑树来提高查找效率。

2.HashMap的优点和缺点是什么？

　　HashMap的优点是查找速度很快，我们可以在常数时间内迅速定位到某个桶以及要找的对象。

        缺点散列算法是依赖key的hashcode，所以如果key的hashcode设计的很烂，将会严重影响性能。  
　　极端情况下，如果每次计算hash值都是同一个值，那么会造成链表中长度过长然后转化成树，扩容时再散列的效果也很差的问题。 另一个极端情况. 每次计算hash值都是不同的值，那么就是HashMap中的数组会不断的扩容，造成HashMap的容量不断增大。

　　另一方面，HashMap是线程不安全的，如果想在并发编程中使用到HashMap，就需要使用它的同步类，Collections.synchronizedMap()方法将普通的HashMap转化成线程安全的，或者使用Concurrent包下的ConcurrentHashMap进行替换。

3.HashMap的get()方法和put()方法的工作原理是什么？

　　通过对key的hashCode()进行hashing，并计算下标( n-1 & hash)，从而获得桶的位置。如果发生哈希冲突，则利用key.equals()方法去链表或树中去查找对应的节点。

4.HashMap中的碰撞探测(collision detection)以及碰撞的解决方法是什么？

　　当两个key的hashCode相同时，就会发生碰撞，就像上面的小明和小李，这时候后添加的元素将会以链表或树节点的形式挂在桶后面。

5.如果HashMap的大小超过了负载因子(load factor)定义的容量，怎么办？

　　如果HashMap的大小超过了负载因子*容量，那么将会进行扩容操作，扩容到原来的两倍。

还有一个比较特殊的地方,细心观察,你会发现HashMap中的table,entrySet,value等成员变量都是用transient修饰的,为什么要这么做呢?

transient,涉及到java的序列化,使用transient关键字来标志,表示不想让这个字段被序列化,HashMap有他的考虑,HashMap中有很多桶都是空的,将其序列化没有任何意义,另一个更重要的原因是:HashMap存储是依赖对象的HashCode的,而object.hashCode方法是依赖具体虚拟机的,同一个对象,在不同虚拟机的HashCode可能不同,那么映射到HashMap中的位置也不一样;

```java
public V get(Object key) {
 Node<K,V> e;
 return (e = getNode(hash(key), key)) == null ? null : e.value;
 }
```


