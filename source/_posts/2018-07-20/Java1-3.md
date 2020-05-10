---
title: Java基础系列(三)-Java反射
date: 2018-07-20 20:41:44
tags: java
---

反射机制是java中的一个很强大的特性,能够在运行时获取类的信息,比如说,类的父类,接口,全部方法和参数,全部常量和变量;

反射的功能不仅仅是获取类的信息,还可以在运行时,动态的创建对象,甚至可以直接查看类的私有成员变量,还能获取类的注解信息,在泛型中类型的判断也经常会用到;

反射可以说完全打破了类的封装新,将类的信息全部暴露出来;

<!--more-->

在实际开发中,我们经常会赋值一个类对象的所有字段信息到另一个类对象中,首先,如何获取一个类的某个字段的值呢?

```
private static void copyFieldValue(Object objA, Object objB){
    if(objA == null)
        return;
    Class clzzA = objA.getClass();
    Class clzzB = objB.getClass();
    Fields[] fieldA = clzzA.getDeclaredFields();
    Fields[] fieldB = clzzB.getDeclaredFields();
    Map<String, Field> fieldMap = new HashMap<>();
    for(Field field : fieldA){
        fieldMap.put(field.getName(), field);        
    }
    for(Field field : fieldB){
        Field fieldA = fieldMap.get(field.getName());
        if(fieldA!=null){
            fieldB.setAccessible(true);
            fieldB.set(objB,getFieldValue(objA,fieldA.getName()))
        }
    }
}
```

当然,使用反射机制,会占用更多的资源,运行效率会相应的降低;

[**`下面介绍Class类常用的方法:`**]()

获取公共构造器:getConstructors()

获取所有构造方法:getDeclaredConstructors()

获取包含的方法:getMethod()

获取包含的属性:getField(String name)

获取内部类:getDeclaredClasseds()

获取实现的接口:getInterfaces()

获取修饰符:getModifiers()

获取所在包:getPackage()

获取类所在路径:getName()

获取类名:getSimpleName()

获取父类:getSuperClass()



[介绍几种获取类实例的方法:假设类为User]()

1.User user = new User();Class clzz = user.getClass();

2.Class clzz = Usr.class;

3.Class clzz = Class.forName("com.bistu.kiven.User");

相反的,clzz是Class的实例对象,我们可以对应的获取到他的实例对象;

User user = (User)clzz.newInstance();

[以下介绍易于混淆的方法]()

| getMethods()              | 返回该类继承以及自身声明的所有public的方法数组         |
| ------------------------- | ---------------------------------- |
| getDeclaredMethods()      | 返回该类自身声明的所有public的方法数组,不包括继承而来的    |
| getFields()               | 获取所有的public的成员变量信息,包括继承的           |
| getDeclaredFields()       | 该类自己声明的成员变量信息,public,private等      |
| getConstructors()         | 返回所有public构造方法                     |
| getDeclaredConstructors() | 返回类的所有构造方法,public,default,protect等 |

泛型不同对类型没有影响,即:

ArrayList<*String*> stringArrayList = new ArrayList<>();

ArrayList arrayList = new ArrayList();

stringArrayList == arrayList;


