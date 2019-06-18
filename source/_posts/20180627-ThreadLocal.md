---
title: 揭秘ThreadLocal
date: 2018-06-27
tags: 
- java
- 面试
categories: java
---

> ThreadLocal是开发中最常用的技术之一，也是面试重要的考点。本文将由浅入深，介绍ThreadLocal的使用方式、实现原理、内存泄漏问题以及使用场景。

### ThreadLocal作用

在并发编程中时常有这样一种需求：每条线程都需要存取一个同名变量，但每条线程中该变量的值均不相同。

如果是你，该如何实现上述功能？常规的思路如下：
 使用一个线程共享的`Map<Thread,Object>`，Map中的key为线程对象，value即为需要存储的值。那么，我们只需要通过`map.get(Thread.currentThread())`即可获取本线程中该变量的值。

这种方式确实可以实现我们的需求，但它有何缺点呢？——答案就是：需要同步，效率低！

由于这个map对象需要被所有线程共享，因此需要加锁来保证线程安全性。当然我们可以使用`java.util.concurrent.*`包下的`ConcurrentHashMap`提高并发效率，但这种方法只能降低锁的粒度，不能从根本上避免同步锁。而JDK提供的`ThreadLocal`就能很好地解决这一问题。下面来看看ThreadLocal是如何高效地实现这一需求的。

### 如何使用ThreadLocal

在介绍ThreadLocal原理之前，首先简单介绍一下它的使用方法。

 ```java
public class Main{
    private ThreadLocal<Integer> threadLocal = new ThreadLocal<>();
    
    public void start() {
        for (int i=0; i<10; i++) {
            new Thread(new Runnable(){
                @override
                public void run(){
                    threadLocal.set(i);
                    threadLocal.get();
                    threadLocal.remove();
                }
            }).start();
        }
    }
}
 ```

- 首先我们需要创建一个线程共享的ThreadLocal对象，该对象用于存储Integer类型的值；
- 然后在每条线程中可以通过如下方法操作ThreadLocal：
  * `set(obj)`：向当前线程中存储数据
  * `get()`：获取当前线程中的数据
  * `remove()`：删除当前线程中的数据

ThreadLocal的使用方法非常简单，关键在于它背后的实现原理。回到上面的问题：ThreadLocal究竟是如何避免同步锁，从而保证读写的高效？

