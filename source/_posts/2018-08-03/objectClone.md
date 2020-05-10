---
title: Java基础系列(六)-对象克隆
date: 2018-08-03 15:27:58
tags: java
---



在Java中对象的克隆有深克隆和浅克隆之分。有这种区分的原因是Java中分为基本数据类型和引用数据类型，对于不同的数据类型在内存中的存储的区域是不同的。基本数据类型存储在栈中，引用数据类型存储在堆中。
java有8中基本数据类型:byte,short,int,long,float,double,char,boolean
<!--more-->

由浅入深,先看个克隆对象的例子

```java
public class Apple{
    private String color;
    private double weight;
    //constructArgs....
    //get set...
}
main(){
    Apple a = new Apple("red","10.00");
    Apple b = a;
    sout(a.getWeight());
    sout(b.getWeight());
    //change apple b property
    b.setWeight(20.00);
    sout(b.getWeight());
    sout(a.getWeight());
}
```

输出如何呢?我们会发现当改变苹果b的重量属性时,苹果a的重量属性也被修改了;
这与基本数据类型的赋值有差异,即,**传值**与**传址**的区别;基本数据类型传的是变量的值,所以两个变量就是两个值,修改其中一个变量不会影响到另外一个变量的值;而非基本数据类型传递的是对象在堆中的地址,两个对象指向的是同一个地址,所以b对象改变了地址中对象的属性值,那么a获取属性值当然也会变了.
以上的简单粗暴直接等号赋值的方式,明显是有坑;如何让他们互不影响呢,接下来介绍"浅克隆";

```java
public class Apple implements Cloneable{
    ......
@Override

protected Object clone(){
	Apple g = null;

	try{
		g = (Apple)super.clone();
	}catch (CloneNotSupportedException e){
		System.out.println(e.toString());
	}
	return g;
}
    
}
```

通过实现Cloneable接口,重写他的clone方法实现"对象分离";

通常遇到这种Apple只是属于"简单对象",没有引用其他的对象,我们一般现在使用:Apple b= (Apple)a.clone();即可实现分离的效果,但是如果有引用其他的对象,还需要做进一步的改造;

```java
public class Customer implements Cloneable{
    private String name;
    private String address;
    private Apple apple;
    //constructArgs
    //set and get
    @Override  
    public Customer clone() throws CloneNotSupportedException {  
        return (Customer) super.clone();  
    }
}
main(){
    Apple apple = new Apple("RED" , 10.00);  
    Customer customer1 = new Customer("name1", "address1" , apple);  
    Customer customer2 = customer1.clone();  
    customer2.getApple().setColor("GREEN");  
    customer2.setName("name2");  
    sout("customer1:"+customer1.toString());  
    sout("customer2:"+customer2.toString());
}
```

根据输出情况,发现当我们更改customer的name属性时,customer1与customer2互不影响;但是,更改Apple的color属性,customer1与customer2同时更改了;因为我们再对customer对象做clone操作的时候,只是针对了customer对象,并没有对Apple对象进行处理,所以customer1与customer2对象的apple属性仍然是指向了同一个地址;

通过如下方式,进一步的克隆对象:

```java
@Override  
public Customer clone() throws CloneNotSupportedException {  
    Customer customer = (Customer) super.clone();  
    customer.apple = apple.clone();  
    return customer;  
}
```

# 总结:

1.浅克隆：只复制基本类型的数据，引用类型的数据只复制了引用的地址，引用的对象并没有复制，在新的对象中修改引用类型的数据会影响原对象中的引用。  
2.深克隆：是在引用类型的类中也实现了clone，是clone的嵌套，复制后的对象与原对象之间完全不会影响。
3.使用clone实现的深克隆其实是浅克隆中嵌套了浅克隆，与toString方法类似
