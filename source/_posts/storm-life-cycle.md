---
title: Topology的初始化流程
date: 2017-05-31 17:30:10
tags: storm
categories: storm
---

> 1、 首先配置好topology，并定义好Spout实例和Bolt实例
> 
> 2、 在提交Topology实例给Nimbus的过程中，会调TopologyBuilder实例的createTopology()方法，以获取定义的Topology实例。在运行createTopology()方法的过程中，会去调用Spout和Bolt实例上的declareOutputFields()方法和getComponentConfiguration()方法，declareOutputFields()方法配置Spout和Bolt实例的输出，getComponentConfiguration()方法输出特定于Spout和Bolt实例的配置参数值对。Storm会将以上过程中得到的实例，输出配置和配置参数值对等数据序列化，然后传递给Nimbus。
> 
> 3、在Worker Node上运行的thread，从Nimbus上复制序列化后得到的字节码文件，从中反序列化得到Spout和Bolt实例，实例的输出配置和实例的配置参数值对等数据，在thread中Spout和Bolt实例的declareOutputFields()和getComponentConfiguration()不会再运行。
> 
> 4、在thread中，反序列化得到一个Bolt实例后，它会先运行Bolt实例的prepare()方法，在这个方法调用中，需要传入一个OutputCollector实例，后面使用该OutputCollector实例输出Tuple
> 
> 5、接下来在该thread中按照配置数量建立task集合，然后在每个task中就会循环调用thread所持有Bolt实例的execute()方法
> 
> 6、在关闭一个thread时，thread所持有的Bolt实例会调用cleanup()方法

	不过如果是强制关闭，这个cleanup()方法有可能不会被调用到



