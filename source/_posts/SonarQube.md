---
title: SonarQube 代码质量管理平台的搭建
date: 2017-06-15 10:03:44
tags:
- SonarQube
- java
categories: SonarQube
---

##### 背景：
> 由于公司项目的代码量比较大，框架使用也比较混乱，还有项目中没有用到类似spring管理的框架，例如获取数据库的session等操作，都是需要手动操作的，操作多了难免会造成session的未关闭等问题。上周就是因为session的为关闭导致应用运行一段时间后，莫名的不再接受任何请求和处理，也没有任何的异常信息，就像卡住一样。因此排查问题浪费了很多时间，最后检查代码才发现有一处因为数据库的session未关闭的原因造成的。其实如果用到了代码检查工具，这些细节问题是完全可以避免的。
> 
	这里推荐使用开源的SonarQube质量管理平台工具
	
##### 下面是搭建步骤：
##### 1、准备环境
* jdk1.8 
* mysql5.6+

##### 2、 安装mysql并创建数据库sonar

```
# mysql -u root -p

# mysql> CREATE DATABASE sonar CHARACTER SET utf8 COLLATE utf8_general_ci;

# mysql> CREATE USER 'sonar' IDENTIFIED BY 'sonar';

# mysql> GRANT ALL ON sonar.* TO 'sonar'@'%' IDENTIFIED BY 'sonar';

# mysql> GRANT ALL ON sonar.* TO 'sonar'@'localhost' IDENTIFIED BY 'sonar';

# mysql> FLUSH PRIVILEGES;


```

##### 3、 下载安装包
###### 服务端工具：
sonarqube:http://www.sonarqube.org/downloads/
###### 客户端工具
sonar-runner:http://repo1.maven.org/maven2/org/codehaus/sonar/runner/sonar-runner-dist/2.3/sonar-runner-dist-2.4.zip

##### 4、 安装SonarQube
> 第一步： 将下载的SonarQube解压安装到/usr/local 目录下。具体步骤如下：

```
# wget -c http://downloads.sonarsource.com/sonarqube/sonarqube-5.1.1.zip

# unzip sonarqube-5.1.1.zip

# mv sonarqube-5.1.1 /usr/local/
```

> 第二步： 配置环境变量

```
vim + /etc/profile`

添加
SONAR_HOME=/usr/local/sonarqube-5.1.1 

export SONAR_HOME

保存退出并使配置生效
source /etc/profile

```

> 第三步： 修改配置文件sonar.properties

```
# vim /usr/local/sonarqube-5.1.1/conf/sonar.properties
打开后，找到
sonar.host.url=http://192.168.1.168:9000
sonar.jdbc.username=root
sonar.jdbc.password=123456
sonar.jdbc.url=jdbc:mysql://localhost:3306/sonar?useUnicode=true&characterEncoding=utf8&rewriteBatchedStatements=true&useConfigs=maxPerformance
sonar.web.host=0.0.0.0
sonar.web.context=/
sonar.web.port=9000

```

> 第四步： 修改配置文件wrapper.conf
> 
`wrapper.java.command=/usr/local/sonar/jdk/bin/java `

注意：把wrapper.conf中的wrapper.java.command修改成jdk1.8路径
否则会找系统自带jdk版本的命令执行，启动的时候可能报错 , bin后面还需要加上/java

> 第五步： 启动服务

`# /usr/local/sonarqube-5.1.1/bin/linux-x86-64/sonar.sh start`

 &nbsp;&nbsp;另外，启动/停止/重启命令如下： 

```
# ./sonar.sh start   启动服务 
# ./sonar.sh stop    停止服务 
# ./sonar.sh restart 重启服务

查看启动日志:
# tail -f /usr/local/sonarqube-5.6.6/logs/sonar.log

关闭命令:
# ./sonar.sh stop

登录：http://localhost:9000

默认密码:admin/admin

```

> 第六步： 访问SonarQube Web管理界面。如果能够看到这个界面证明SonarQube安装成功啦。
 
> 注：我这里访问的地址是：http://192.168.1.168:9000

##### 5、 安装SonarQube Runner
> 第一步：将下载的http://repo1.maven.org/maven2/org/codehaus/sonar/runner/sonar-runner-dist/2.4/sonar-runner-dist-2.4.zip解压后放到/usr/local目录下。具体步骤如下：
 
