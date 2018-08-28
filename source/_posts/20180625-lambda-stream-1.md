---
title: Java8新特性——StreamAPI(一)
date: 2018-06-25
tags: 
- java8
- lambda
categories: java
---

### 1. 流的基本概念

#### 1.1 什么是流？

流是Java8引入的全新概念，它用来处理集合中的数据，暂且可以把它理解为一种高级集合。

众所周知，集合操作非常麻烦，若要对集合进行筛选、投影，需要写大量的代码，而流是以声明的形式操作集合，它就像SQL语句，我们只需告诉流需要对集合进行什么操作，它就会自动进行操作，并将执行结果交给你，无需我们自己手写代码。

因此，流的集合操作对我们来说是透明的，我们只需向流下达命令，它就会自动把我们想要的结果给我们。由于操作过程完全由Java处理，因此它可以根据当前硬件环境选择最优的方法处理，我们也无需编写复杂又容易出错的多线程代码了。

#### 1.2 流的特点

1. 只能遍历一次
    我们可以把流想象成一条流水线，流水线的源头是我们的数据源(一个集合)，数据源中的元素依次被输送到流水线上，我们可以在流水线上对元素进行各种操作。一旦元素走到了流水线的另一头，那么这些元素就被“消费掉了”，我们无法再对这个流进行操作。当然，我们可以从数据源那里再获得一个新的流重新遍历一遍。
2. 采用内部迭代方式
    若要对集合进行处理，则需我们手写处理代码，这就叫做外部迭代。而要对流进行处理，我们只需告诉流我们需要什么结果，处理过程由流自行完成，这就称为内部迭代。

#### 1.3 流的操作种类

流的操作分为两种，分别为中间操作 和 终端操作。

1. 中间操作
    当数据源中的数据上了流水线后，这个过程对数据进行的所有操作都称为“中间操作”。
    中间操作仍然会返回一个流对象，因此多个中间操作可以串连起来形成一个流水线。
2. 终端操作
    当所有的中间操作完成后，若要将数据从流水线上拿下来，则需要执行终端操作。
    终端操作将返回一个执行结果，这就是你想要的数据。

#### 1.4 流的操作过程

使用流一共需要三步：

1. 准备一个数据源
2. 执行中间操作
    中间操作可以有多个，它们可以串连起来形成流水线。
3. 执行终端操作
    执行终端操作后本次流结束，你将获得一个执行结果。

### 2. 流的使用

#### 2.1 获取流

在使用流之前，首先需要拥有一个数据源，并通过StreamAPI提供的一些方法获取该数据源的流对象。数据源可以有多种形式：

1. 集合
    这种数据源较为常用，通过stream()方法即可获取流对象：

```
List<Person> list = new ArrayList<Person>(); 
Stream<Person> stream = list.stream();
```

1. 数组
    通过Arrays类提供的静态函数stream()获取数组的流对象：

```
String[] names = {"chaimm","peter","john"};
Stream<String> stream = Arrays.stream(names);
```

1. 值
    直接将几个值变成流对象：

```
Stream<String> stream = Stream.of("chaimm","peter","john");
```

1. 文件
    try(Stream<String> lines = Files.lines(Paths.get("文件路径名"),Charset.defaultCharset())){
    //可对lines做一些操作
    }catch(IOException e){
    }
    PS：Java7简化了IO操作，把打开IO操作放在try后的括号中即可省略关闭IO的代码。

#### 2.2 筛选filter

filter函数接收一个Lambda表达式作为参数，该表达式返回boolean，在执行过程中，流将元素逐一输送给filter，并筛选出执行结果为true的元素。
 如，筛选出所有学生：

```
List<Person> result = list.stream()
                    .filter(Person::isStudent)
                    .collect(toList());
```

#### 2.3 去重distinct

去掉重复的结果：

```
List<Person> result = list.stream()
                    .distinct()
                    .collect(toList());
```

#### 2.4 截取

截取流的前N个元素：

```
List<Person> result = list.stream()
                    .limit(3)
                    .collect(toList());
```

#### 2.5 跳过

跳过流的前n个元素：

```
List<Person> result = list.stream()
                    .skip(3)
                    .collect(toList());
```

#### 2.6 映射

对流中的每个元素执行一个函数，使得元素转换成另一种类型输出。流会将每一个元素输送给map函数，并执行map中的Lambda表达式，最后将执行结果存入一个新的流中。
 如，获取每个人的姓名(实则是将Perosn类型转换成String类型)：

```
List<Person> result = list.stream()
                    .map(Person::getName)
                    .collect(toList());
```

#### 2.7 合并多个流

> 例：列出List中各不相同的单词，List集合如下：

```
List<String> list = new ArrayList<String>();
list.add("I am a boy");
list.add("I love the girl");
list.add("But the girl loves another girl");
```

思路如下：