![1](https://raw.githubusercontent.com/kevincefang/hexo_blog/master/static/images/20180627/1.png)

`ThreadLocal`并不维护`ThreadLocalMap`，并不是一个存储数据的容器，它只是相当于一个工具包，提供了操作该容器的方法，如get、set、remove等。而`ThreadLocal`内部类`ThreadLocalMap`才是存储数据的容器，并且该容器由`Thread`维护。

每一个`Thread`对象均含有一个`ThreadLocalMap`类型的成员变量`threadLocals`，它存储本线程中所有ThreadLocal对象及其对应的值。

`ThreadLocalMap`由一个个`Entry`对象构成，`Entry`的代码如下：

 ```java
static class Entry extends WeakReference<ThreadLocal<?>> {
    Object value;

    Entry(ThreadLocal<?> k, Object v) {
        super(k);
        value = v;
    }
}
 ```

`Entry`继承自`WeakReference<ThreadLocal<?>>`，一个`Entry`由`ThreadLocal`对象和`Object`构成。由此可见，`Entry`的key是ThreadLocal对象，并且是一个弱引用。当没指向key的强引用后，该key就会被垃圾收集器回收。

那么，ThreadLocal是如何工作的呢？下面来看set和get方法。

 ```java
public void set(T value) {
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null)
        map.set(this, value);
    else
        createMap(t, value);
}

public T get() {
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null) {
        ThreadLocalMap.Entry e = map.getEntry(this);
        if (e != null) {
            @SuppressWarnings("unchecked")
            T result = (T)e.value;
            return result;
        }
    }
    return setInitialValue();
}

ThreadLocalMap getMap(Thread t) {
    return t.threadLocals;
}
 ```

当执行set方法时，ThreadLocal首先会获取当前线程对象，然后获取当前线程的ThreadLocalMap对象。再以当前ThreadLocal对象为key，将值存储进ThreadLocalMap对象中。

get方法执行过程类似。ThreadLocal首先会获取当前线程对象，然后获取当前线程的ThreadLocalMap对象。再以当前ThreadLocal对象为key，获取对应的value。

由于每一条线程均含有各自**私有的**ThreadLocalMap容器，这些容器相互独立互不影响，因此不会存在线程安全性问题，从而也无需使用同步机制来保证多条线程访问容器的互斥性

### 为何要使用弱引用？

对弱引用不了解的同学可以参考笔者的另一篇文章：[http://blog.csdn.net/u010425776/article/details/50760053](https://link.jianshu.com?t=http%3A%2F%2Fblog.csdn.net%2Fu010425776%2Farticle%2Fdetails%2F50760053)。

Java设计之初的一大宗旨就是——弱化指针。
 Java设计者希望通过合理的设计简化编程，让程序员无需处理复杂的指针操作。然而指针是客观存在的，在目前的Java开发中也不可避免涉及到“指针操作”。如：

 ```java
Object a = new Object();
 ```

上述代码创建了一个强引用a，只要强引用存在，垃圾收集器是不会回收该对象的。如果该对象非常庞大，那么为了节约内存空间，在该对象使用完成后，我们需要手动拆除该强引用，如下面代码所示：

 ```java
a = null;
 ```

此时，指向该对象的强引用消除了，垃圾收集器便可以回收该对象。但在这个过程中，仍然需要程序员处理指针。为了弱化指针这一概念，弱引用便出现了，如下代码创建了一个Person类型的弱引用：

```java
WeakReference<Person> wr = new WeakReference<Person>(new Person());  
```

此时程序员不用再关注指针，只要没有强引用指向Person对象，垃圾收集器每次运行都会自动将该对象释放。

那么，ThreadLocalMap中的key使用弱引用的原因也是如此。当一条线程中的ThreadLocal对象使用完毕，没有强引用指向它的时候，垃圾收集器就会自动回收这个Key，从而达到节约内存的目的。

那么，问题又来了——这会导致内存泄漏问题！

### ThreadLocal的内存泄漏问题 

 在ThreadLocalMap中，只有key是弱引用，value仍然是一个强引用。当某一条线程中的ThreadLocal使用完毕，没有强引用指向它的时候，这个key指向的对象就会被垃圾收集器回收，从而这个key就变成了null；然而，此时value和value指向的对象之间仍然是强引用关系，只要这种关系不解除，value指向的对象永远不会被垃圾收集器回收，从而导致内存泄漏！

不过不用担心，ThreadLocal提供了这个问题的解决方案。

每次操作set、get、remove操作时，ThreadLocal都会将key为null的Entry删除，从而避免内存泄漏。

那么问题又来了，如果一个线程运行周期较长，而且将一个大对象放入LocalThreadMap后便不再调用set、get、remove方法，此时该仍然可能会导致内存泄漏。

这个问题确实存在，没办法通过ThreadLocal解决，而是需要程序员在完成ThreadLocal的使用后要养成手动调用remove的习惯，从而避免内存泄漏。

### ThreadLocal的使用场景

Web系统Session的存储就是ThreadLocal一个典型的应用场景。

Web容器采用线程隔离的多线程模型，也就是每一个请求都会对应一条线程，线程之间相互隔离，没有共享数据。这样能够简化编程模型，程序员可以用单线程的思维开发这种多线程应用。

当请求到来时，可以将当前Session信息存储在ThreadLocal中，在请求处理过程中可以随时使用Session信息，每个请求之间的Session信息互不影响。当请求处理完成后通过remove方法将当前Session信息清除即可。

> 转载地址: https://www.jianshu.com/p/3f3620f9011d

 

 

 

 

 

  

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 