```
# wget -c http://repo1.maven.org/maven2/org/codehaus/sonar/runner/#sonar-runner-dist/2.4/sonar-runner-dist-2.4.zip
# unzip sonar-runner-dist-2.4.zip
# mv sonar-runner-2.4/ /usr/local/

```
> 第二步：配置环境变量
 
```
# vim + /etc/profile

添加
SONAR_RUNNER_HOME=/usr/local/sonar-runner-2.4/
PATH=.:$SONAR_RUNNER_HOME/bin 
export SONAR_RUNNER_HOME

保存并退出 
# source /etc/profile

```
> 第三步：配置sonar-runner.properties
 
```
# vim /usr/local/sonar-runner-2.4/conf/sonar-runner.properties

找到
sonar.host.url=http://192.168.1.168
sonar.jdbc.url=jdbc:mysql://localhost:3306/sonar?useUnicode=true&amp;characterEncoding=utf8
sonar.jdbc.username=sonar
sonar.jdbc.password=sonar
sonar.login=admin
sonar.password=admin
将前面的#去掉

```
> 第四步：运行sonar-runner分析源代码
 
Sonar官方已经提供了非常全的代码样例便于新手入门用。

下载地址：https://github.com/SonarSource/sonar-examples/archive/master.zip

下载后使用unzip解压。进入java执行sonar-runner命令即可。操作命令如下：

```
# wget -c https://github.com/SonarSource/sonar-examples/archive/master.zip
# unzip master.zip
# cd sonar-examples-master/projects/languages/java/maven/
# sonar-runner
```
如果能够看到下面的输出信息，证明你的SonarQube Runner安装并配置正确啦。

```
INFO: ------------------------------------------------------------------------
INFO: EXECUTION SUCCESS
INFO: ------------------------------------------------------------------------
Total time: 2:59.167s
Final Memory: 17M/204M
INFO: ------------------------------------------------------------------------
```

> 第五步：看看SonarQube的Web界面，是否已经可以看到分析的结果啦。

##### 6、扫描项目

> (1) maven环境 (推荐方式，比较方便，扫描完了自动上传结果至sonar服务器中)
 
Maven仓库中就有SonarQube Scanner工具的插件，只要在$M2_HOME/conf/setting.xml文件中添加如下配置

```
<pluginGroups>

         <pluginGroup>org.sonarsource.scanner.maven</pluginGroup>

</pluginGroups>

<profile>

     <id>sonar</id>

<activation>

      <activeByDefault>true</activeByDefault>

</activation>

<properties>

     <sonar.host.url>http://192.168.1.168:9000/sonarqube</sonar.host.url>

</properties>

</profile>
```
配置完成后，在项目中，执行mvn sonar:sonar，SonarQube Scanner会自动扫描，根据pom.xml文件得出项目相关信息，不需要自定义sonar-project.properties。扫描完成后就会上传只Sonarqube服务器中。稍后，登陆服务器中就可以看到分析结果了。

>(2) 手动扫描

```
打开要进行代码分析的项目根目录，新建sonar-project.properties文件

sonar.projectKey=my:task

# this is the name displayed in the SonarQube UI

sonar.projectName=My task

sonar.projectVersion=1.0

sonar.projectDescription= task 定时任务调度

# Path is relative to the sonar-project.properties file. Replace "\" by "/" on Windows.

# Since SonarQube 4.2, this property is optional if sonar.modules is set.

# If not set, SonarQube starts looking for source code from the directory containing

# the sonar-project.properties file.

#sources是源文件所在的目录

sonar.sources=master/src,slave/src

sonar.binaries=WebRoot/WEB-INF/classes

# Encoding of the source code. Default is default system encoding

sonar.language=java

sonar.my.property=value

sonar.sourceEncoding=UTF-8

在项目跟目录执行  输入命令：sonar-runner
```


##### 7、SonarQube默认是没有安装中文语言包的。如何安装语言包呢？进入SonarQube插件目录，下载语言包即可。步骤如下

```
# cd /usr/local/sonarqube-5.1.1/extensions/plugins
# wget -c http://repo1.maven.org/maven2/org/codehaus/sonar-plugins/l10n/sonar-l10n-zh-plugin/1.8/sonar-l10n-zh-plugin-1.8.jar
```
> 这是中文语言包的源码地址：https://github.com/SonarCommunity/sonar-l10n-zh