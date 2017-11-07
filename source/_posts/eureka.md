---
title: Eureka
date: 2017-11-01 10:09:36
tags:
- springcloud
- REST服务
categories: springcloud
---
### Eureka 是什么?

Eureka是一个基于REST(表述性状态传递)的服务,主要用在AWS云定位服务中,目的是负载均衡和中间层故障转移服务.我们称之为Eureka服务. Eureka也配备了一个基于JAVA的客户端组件,Eureka客户端, 这使得服务的交互更加容易.这个客户端同样内置了一个负载均衡器,可以进行基本的循环负载均衡.在Netflix上,一个更为复杂的Eureka提供加权负载均衡基于多种因素如流量,资源使用错误条件等,以提供更高的弹性.


