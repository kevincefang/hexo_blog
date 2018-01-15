---
title: storm 启动相关命令
date: 2017-05-31
tags: storm 
categories: storm
---

##### 在启动storm之前要确保nimbus和supervisor上的Zookeeper已经启动 

###### 1.查看zk的状态：
`./zkServer.sh status`
##### 2.如果zk没有开启，将nimbus和supervisor的zk开启
`./zkServer.sh start`
##### 3.启动nimbus（切换到storm的bin目录下）
`nohup ./storm nimbus & `
##### 4.启动supervisor
`nohup ./storm supervisor  &`
##### 4.启动storm UI
`nohup ./storm ui & `	
	
	在浏览器中输入ip:8080/index.html进入storm UI界面（注意端口不一定是8080，注意配置）

 


