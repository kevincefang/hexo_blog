---
title: CentOS7.3更换内核安装锐速破解版
date: 2017-12-07
tags: 
- vpn
- centos
categories: 工具
---

> centos更换内核安装serverspeeder锐速, 加速后的访问速度提高很多.

1. CentO S7.3的内核3.10.0-514.16.1.el7.x86_64暂不支持安装锐速，故更换为3.10.0-229.1.2.el7.x86_64

```shell
rpm -ivh http://soft.91yun.org/ISO/Linux/CentOS/kernel/kernel-3.10.0-229.1.2.el7.x86_64.rpm --force
```

2. 查看内核是否安装成功

```shell
rpm  -qa | grep kernel
```

3. 如果显示里面有这个内核就对了。

![1](http://fhaoer.com/hexo_blog/static/images/20180626/1.png)

4. 重启，查看内核是否更换成功

```shell
reboot
uname -r
```

5. 更换完内核后开始安装锐速破解版

```shell
wget -N --no-check-certificate https://github.com/91yun/serverspeeder/raw/master/serverspeeder.sh && bash serverspeeder.sh
```

查看状态

```shell
service serverSpeeder status
```

查看实时信息

```shell
service serverSpeeder stats
```

锐速服务是开机启动的，所以不用进行设置

锐速破解版卸载方法

```shell
chattr -i /serverspeeder/etc/apx* && /serverspeeder/bin/serverSpeeder.sh uninstall -f
```