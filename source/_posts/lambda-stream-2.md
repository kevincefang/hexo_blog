---
title: Java8新特性——StreamAPI(二)
date: 2018-06-27
tags: 
- java8
- lambda
categories: java
---

# 1. 收集器简介

收集器用来将经过筛选、映射的流进行最后的整理，可以使得最后的结果以不同的形式展现。

collect方法即为收集器，它接收Collector接口的实现作为具体收集器的收集方法。

Collector接口提供了很多默认实现的方法，我们可以直接使用它们格式化流的结果；也可以自定义Collector接口的实现，从而定制自己的收集器。

这里先介绍Collector常用默认静态方法的使用，自定义收集器会在下一篇博文中介绍。

# 2. 收集器的使用

## 2.1 归约

流由一个个元素组成，归约就是将一个个元素“折叠”成一个值，如求和、求最值、求平均值都是归约操作。

### 2.1.1 计数

```
long count = list.stream()
                    .collect(Collectors.counting());
```

也可以不使用收集器的计数函数：

```
long count = list.stream().count();
```

注意：计数的结果一定是long类型。

### 2.1.2 最值

> 例：找出所有人中年龄最大的人

```
Optional<Person> oldPerson = list.stream()
                    .collect(Collectors.maxBy(Comparator.comparingInt(Person::getAge)));
```

