---
title: Java基础系列(二)-Java代理
date: 2018-07-19 19:13:24

tags: java

---

[一,Java代理-静态代理]()

什么是代理?举个例子,最形象的代表便是经纪人，明星一般都有经纪人，经纪人作为中间人，负责代理明星的相关事宜;经纪人就是代理;

```
public interface IStarts{
void sing();
void dance();
}
public class Start implements IStart{
piblic Start(String name){
}
public void sing(){
System.out.println(name+"唱了一首歌!")
}
public void dance(){
System.out.println(name+"跳了一支舞!");
}
}
```

<!--more-->

现在创建一个代理类:

```
public class StartProxy implements IStart{
private Start start;
public StartProxy(Start start){
this.start = start;
}
public void sing(){
System.out.println('我是代理,我收到唱歌请求');
starts.sing();
System.out.println("唱歌完毕");
}
...
}
```

上面的测试,可以看出,实际上代理类只是保存了Start类饿一个实例,实现的是相同的接口,StartProxy类必须实现dance和sing方法,你可以这么理解:代理接的活和实际做的是一致的,只是交给了具体的Start来实现;

`那么,为什么要使用代理呢?`

其实,主要目的是为了扩展原有类的功能,代理类实现了相同的接口,做的事情是一样的,只不过可以控制Start在何种条件下去执行,比如需要统计唱歌和跳舞的次数,次数大于3则不进行该操作,直接返回.这时候,使用代理就很好地实现,把代理类做如下修改:

```java
public class StartProxy implements IStart{
private Star start;
private int num;//保存次数
public void sing(){
if(!ifWork()) return;
else ...
}
public boolean ifWork(){
if(num>3) return false;
else
num++;return true;
}
}
```

当然,这里仅仅是一种代理饿思想,用这种代理思想可以比较方便的在不修改原有类的前提下对原有类进行扩展;

但是,限制也是显而易见的:

1.代理类需要与被代理类实现相同的接口

2.当被代理的类需要进行的扩展增多时,管理会变得更加困难,之后对被代理类的修改,同时需要修改代理类

[二,JDK动态代理]()

动态代理,指的是运行时创建代理对象,动态代理具有更强大的拦截请求功能,因为可以获取类的运行时信息,所以可以根据运行时信息来获取更加灵活的执行

还是以明星和经纪人作为例子;

修改代理类;创建JDK动态代理需要先实现InvocationHandler接口,并且重写其中的invoke方法,具体步骤如下:

1.创建一个类实现InvocationHandler接口

2.给Proxy类提供委托类的ClassLoader和Interfaces来创建动态代理类

3.利用反射机制得到动态代理类的构造函数

4.利用动态代理类的构造函数创建动态代理类对象

```java
public class StartNewProxy implements InvocationHandler{
private Object object;//代理类持有委托类的对象引用
public StartNewProxy(Object object){
this.object = object;
}
public Object invoke(Object proxy,Method method,Object[]args){
//利用反射机制将请求分派给委托类处理,Method的invoke返回Object对象作为方法执行结果
//**这前后可以做一些条件控制**
Object result = method.invoke(object,args);
return result;
}
}
```

新建一个工厂类来返回代理实例：

```java
public class StarsNewProxyFactory {
    //构建工厂类，客户类调用此方法获得代理对象
    //对于客户类而言，代理类对象和委托类对象是一样的，不需要知道具体返回的类型
    public static IStars getInstance(String name){
        IStars stars = new Stars(name);
        InvocationHandler handler = new StarsNewProxy(stars);
        IStars proxy = null;
        //相当于,拦截了代理类中的所有方法
        proxy = (IStars) Proxy.newProxyInstance(
                stars.getClass().getClassLoader(),
                stars.getClass().getInterfaces(),
                handler
        );
        return proxy;
    }
}
private  static  void test(){  
  IStars proxy =  StarsNewProxyFactory.getInstance("Frank"); 
  proxy.dance(); proxy.sing();  
}
```

*动态代理跟静态代理最大的不同便是生成代理类的时期不同，静态代理是在编译期，而动态代理则是在运行时根据委托类信息动态生成。*
