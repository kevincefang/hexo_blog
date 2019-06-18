---
title: 安装并配置shadowsocks-libev（yum源方式）
date: 2019-06-17
tags: 
- vpn
categories: vpn
---

##### 安装常用工具

```shell
yum install git wget vim -y
```

##### 安装编译工具

* CentOS

  ```shell
  yum install epel-release gcc gettext autoconf libtool automake make asciidoc xmlto c-ares-devel libev-devel pcre-devel -y
  ```

* Ubuntu

  ```shell
  sudo apt-get install --no-install-recommends gettext build-essential autoconf libtool libpcre3-dev asciidoc xmlto libev-dev libc-ares-dev automake -y
  ```

##### 单独安装 libsodium

libsodium 是一个先进而且易用的加密库。 主要用于加密、解密、签名和生成密码哈希。 Note：可以点击这里查看 [Libsodium](https://download.libsodium.org/libsodium/releases/) 的最新版本。

```shell
export LIBSODIUM_VER=1.0.17
wget https://download.libsodium.org/libsodium/releases/libsodium-$LIBSODIUM_VER.tar.gz
tar xvf libsodium-$LIBSODIUM_VER.tar.gz
pushd libsodium-$LIBSODIUM_VER
./configure --prefix=/usr && make
sudo make install
popd
sudo ldconfig
```

###### 单独安装 mbedtls

SSL和TLS

```shell
export MBEDTLS_VER=2.6.0
wget https://tls.mbed.org/download/mbedtls-$MBEDTLS_VER-gpl.tgz
tar xvf mbedtls-$MBEDTLS_VER-gpl.tgz
pushd mbedtls-$MBEDTLS_VER
make SHARED=1 CFLAGS=-fPIC
sudo make DESTDIR=/usr install
popd
sudo ldconfig
```

##### 安装 simple-obfs（已被 v2ray-plugin 取代 ）

>混淆插件，如果不需要可以跳过，隐蔽性:v2ray-plugin 优于 simple-obfs；但 V2ray-plugin 延迟较高。

* 安装编译必须的软件

  ```shell
  yum install zlib-devel openssl-devel -y
  ```

* 安装 simple-obfs

  ```shell
  git clone https://github.com/shadowsocks/simple-obfs.git
  cd simple-obfs
  git submodule update --init --recursive
  ./autogen.sh
  ./configure && make
  make install
  ```

##### 安装 v2ray-plugin （simple-obfs 和 v2ray-plugin 不能同时开启，安装其中一个即可）

> 请先安装 SS 后，再来看此连接内容

避免文章内容过长，影响浏览体验，故单独提取到了 gist 上（可能得科学上网才能访问） [v2ray-plugin 安装](https://gist.github.com/Shuanghua/c9c448f9bd12ebbfd720b34f4e1dd5c6)

##### 安装 SS

> 前面一些环境搭建好了，现在正式安装我们的主角。

(1)下载源码

```shell
git clone https://github.com/shadowsocks/shadowsocks-libev.git
cd shadowsocks-libev
git submodule update --init --recursive
```

(2)编译源码并安装

```shell
cd shadowsocks-libev //确保是在 ss 目录中执行编译
./autogen.sh && ./configure && make
sudo make install // root 权限下安装
```

(3)创建账号配置文件

```shell
mkdir -p /etc/shadowsocks-libev
vim /etc/shadowsocks-libev/config.json
```

先按 **i** 键进入编辑模式，然后复制下面内容到文件中，自行替换相应地方。

```json
{
    "server":"0.0.0.0",
    "port_password":{
     "port1":"pass1",
     "port2":"pass2"
     },
    "timeout":300,
    "method":"xchacha20-ietf-poly1305",
    "mode":"tcp_and_udp",
   "fast_open": true,
   "workers": 5,
   "plugin":"obfs-server",
   "plugin_opts":"obfs=http;fast-open"
}
```

> 上面包含了 simple-obfs 混淆插件，如果没有安装该插件，请去掉最后两行。

##### 开放防火墙端口

上面设置了一个端口后，现在让防火墙接受该端口号（允许该端口的流量通过防火墙）

* 启动 Firewalld 防火墙

* ```shell
  systemctl enable firewalld
  systemctl start firewalld
  ```

* 添加开放端口(就是上面配置中的端口号)

  ```shell
  firewall-cmd --permanent --zone=public --add-port=port1/tcp
  firewall-cmd --permanent --zone=public --add-port=port2/udp
  ```

* 使端口生效

  ```shell
  firewall-cmd --reload
  ```

* 查看已开放的端口（可跳过）

  ```shell
  firewall-cmd --list-all
  ```

##### 开启服务

到此已经完成所有安装，启动 ss 有两种方式，任选一种方式

* 是通过使用创建好的配置文件来启动

单个端口号配置：nohup ss-server -c /etc/shadowsocks-libev/config.json &

多个端口号配置：nohup ss-manager -c /etc/shadowsocks-libev/config.json &

* 是通过将信息写在命令行中启动

```shell
ss-server -s 服务器IP -p 端口 -k 密码 -m chacha20-ietf-poly1305 -u -l 1080 -t 600 --fast-open true --plugin obfs-server --plugin-opts obfs=http;fast-open
```

##### 加速优化(可选)

> 加速可选 bbr 和 kcptun

###### Google BBR

google bbr 加速优化教程链接：[CentOS 安装 BBR](https://www.vultr.com/docs/how-to-deploy-google-bbr-on-centos-7)

###### Kcptun

去 github 下载对应的版本软件：<https://github.com/xtaci/kcptun/releases>

我是 CentOS7 64 位的系统，所以下载的是 kcptun-linux-amd64-20180305.tar.gz，下载完解压，把 server_linux_amd64 这个文件传到 vps （或者直接通过 vps 下载），然后把该文件移动到 /usr/local/kcptun/ 目录下，如果没有执行权限，记得给该文件赋予执行权限。

创建 kcptun 的配置文件

```
vim /usr/local/kcptun/server-config.json
```

内容如下：

```
{
  "listen": ":29900",
  "target": "127.0.0.1:端口",
  "key": "密码",
  "crypt": "aes",
  "mode": "fast2",
  "mtu": 1350,
  "sndwnd": 512,
  "rcvwnd": 512,
  "datashard": 10,
  "parityshard": 3,
  "dscp": 0,
  "nocomp": false,
  "quiet": false,
  "pprof": false
}
```

后台启动Kcptun server服务

```
nohup /usr/local/kcptun/server_linux_amd64 -c /usr/local/kcptun/server-config.json &
```