计算最值需要使用Collector.maxBy和Collector.minBy，这两个函数需要传入一个比较器Comparator.comparingInt，这个比较器又要接收需要比较的字段。
 这个收集器将会返回一个Optional类型的值。
 Optional类简介请移步至：[Java8新特性——StreamAPI(一)](https://link.jianshu.com?t=http%3A%2F%2Fblog.csdn.net%2Fu010425776%2Farticle%2Fdetails%2F52344425)

### 2.1.3 求和

> 例：计算所有人的年龄总和

```
int summing = list.stream()
                    .collect(Collectors.summingInt(Person::getAge));
```

当然，既然Java8提供了summingInt，那么还提供了summingLong、summingDouble。

### 2.1.4 求平均值

> 例：计算所有人的年龄平均值

```
double avg = list.stream()
                    .collect(Collectors.averagingInt(Person::getAge));
```

注意：计算平均值时，不论计算对象是int、long、double，计算结果一定都是double。

### 2.1.5 一次性计算所有归约操作

Collectors.summarizingInt函数能一次性将最值、均值、总和、元素个数全部计算出来，并存储在对象IntSummaryStatisics中。
 可以通过该对象的getXXX()函数获取这些值。

### 2.1.6 连接字符串

> 例：将所有人的名字连接成一个字符串

```
String names = list.stream()
                        .collect(Collectors.joining());
```

每个字符串默认分隔符为空格，若需要指定分隔符，则在joining中加入参数即可：

```
String names = list.stream()
                        .collect(Collectors.joining(", "));
```

此时字符串之间的分隔符为逗号。

### 2.1.7 一般性的归约操作

若你需要自定义一个归约操作，那么需要使用Collectors.reducing函数，该函数接收三个参数：

- 第一个参数为归约的初始值
- 第二个参数为归约操作进行的字段
- 第三个参数为归约操作的过程

> 例：计算所有人的年龄总和

```
Optional<Integer> sumAge = list.stream()
                                    .collect(Collectors.reducing(0,Person::getAge,(i,j)->i+j));
```

上面例子中，reducing函数一共接收了三个参数：

- 第一个参数表示归约的初始值。我们需要累加，因此初始值为0
- 第二个参数表示需要进行归约操作的字段。这里我们对Person对象的age字段进行累加。
- 第三个参数表示归约的过程。这个参数接收一个Lambda表达式，而且这个Lambda表达式一定拥有两个参数，分别表示当前相邻的两个元素。由于我们需要累加，因此我们只需将相邻的两个元素加起来即可。

Collectors.reducing方法还提供了一个单参数的重载形式。
 你只需传一个归约的操作过程给该方法即可(即第三个参数)，其他两个参数均使用默认值。

- 第一个参数默认为流的第一个元素
- 第二个参数默认为流的元素
   这就意味着，当前流的元素类型为数值类型，并且是你要进行归约的对象。

> 例：采用单参数的reducing计算所有人的年龄总和

```
Optional<Integer> sumAge = list.stream()
            .filter(Person::getAge)
            .collect(Collectors.reducing((i,j)->i+j));
```

## 2.2 分组

分组就是将流中的元素按照指定类别进行划分，类似于SQL语句中的GROUPBY。

### 2.2.1 一级分组

> 例：将所有人分为老年人、中年人、青年人

```
Map<String,List<Person>> result = list.stream()
                                    .collect(Collectors.groupingby((person)->{
        if(person.getAge()>60)
            return "老年人";
        else if(person.getAge()>40)
            return "中年人";
        else
            return "青年人";
                                    }));
```

groupingby函数接收一个Lambda表达式，该表达式返回String类型的字符串，groupingby会将当前流中的元素按照Lambda返回的字符串进行分组。
 分组结果是一个Map< String,List< Person>>，Map的键就是组名，Map的值就是该组的Perosn集合。

### 2.2.2 多级分组

多级分组可以支持在完成一次分组后，分别对每个小组再进行分组。
 使用具有两个参数的groupingby重载方法即可实现多级分组。

- 第一个参数：一级分组的条件
- 第二个参数：一个新的groupingby函数，该函数包含二级分组的条件

> 例：将所有人分为老年人、中年人、青年人，并且将每个小组再分成：男女两组。

```
Map<String,Map<String,List<Person>>> result = list.stream()
                                    .collect(Collectors.groupingby((person)->{
        if(person.getAge()>60)
            return "老年人";
        else if(person.getAge()>40)
            return "中年人";
        else
            return "青年人";
                                    },
                                    groupingby(Person::getSex)));
```

此时会返回一个非常复杂的结果：Map< String,Map< String,List< Person>>>。

### 2.2.3 对分组进行统计

拥有两个参数的groupingby函数不仅仅能够实现多几分组，还能对分组的结果进行统计。

> 例：统计每一组的人数

```
Map<String,Long> result = list.stream()
                                    .collect(Collectors.groupingby((person)->{
        if(person.getAge()>60)
            return "老年人";
        else if(person.getAge()>40)
            return "中年人";
        else
            return "青年人";
                                    },
                                    counting()));
```

此时会返回一个Map< String,Long>类型的map，该map的键为组名，map的值为该组的元素个数。

#### 将收集器的结果转换成另一种类型

当使用maxBy、minBy统计最值时，结果会封装在Optional中，该类型是为了避免流为空时计算的结果也为空的情况。在单独使用maxBy、minBy函数时确实需要返回Optional类型，这样能确保没有空指针异常。然而当我们使用groupingBy进行分组时，若一个组为空，则该组将不会被添加到Map中，从而Map中的所有值都不会是一个空集合。既然这样，使用maxBy、minBy方法计算每一组的最值时，将结果封装在optional对象中就显得有些多余。
 我们可以使用collectingAndThen函数包裹maxBy、minBy，从而将maxBy、minBy返回的Optional对象进行转换。

> 例：将所有人按性别划分，并计算每组最大的年龄。

```
Map<String,Integer> map = list.stream()
                            .collect(groupingBy(Person::getSex,
                            collectingAndThen(
                            maxBy(comparingInt(Person::getAge)),
                            Optional::get
                            )));
```

此时返回的是一个Map< String,Integer>，String表示每组的组名(男、女)，Integer为每组最大的年龄。
 如果不用collectingAndThen包裹maxBy，那么最后返回的结果为Map< String,Optional< Person>>。
 使用collectingAndThen包裹maxBy后，首先会执行maxBy函数，该函数执行完后便会执行Optional::get，从而将Optional中的元素取出来。

## 2.3 分区

分区是分组的一种特殊情况，它只能分成true、false两组。
 分组使用partitioningBy方法，该方法接收一个Lambda表达式，该表达是必须返回boolean类型，partitioningBy方法会将Lambda返回结果为true和false的元素各分成一组。
 partitioningBy方法返回的结果为Map< Boolean,List< T>>。
 此外，partitioningBy方法和groupingBy方法一样，也可以接收第二个参数，实现二级分区或对分区结果进行统计。

>  转载地址: https://www.jianshu.com/p/8854fa9b72c2

