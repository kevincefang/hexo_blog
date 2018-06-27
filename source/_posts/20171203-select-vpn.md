---
title: 搭建自己专属的vpn——Centos搭建vpn的几种办法
date: 2017-12-03
tags: 
- vpn
categories: 工具
---

### 方法一：搭建shadowsocks+serverspeeder（特别推荐）

### shadowsocks服务端安装

参考[官方Shadowsocks使用说明](https://github.com/shadowsocks/shadowsocks/wiki/Shadowsocks-%E4%BD%BF%E7%94%A8%E8%AF%B4%E6%98%8E)：

CentOS:

```
yum install python-setuptools && easy_install pip  
pip install shadowsocks
```

Debian / Ubuntu:

```
apt-get install python-pip  
pip install shadowsocks
```

配置
参考[Configuration via Config File](https://github.com/shadowsocks/shadowsocks/wiki/Configuration-via-Config-File)

修改配置文件/etc/shadowsocks.json，如果没有则新建。
内容如下：

```
{  
    "server":"0.0.0.0",  
    "server_port":8388,  
    "local_address": "127.0.0.1",  
    "local_port":1080,  
    "password":"mypassword",  
    "timeout":300,  
    "method":"aes-256-cfb",  
    "fast_open": false  
}
```

或（多个SS账号）

```
{  
    "server":"0.0.0.0",  
    "port_password":{  
     "8381":"xxxxxxx",  
     "8382":"xxxxxxx",  
     "8383":"xxxxxxx",  
     "8384":"xxxxxxx"  
     },  
    "timeout":300,  
    "method":"aes-256-cfb",  
    "fast_open": false  
}
```

配置说明：

| 字段          | 说明                            |
| ------------- | ------------------------------- |
| server        | ss服务监听地址                  |
| server_port   | ss服务监听端口                  |
| local_address | 本地的监听地址                  |
| local_port    | 本地的监听端口                  |
| password      | 密码                            |
| timeout       | 超时时间，单位秒                |
| method        | 加密方法，默认是aes-256-cfb     |
| fast_open     | 使用TCP_FASTOPEN, true / false  |
| workers       | workers数，只支持Unix/Linux系统 |

启动：
前台启动

```
ssserver -c /etc/shadowsocks.json
```

后台启动与停止

```
ssserver -c /etc/shadowsocks.json -d start  
ssserver -c /etc/shadowsocks.json -d stop
```

如需开机启动
修改/etc/rc.local，加入以下内容

```
ssserver -c /etc/shadowsocks.json -d start
```

日志
shadowsocks的日志保存在 /var/log/shadowsocks.log

## shadowsocks客户端安装

下载地址：

```
Windows   
https://github.com/shadowsocks/shadowsocks-windows/releases   
  
Mac OS X   
https://github.com/shadowsocks/ShadowsocksX-NG/releases 

Linux   
https://github.com/shadowsocks/shadowsocks-qt5/wiki/Installation   
https://github.com/shadowsocks/shadowsocks-qt5/releases  

IOS   
https://itunes.apple.com/app/apple-store/id1070901416?pt=2305194&ct=shadowsocks.org&mt=8   
https://github.com/shadowsocks/shadowsocks-iOS/releases

Android   
https://play.google.com/store/apps/details?id=com.github.shadowsocks   
https://github.com/shadowsocks/shadowsocks-android/releases
```

## serverspeeder加速安装

注意：serverspeeder加速是可选的，如果你使用vpn测速发现很慢，可以安装试试。

加速前
[![img](http://ou7ri10jn.bkt.clouddn.com/vpn/before_speed_test1.jpeg)](http://ou7ri10jn.bkt.clouddn.com/vpn/before_speed_test1.jpeg)

加速后
[![img](http://ou7ri10jn.bkt.clouddn.com/vpn/after_speed_test1.jpeg)](http://ou7ri10jn.bkt.clouddn.com/vpn/after_speed_test1.jpeg)

下行速度瞬间提升，是不是觉得有点小激动？
一键安装serverspeeder
注：参考[serverspeeder锐速一键破解安装版](https://github.com/91yun/serverspeeder)

```
wget -N --no-check-certificate https://github.com/91yun/serverspeeder/raw/master/serverspeeder.sh && bash serverspeeder.sh
```

如果报内核不支持，可以更换系统内核

```
下载内核安装包  
wget http://ftp.scientificlinux.org/linux/scientific/6.6/x86_64/updates/security/kernel-2.6.32-504.3.3.el6.x86_64.rpm  
  
更换内核  
rpm -ivh kernel-2.6.32-504.3.3.el6.x86_64.rpm --force  
  
重启  
reboot  
  
查看内核版本是否替换成功  
cat /proc/version
```

如果系统内核已更新，再次执行一键安装serverspeeder方法即可。至此serverspeeder安装完毕，快去试试速度是不提升了。

卸载serverspeeder的方法

```
chattr -i /serverspeeder/etc/apx* && /serverspeeder/bin/serverSpeeder.sh uninstall -f
```

### 方法二：搭建l2tp vpn（推荐）

参考[DearTanker’s Blog](http://blog.deartanker.com/)的一键安装方法：

```
wget --no-check-certificate https://raw.githubusercontent.com/teddysun/across/master/l2tp.sh  
chmod +x l2tp.sh  
./l2tp.sh
```

基本上，按交互式命令的提示按回车或者自定义自己的选择即可，然后再次验证ipsec（L2TP）并重启相关服务，否则提示服务器无响应

```
service ipsec restart  
service xl2tpd restart  
ipsec verify
```

如果需要修改或者增加账号密码，可以修改/etc/ppp/chap-secrets

```
账户    l2tpd    密码       *
```

测试成功使用的客户端有，win7+win10自带vpn客户端，andriod自带vpn客户端。

测试成功使用的网络环境有，电信+联通宽带，移动4g无法连接成功，如果哪位朋友在移动网络下有成功的经验，麻烦分享一下。

不使用vpn时，使用[speedtest](http://www.speedtest.net/)的测速结果：
[![img](http://ou7ri10jn.bkt.clouddn.com/vpn/before_speed_test2.png)](http://ou7ri10jn.bkt.clouddn.com/vpn/before_speed_test2.png)

下面是成功连接vpn后，使用speedtest的测速结果，虽然不是非常快，但基本够用。
[![img](http://ou7ri10jn.bkt.clouddn.com/vpn/after_speed_test2.png)](http://ou7ri10jn.bkt.clouddn.com/vpn/after_speed_test2.png)

# 方法三：搭建openvpn（不建议）

vultr面板提供一键安装openvpn的办法，方法是在新建一个vps实例时选择默认安装一个应用程序：
[![img](http://ou7ri10jn.bkt.clouddn.com/vpn/vultr_openvpn.png)](http://ou7ri10jn.bkt.clouddn.com/vpn/vultr_openvpn.png)

具体安装办法参考官方的[一键安装openvpn](https://www.vultr.com/docs/one-click-openvpn)说明。openvpn是使用操作系统的登录账号登录的，所以搭建完openvpn后，你可以参考linux新建用户的办法新建一个用户及修改用户密码，然后使用opevpn提供的客户端或者网页（一般情况下是 <https://your_vps_ip:943/> ）登录即可。

之所以不建议使用openvpn，原因是测速发现比较慢，下载带宽不到1M，而且openvpn貌似不支持移动端登录。

# 方法四：搭建pptpd vpn（不建议）

注：之所以不建议使用pptpd，一方面是pptpd经常被墙，二是容易出问题。

## 安装

```
yum install ppp iptables pptpd
```

## 配置

编辑/etc/pptpd.conf，搜索localip，去掉下面字段前面的#，然后保存退出

```
localip 192.168.0.1  
remoteip 192.168.0.234-238,192.168.0.245
```

注意，pptpd默认支持最大100个连接，每个remoteip分配一个连接，如果remoteip数不够100个，那么默认连接数就会变成remoteip数。如果默认连接数不够的话，就会出现自动断开的情况，比如手机上的vpn连上了，PC端的vpn就会断开。

编辑options.pptpd，搜索ms-dns，去掉搜索到的两行ms-dns前面的#，并修改为下面的字段

```
ms-dns 8.8.8.8  
ms-dns 8.8.4.4
```

编辑/etc/ppp/chap-secrets设置VPN的帐号密码，注意，用户名与密码是区分大小写的

```
用户名 pptpd 密码 *
```

编辑/etc/sysctl.conf，修改内核参数，在末尾添加下面的代码，使内核支持转发

```
net.ipv4.ip_forward=1
```

运行下面的命令使内核修改生效

```
sysctl -p
```

添加下面的iptables转发规则（直接在SSH运行下面命令即可）

```
iptables -t nat -A POSTROUTING -s 192.168.0.0/24 -o eth0 -j MASQUERADE
```

添加转发规则后重启就会失效，Centos 6系统可以使用service iptables save保存配置，而Centos7则可以修改/etc/rc.d/rc.local保存上面的命令，这样开机会自动执行上面的命令。

## 启动

用下面的命令使pptpd开机自动启动

```
chkconfig pptpd on
```

启动pptpd

```
service pptpd start
```

## 使用

使用你的vpn客户端连接即可，如果配置没问题的话，就可以连接成功。
测试成功连接的vpn客户端有，win7+win10自带vpn客户端，andriod自带vpn客户端。但只在宽带网络上连接成功，4g网络连接不成功。

## 排错

如果你的vpn连接不成功，有可能是iptable防火墙的问题，你可以使用下面命令

```
iptables -A INPUT -p tcp --dport 1723 -j ACCEPT  
或者  
iptables -F
```

然后在你的其他电脑使用telnet your_ip 1723测试是否连通。

如果你遇到访问网站偶尔连接上又断的问题，可能是MTU太大导致，可以

```
执行  
iptables -I FORWARD -p tcp --syn -i ppp+ -j TCPMSS --set-mss 1356  
或者修改/etc/ppp/options.pptpd，在文件最后添加  
mtu 1356
```

更多问题，可以打开pptpd的debug日志，根据debug日志的输出上网搜索一步步解决
修改/etc/ppp/options.pptpd
取消下面的注释

```
debug  
dump  
logfile /var/log/pptpd.log（没有则手工修改）
```

> 转载地址: http://blog.noahsun.top/2018/04/07/%E6%90%AD%E5%BB%BA%E8%87%AA%E5%B7%B1%E4%B8%93%E5%B1%9E%E7%9A%84vpn%E2%80%94%E2%80%94Centos%E6%90%AD%E5%BB%BAvpn%E7%9A%84%E5%87%A0%E7%A7%8D%E5%8A%9E%E6%B3%95/