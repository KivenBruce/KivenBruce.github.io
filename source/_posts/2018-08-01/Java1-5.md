---
title: Java基础系列(五) - Java容器
date: 2018-08-01 22:14:17
tags: java
---

所谓容器,就是专门用来装对象的东西,当你有东西需要存放和管理的时候,就需要用到它,也行你会说,不是有数组吗?确实,使用数组存放相同类型对象是一个不错的选择,但是,他也有一个很大的缺陷,那就是数组的大小只能是固定的,不能错数组中动态的添加和删除一个对象,要扩容的时候,只能新建一个数组,然后把原来的对象全部复制到新的数组中,而且只能存放相同类型的对象,使用起来不太灵活.

<!--more-->

## **一,集合Collection接口**

java集合框架支持以下两种数据类型的集合,一种是存储一个元素的集合,简称集合<collection>,
一种是为了存储键值对<key-value>简称图(Map)
这里讲述java集合框架中的三种主要类型的集合:规则集<set>,线性表<list>,队列<queue>

![å¾çæè¿°](https://img.mukewang.com/5b076e86000183ef12800659.jpg)需要注意的是，容器中只能存放对象，而不能存放基本类型。所以当你将一个 int 型数据 1放入容器中的时候，其实它会自动装箱转换成 Integer 类后存入的，Java中每一种基本类型都有对应的引用类型。在容器存放的是多个对象的引用，对象本身还是放在堆内存中。容器可以存放不同类型，不限数量的数据类型。

set:实例用于存储一组不重复的元素
set包含三个具体类是:散列类hashset,链式散列类linkedhashset和树形集treeset

---

list:实例用于存储一个由元素构成的有序集合,元素可以为空

list:线性表不仅可以存储重复元素,而且还允许用户指定它们的存储位置,用户可以通过下标来访问元素.
包括数组线性表类ArrayList和链表类LinkedList
ArrayList用数组存储元素,这个数组是动态建立的,自动扩容,ArrayList中，默认的大小是10，当你使用new ArrayList();时并不会立即为数组分配大小为10的空间，而是等插入第一个元素时才会真正分配，这样做是为了节约内存空间。在扩容时，默认的扩容因子是1.5，每次需要扩容时，会将原数组大小的1.5倍和实际需要的数组空间进行比较，从中取最大值作为数组大小。然后新建一个数组，把原数组中的所有元素复制到新数组中去。所以扩容其实是最耗费时间的操作，不仅仅需要重新分配空间，而且需要重新赋值。将扩容因子选为1.5而不是2，也是为了在满足需求的前提下尽可能的节约空间，但如果事先就知道元素的大概个数时，最好先在构造器中设置好列表的容量，这样就可以省掉不少扩容时的开销。

因为ArrayList的方法操作的都是同一个内部数组，而所有方法都没有加锁，没有同步机制，所以它是线程不安全的。

---

queue:实例由于存储先进先出的方式处理对象

---

List特点：元素有放入顺序，元素可重复 ，Set特点：元素无放入顺序，元素不可重,queue,先进先出

## **二,Map接口**

map里面的元素都是以键值对(key-value)的形式存在的;

键值很像list中的下标,只不过在list中下标是数字,
而在map中下标可以是任意类型的对象,图中不可以有重复的键值,每个键值对应一个值
图的类型有三种,hashmap,linkedhashmap,treemap.

map有三种遍历方式:

1.通过遍历KeySet来遍历所有键值对(如果只是需要key,则效率较高,如果还需要同时得到value,则效率很低,因为map.get(key)效率低)

```
Map<Integer, Integer> map = new HashMap<Integer, Integer>();
Set<Integer> keys = map.keySet();
//遍历map中的键,值

for (Integer key : map.keySet()) { 
  System.out.println("Key = " + key);
  System.out.println("value = " + map.get(key));

} 
//遍历map中的键 
for (Integer key : map.keySet()) { 
  System.out.println("Key = " + key); 
} 
//遍历map中的值 
for (Integer value : map.values()) { 
  System.out.println("Value = " + value); 
}
```

2.通过entrySet来实现

```
Map<Integer, Integer> map = new HashMap<Integer, Integer>(); 
for (Map.Entry<Integer, Integer> entry : map.entrySet()) { 
  System.out.println("Key = " + entry.getKey() + ", Value = " + entry.getValue()); 
}
```

[3.使用EntrySet的Iterator来遍历(删除,修改元素值,使用该方法)]()

```
Map<Integer, Integer> map = new HashMap<Integer, Integer>(); 
Iterator<Map.Entry<Integer, Integer>> entries = map.entrySet().iterator(); 
while (entries.hasNext()) { 
  Map.Entry<Integer, Integer> entry = entries.next(); 
  System.out.println("Key = " + entry.getKey() + ", Value = " + entry.getValue()); 
}
```


