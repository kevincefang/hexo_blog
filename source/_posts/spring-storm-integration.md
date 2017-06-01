---
title: springboot-storm-integration
date: 2017-06-01 13:38:31
tags:
- storm
- spring
- springboot
categories: storm
---

 > 最近在搭建storm与springboot框架的集成，由于本身对storm分布式框架的不熟悉，加上网上能搜到的spring与storm集成的案例也比较少，这一路走来真的是各种坎坷啊，所以也在此总结一下遇到的一些问题，同时也希望能帮到有需要的朋友。
 
	在搭建框架之前，有必要先熟悉一下storm的topology在提交过程中，初始化了什么，实例化了哪些类？
##### (1) 首先简单说一下，提交topology的流程：
> 1、首先定义好topology的Config，实例化DrpcSpout、以及Bolt之间的拓扑结构
> 
> 2、提交Topology之前确认storm的各种服务都启动了，包括zk、nimbus、supervisor、logviewer、ui 
> 
> 3、提交Topology实例给nimbus，这时候调用TopologyBuilder实例的createTopology()方法，以获取定义的Topology实例。在运行createTopology()方法的过程中，会去调用Spout和Bolt实例上的declareOutputFields()方法和getComponentConfiguration()方法，declareOutputFields()方法配置Spout和Bolt实例的输出，getComponentConfiguration()方法输出特定于Spout和Bolt实例的配置参数值对。Storm会将以上过程中得到的实例，输出配置和配置参数值对等数据序列化，然后传递给Nimbus。
> 
> 4、Worker Node上运行的thread，从nimbus上复制序列化后得到的字节码文件，从中反序列化得到Spout和Bolt实例，实例的输出配置和实例的配置参数值对等数据，在thread中Spout和Bolt实例的declareOutputFields()和getComponentConfiguration()不会再运行。
> 
> 4、在thread中，反序列化得到一个Bolt实例后，它会先运行Bolt实例的prepare()方法，在这个方法调用中，需要传入一个OutputCollector实例，后面使用该OutputCollector实例输出Tuple

> 5、接下来在该thread中按照配置数量建立task集合，然后在每个task中就会循环调用thread所持有Bolt实例的execute()方法

> 6、在关闭一个thread时，thread所持有的Bolt实例会调用cleanup()方法

##### （2）storm与springboot的集成
> 1、定义springboot的主入口，也就是Application的启动类

 ```
 @Configuration
 @EnableAutoConfiguration
 @ComponentScan(basePackages="com.demo")
 public class Main extends ApplicationObjectSupport {
 
 }
 ```
 
> 2、由于storm的每个bolt都相当于独立的应用，正好每个bolt提供了一个prepare方法，这个prepare方法是在topology提交的时候调用的，这个时候可以把加载spring的过程，放在此处，从而也保证了每个bolt都能获取到Spring的ApplicationContext，有了ApplicationContext，后面的一切都好说了，springboot的任何功能的可以正常使用。废话不说直接贴代码：

 ```
 public void prepare(Map stormConf, TopologyContext context) {
    super.prepare(stormConf, context);
    logger.info("Main start...");
    new SpringApplicationBuilder(Main.class).web(false).run(new String[]{});
    logger.info("Main end...");
 }
 ```
 
> 3、获取ApplicationContext前，还需要实现ApplicationContextAware接口，注意一定要加上@Component，spring才会去加载当前类
 
```
@Component
public class BeanUtils implements ApplicationContextAware{

    private static ApplicationContext applicationContext = null;

    @Override
    public void setApplicationContext(ApplicationContext arg0) throws BeansException {
         if (BeanUtils.applicationContext == null) {
             BeanUtils.applicationContext = arg0;
         }
     }

     // 获取applicationContext
     public static ApplicationContext getApplicationContext() {
         return applicationContext;
     }

    // 通过name获取 Bean.
     public static Object getBean(String name) {
         return getApplicationContext().getBean(name);
     }

     // 通过class获取Bean.
     public static <T> T getBean(Class<T> clazz) {
         return getApplicationContext().getBean(clazz);
     }

     // 通过name,以及Clazz返回指定的Bean
    public static <T> T getBean(String name, Class<T> clazz) {
         return getApplicationContext().getBean(name, clazz);
     }
}
```

> 4、通过ApplicationContext获取Service实现类,注意Service一定要加上@Service(name="demoService")，不加别名的话，会获取不到，你可以试一下。
`
(DemoService) applicationContext.getBean("demoService");
`

**到此简单的整合就完成了，重点是每个bolt都需要独立的ApplicationContext，才能获取beans，切入点也就是bolt的prepare()方法中。** 

 
 
 