- 首先将list变成流：

```
list.stream();
```

- 按空格分词：

```
list.stream()
            .map(line->line.split(" "));
```

分完词之后，每个元素变成了一个String[]数组。

- 将每个String[]变成流：

```
list.stream()
            .map(line->line.split(" "))
            .map(Arrays::stream)
```

此时一个大流里面包含了一个个小流，我们需要将这些小流合并成一个流。

- 将小流合并成一个大流：
   用flagmap替换刚才的map

```
list.stream()
            .map(line->line.split(" "))
            .flagmap(Arrays::stream)
```

- 去重

```
list.stream()
            .map(line->line.split(" "))
            .flagmap(Arrays::stream)
            .distinct()
            .collect(toList());
```

#### 2.8 是否匹配任一元素：anyMatch

anyMatch用于判断流中是否存在至少一个元素满足指定的条件，这个判断条件通过Lambda表达式传递给anyMatch，执行结果为boolean类型。
 如，判断list中是否有学生：

```
boolean result = list.stream()
            .anyMatch(Person::isStudent);
```

#### 2.9 是否匹配所有元素：allMatch

allMatch用于判断流中的所有元素是否都满足指定条件，这个判断条件通过Lambda表达式传递给anyMatch，执行结果为boolean类型。
 如，判断是否所有人都是学生：

```
boolean result = list.stream()
            .allMatch(Person::isStudent);
```

#### 2.10 是否未匹配所有元素：noneMatch

noneMatch与allMatch恰恰相反，它用于判断流中的所有元素是否都不满足指定条件：

```
boolean result = list.stream()
            .noneMatch(Person::isStudent);
```

#### 2.11 获取任一元素findAny

findAny能够从流中随便选一个元素出来，它返回一个Optional类型的元素。

```
Optional<Person> person = list.stream()
                                    .findAny();
```

### Optional介绍

Optional是Java8新加入的一个容器，这个容器只存1个或0个元素，它用于防止出现NullpointException，它提供如下方法：

- isPresent()
   判断容器中是否有值。
- ifPresent(Consume<T> lambda)
   容器若不为空则执行括号中的Lambda表达式。
- T get()
   获取容器中的元素，若容器为空则抛出NoSuchElement异常。
- T orElse(T other)
   获取容器中的元素，若容器为空则返回括号中的默认值。

#### 2.12 获取第一个元素findFirst

```
Optional<Person> person = list.stream()
                                    .findFirst();
```

#### 2.13 归约

归约是将集合中的所有元素经过指定运算，折叠成一个元素输出，如：求最值、平均数等，这些操作都是将一个集合的元素折叠成一个元素输出。

在流中，reduce函数能实现归约。
 reduce函数接收两个参数：

- 初始值
- 进行归约操作的Lambda表达式

##### 2.13.1 元素求和：自定义Lambda表达式实现求和

例：计算所有人的年龄总和

```
int age = list.stream().reduce(0, (person1,person2)->person1.getAge()+person2.getAge());
```

reduce的第一个参数表示初试值为0；
 reduce的第二个参数为需要进行的归约操作，它接收一个拥有两个参数的Lambda表达式，reduce会把流中的元素两两输给Lambda表达式，最后将计算出累加之和。

##### 2.13.2 元素求和：使用Integer.sum函数求和

上面的方法中我们自己定义了Lambda表达式实现求和运算，如果当前流的元素为数值类型，那么可以使用Integer提供了sum函数代替自定义的Lambda表达式，如：

```
int age = list.stream().reduce(0, Integer::sum);
```

Integer类还提供了min、max等一系列数值操作，当流中元素为数值类型时可以直接使用。

#### 2.14 数值流的使用

采用reduce进行数值操作会涉及到基本数值类型和引用数值类型之间的装箱、拆箱操作，因此效率较低。
 当流操作为纯数值操作时，使用数值流能获得较高的效率。

##### 2.14.1 将普通流转换成数值流

StreamAPI提供了三种数值流：IntStream、DoubleStream、LongStream，也提供了将普通流转换成数值流的三种方法：mapToInt、mapToDouble、mapToLong。
 如，将Person中的age转换成数值流：

```
IntStream stream = list.stream()
                            .mapToInt(Person::getAge);
```

##### 2.14.2 数值计算

每种数值流都提供了数值计算函数，如max、min、sum等。
 如，找出最大的年龄：

```
OptionalInt maxAge = list.stream()
                                .mapToInt(Person::getAge)
                                .max();
```

由于数值流可能为空，并且给空的数值流计算最大值是没有意义的，因此max函数返回OptionalInt，它是Optional的一个子类，能够判断流是否为空，并对流为空的情况作相应的处理。
 此外，mapToInt、mapToDouble、mapToLong进行数值操作后的返回结果分别为：OptionalInt、OptionalDouble、OptionalLong

