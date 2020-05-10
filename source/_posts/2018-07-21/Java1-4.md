---
title: Java基础系列(四) - Java泛型
date: 2018-07-21 20:43:04
tags: java
---

泛型是java中比较常见的特性,从以下几个方面来了解泛型:

1,什么是泛型

2,如何使用泛型

3,泛型的好处

---

## **1,什么是泛型**

泛型,指的是参数化类型,平时我们所面对的类型一般是某个具体的类型,如String,Integer,Double,void,而泛型则是把所操作的数据类型当做一个参数,如ArrayList<String>(),通过传入不同的类型来指定容器中存储的类型,而不用为了不同类型创建不同的类,这种参数类型可以用在类,接口和方法的创建中,分别称为泛型类,泛型接口,泛型方法

<!--more-->

## **2,如何使用泛型**

我们先看看泛型是什么样子的:

```
public interface List<E>{
    void add<E>;
    Iterator<E> iterator();
}
```

这是List接口,这里用E来代替具体的类型,这样就可以往里面传入任意类型;

那么,直接使用Object不就好了吗?如下例

```
public class ObjHolder{
    private Object a;
    public ObjHolder(Object a){this.a = a}
    public void set(Object a){this.a = a}
    public Object get(){return a;}
    public void main(String[] args){
        ObjHolder hoder = new ObjHolder("jack");
        System.out.println((String)holder.get());
        holder.set(123);
        System.out.println((Integer)holder.get());
    }
}
```

由上,可以发现,虽然也实现类存储任意类型的对象,但是每次取出来的都需要进行类型转换,如果参数类型为ObjectHolder的话,无法知道它里面存放的对象的确切类型,这样反而带来很多不必要的麻烦;

那么,如果使用泛型来实现呢?

```
public class GeneriHolder<T>{//类则放在类名后面
    private T obj;//变量则放在前面
    public GeneriHolder(T obj){this.obj = obj}
    public T getObj(){return obj}//方法则放在前面
    public void setObj(T obj){this.obj = obj}
    public static void main(String[] args){
        GeneriHolder<String>  holderA = new GenericHolder<String>("java");
        System.out.println(holderA.getObj());
        //holderA.setObj(123);//error ,just can put a value with type of string,not a integer
        GeneriHolder<Integer> holderB = new GeneriHolder<Integer>(123);
        System.out.println(holder.getObj());
    }
}
```

这样通过传入类型信息如String 和Integer,来代替其中的泛型差数T,这里的T你可以理解为一个占位符,用其他字母代替也是可以的;一旦传入的类型确定,如String,则所有使用T的地方都会用String类型替换;

在泛型中可以对类型进行限制,如:<T extends Comparable<T>>表示只能传入已经实现类Comparable接口的类型对象,.注意,这里使用的是extends而不是implement,而且对于接口也只能写一个,<T extends Number>表示只能接收Number类或者其子类对象.与之相反的边界通配符是super,如<T super Phone>表示只能接收类型为Phone或者是其父类的对象;

特别注意的是,使用extends和super在以下情形不使用:

```java
List<? extends Number> list = new ArrayList<Number>();
list.add(1.0);//编译报错
list.add(3);//编译报错 
```

因为泛型是为了类型安全设计的，如果往List<? extends Number> list 塞值的话，在取的时候就无法确认它到底是什么类型了，编译器只知道它是Number类型或者它的派生类型，但无法确定是哪个具体类型。通配符T表示其中存的都是同一种类型，因此使用extend下边界的话是无法进行存操作的(放的类型不兼容,取值会报错)。同理super下边界是不能取值的(放的类型虽然可能不同,但是都是父类向上类型转换,最终能到Object,所以取出来都认为是Object类型)。

所以,经常要读,则使用<? extends T>,经常要写,则使用<? super T>

## **3.泛型的好处**

3.1,类型安全

泛型的主要目标是提高Java程序的类型安全,通过使用泛型定义的变量的类型现在,避免了大量因为使用Object带来的不必要的类型错误.

3.2,代码复用

增加了代码的复用性,比如上面的GenericHolder类,就能存取任意类型的对象而不用为例每种类型写一个包装类


