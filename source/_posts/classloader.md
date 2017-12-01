---
title: 双亲委派模型
date: 2017-11-03 17:09:36
tags:
- java
- 面试
categories: java
---

> 双亲委派模型是java类加载器所使用的模型.

* 双亲委派模型的工作过程：如果一个类加载器收到了类加载器的请求.它首先不会自己去尝试加载这个类.而是把这个请求委派给父加载器去完成.每个层次的类加载器都是如此.

* 因此所有的加载请求最终都会传送到Bootstrap类加载器(启动类加载器)中.只有父类加载反馈自己无法加载这个请求(它的搜索范围中没有找到所需的类)时.子加载器才会尝试自己去加载.

### 双亲委派模型执行流程

![classloader](http://p066mj5r9.bkt.clouddn.com/static/images/20171103/classloader.jpg)

### 双亲委派模型的好处
> java类随着它的加载器一起具备了一种带有优先级的层次关系.

+ 例如类java.lang.Object,它存放在rt.jart之中.无论哪一个类加载器都要加载这个类.最终都是双亲委派模型最顶端的Bootstrap类加载器去加载.因此Object类在程序的各种类加载器环境中都是同一个类.相反.如果没有使用双亲委派模型.由各个类加载器自行去加载的话.如果用户编写了一个称为“java.lang.Object”的类.并存放在程序的ClassPath中.那系统中将会出现多个不同的Object类.java类型体系中最基础的行为也就无法保证.应用程序也将会一片混乱.

### 双亲委派的代码实现(在ClassLoader类中的loadClass中)
``` java
 protected Class<?> loadClass(String name, boolean resolve)
        throws ClassNotFoundException
    {
        synchronized (getClassLoadingLock(name)) {
            // First, check if the class has already been loaded
            Class<?> c = findLoadedClass(name);
            if (c == null) {
                long t0 = System.nanoTime();
                try {
                    if (parent != null) {
                        c = parent.loadClass(name, false);
                    } else {
                        c = findBootstrapClassOrNull(name);
                    }
                } catch (ClassNotFoundException e) {
                    // ClassNotFoundException thrown if class not found
                    // from the non-null parent class loader
                }

                if (c == null) {
                    // If still not found, then invoke findClass in order
                    // to find the class.
                    long t1 = System.nanoTime();
                    c = findClass(name);

                    // this is the defining class loader; record the stats
                    sun.misc.PerfCounter.getParentDelegationTime().addTime(t1 - t0);
                    sun.misc.PerfCounter.getFindClassTime().addElapsedTimeFrom(t1);
                    sun.misc.PerfCounter.getFindClasses().increment();
                }
            }
            if (resolve) {
                resolveClass(c);
            }
            return c;
        }
    }
```
> 逻辑：先检查是否已经被加载过.若没有加载则调用父加载器的loadClass()方法.若父加载器为空则默认使用Bootstrap类加载器作为父加载器.若父加载失败.抛出ClassNotFoundException异常后再调用自己的findClass()方法进行加载