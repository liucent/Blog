---
title: "Guacamole安装部署"
description: ""
date: "2020-08-24"
tags:
  - "guacamole"
  - "ssh"
  - "rdp"
  - "telnet"
author:
  name: "Vincent Liu"
menu:
  sidebar:
    name: Guacamole安装部署
    identifier: Guacamole安装部署
    parent: 2020
    weight: -0824
---

Guacamole是一个开源的web应用, 可以通过浏览器远程访问各种主机, 支持SSH, RDP, TELNET等各种协议. 用来SSH登录多台VPS很是方便.
<!--more-->

目前已经在VPS上部署使用Guacamole很长一段时间了, 最开始是直接使用的xml配置文件, 后面改成了数据库配置. VPS的系统是Ubuntu 18.04, 所以下面的部署方法理论上在Ubuntu上是没问题的. 另外这里使用的是原生安装的方式, 不是Docker方式.

## 1. Guacamole Server部署 {#h2_1}

首先需要编译安装服务端. 但是在此之前, 需要安装好一系列依赖库. 这些库虽然可以根据需要支持的协议(SSH, RDP, TELNET等)选择性的安装, 但还是建议全部安装.

### 1.1. 安装依赖库 {#h3_1_1}

a. 必选依赖库

| Library name | Features | Details |
| ---- | ---- | ---- |
| Cairo | libcairo2-dev |  <code class="language-bash">sudo apt install libcairo2-dev</code> |
| libjpeg-turbo | libjpeg-turbo8-dev |  <code class="language-bash">sudo apt install libjpeg-turbo8-dev</code> |
| libpng | libpng12-dev/libpng-dev |  <code class="language-bash">sudo apt install libpng-dev</code> <br> Ubuntu 16.04之后替换为了libpng-dev |
| libtool | libtool-bin |  <code class="language-bash">sudo apt install libtool-bin</code> |
| OSSP UUID | libtool-bin |  <code class="language-bash">sudo apt install libossp-uuid-dev</code> |

b. 可选依赖库

| Library name | Features | Details |
| ---- | ---- | ---- |
| FFmpeg | libavcodec-dev, libavformat-dev, libavutil-dev, libswscale-dev | <code class="language-bash">sudo apt install libavcodec-dev libavformat-dev libavutil-dev libswscale-dev</code> |
| FreeRDP | freerdp2-dev | <code class="language-bash">sudo apt install freerdp2-dev</code> <br> (Ubuntu 16.04及以前版本安装freerdp2没有成功， 估计是只支持freerdp1. Guacamole 1.0及以后版本也是支持freerdp2, 所以还是建议升级到Ubuntu 18.04) |
| Pango | libpango1.0-dev | <code class="language-bash">sudo apt install libpango1.0-dev</code> |
| libssh2 | libssh2-1-dev | <code class="language-bash">sudo apt install libssh2-1-dev</code> |
| libtelnet | libtelnet-dev | <code class="language-bash">sudo apt install libtelnet-dev</code> |
| libVNCServer | libvncserver-dev | <code class="language-bash">sudo apt install libvncserver-dev</code> |
| libwebsockets | libwebsockets-dev | <code class="language-bash">sudo apt install libwebsockets-dev</code> |
| PulseAudio | libpulse-dev | <code class="language-bash">sudo apt install libpulse-dev</code> |
| OpenSSL | libssl-dev | <code class="language-bash">sudo apt install libssl-dev</code> |
| libvorbis | libvorbis-dev | <code class="language-bash">sudo apt install libvorbis-dev</code> |
| libwebp | libwebp-dev | <code class="language-bash">sudo apt install libwebp-dev</code> |

### 1.2. 下载源码 {#h2_1_2}

访问[release页面](http://guacamole.apache.org/releases/)下载最新版本并解压

```bash
# 下载
wget https://mirror.bit.edu.cn/apache/guacamole/1.2.0/source/guacamole-server-1.2.0.tar.gz
# 解压
tar -xzf guacamole-server-1.2.0.tar.gz
# 进入到源码文件夹
cd guacamole-server-1.2.0/
```

### 1.3. 编译源码 {#h2_1_3}

首先做configure检查环境, 查看系统环境和依赖库是否满足编译要求.

```bash
./configure --with-init-dir=/etc/init.d
```

若环境正常会输出如下检查结果.

```bash
------------------------------------------------
guacamole-server version 1.2.0
------------------------------------------------

   Library status:

     freerdp2 ............ yes
     pango ............... yes
     libavcodec .......... yes
     libavformat ......... yes
     libavutil ........... yes
     libssh2 ............. yes
     libssl .............. yes
     libswscale .......... yes
     libtelnet ........... yes
     libVNCServer ........ yes
     libvorbis ........... yes
     libpulse ............ yes
     libwebsockets ....... yes
     libwebp ............. yes
     wsock32 ............. no

   Protocol support:

      Kubernetes .... yes
      RDP ........... yes
      SSH ........... yes
      Telnet ........ yes
      VNC ........... yes

   Services / tools:

      guacd ...... yes
      guacenc .... yes
      guaclog .... yes

   Init scripts: /etc/init.d
   Systemd units: no

Type "make" to compile guacamole-server.
```

编译安装源码并配置

```bash
# 编译
make
...
# 安装
make install
...
# 配置
ldconfig
```

至此, 服务端就安装完成了, 也会新增一个guacd的service.


## 2. Guacamole Client部署 {#h2_2}

客户端需要用到servlet container. 这里直接使用Tomcat 9版本.

### 2.1. 安装Tomcat {#h3_2_1}

[Tomcat官网下载页面](https://tomcat.apache.org/download-90.cgi)下载tomcat包.

```bash
# 下载
wget https://mirror.bit.edu.cn/apache/tomcat/tomcat-9/v9.0.37/bin/apache-tomcat-9.0.37.tar.gz
# 解压
tar -xvf apache-tomcat-9.0.37.tar.gz
# 重命名
mv apache-tomcat-9.0.37 tomcat
```

Tomcat需要java环境, 若系统中未安装jdk需要运行如下命令安装.

```bash
sudo apt install default-jdk
```

安装完成后需要设置环境变量及其他配置, 不过Guacamole也需要设置环境变量, 所以这里暂时略过, 后面再一起设置.

### 2.2. 部署客户端 {#h3_2_2}

客户端使用mvn package编译源码, 不过官网有发布编译好的war包, 所以直接下载war包即可. 下载链接同样是在release页面.

```bash
# 下载
wget https://mirror.bit.edu.cn/apache/guacamole/1.2.0/binary/guacamole-1.2.0.war
# 重命名
mv guacamole-1.2.0.war guacamole.war
```

下载好后放在Tomcat下的webapps文件夹里即可.

```bash
mv guacamole.war tomcat/webapps/
```

至此, 客户端也已经部署完了, 但还需要做进一步的配置.


## 3. 配置 {#h2_3}

配置主要分为四大类: 环境变量, Tomcat配置, nginx设置, Guacamole配置.

### 3.1. 环境变量 {#h3_3_1}

这里需要设置jdk和guacamole两个环境变量. 环境变量可以直接添加到/etc/profile文件尾部. 如果jdk版本不一致, 则需要按实际版本填写路径, 如果/etc/guacamole文件不存在, 需要mkdir新建一个.

```bash
# 在/etc/profile文件末尾添加如下变量
#set java env
export JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64

#set guacamole env
export GUACAMOLE_HOME=/etc/guacamole
```

设置完后需要运行如下命令立即生效.

```bash
source /etc/profile
```

### 3.2. Tomcat配置 {#h3_3_2}

虽然Tomcat可以直接通过外网访问, 不过一般都是用nginx反代. 所以这里需要设置禁止外网访问Tomcat. 在tomcat/conf/server.xml文件里的Host节点下添加如下字段.

```xml
<Valve className="org.apache.catalina.valves.RemoteAddrValve" allow="127.0.0.1" denyStatus="403" directory="logs"
	       prefix="localhost_access_log" suffix=".txt"
	       pattern="%h %l %u %t &quot;%r&quot; %s %b" />
```

另外, 如果服务器的内存比较小则需要限制下Tomcat的内存占用. 在tomcat/bin/catalina.sh文件开头添加如下代码.

```bash
# 根据实际情况设置内存大小.
JAVA_OPTS="-server -Xms64m -Xmx256m"
```

### 3.3. nginx设置 {#h3_3_3}

设置反代, Tomcat默认端口8080. 在nginx站点配置文件里设置如下跳转.

```nginx
location / {
            proxy_pass http://127.0.0.1:8080/guacamole/;
            proxy_buffering off;
            proxy_http_version 1.1;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection $http_connection;
        }
```

### 3.4. Guacamole配置 {#h3_3_4}

这里的配置既可以简单的使用文件也可以使用数据库. 二者选择其一就可以了.

#### 3.4.1. 文件配置 {#h3_3_4_1}

在$GUACAMOLE_HOME路径下新建两个文件guacamole.properties和user-mapping.xml.

在guacamole.properties中设置主机名和端口号, 默认本机localhost和4822端口. 并设置好user-mapping.xml的路径.

```properties
guacd-hostname: localhost
guacd-port: 4822
user-mapping: /etc/guacamole/user-mapping.xml
auth-provider: net.sourceforge.guacamole.net.basic.BasicFileAuthenticationProvider
basic-user-mapping: /etc/guacamole/user-mapping.xml
```

在user-mapping.xml设置远程登录的主机信息和通信协议. authorize字段是页面登录用户名密码. 每个connect对应设置一个远程连接配置.

```xml
<user-mapping>
  <authorize username="user" password="password">
    <connection name="demo_ssh">
      <protocol>ssh</protocol>
      <param name="hostname">server_ip_</param>
      <param name="port">22</param>
    </connection>
    <connection name="demo_rdp">
      <protocol>rdp</protocol>
      <param name="hostname">server_ip</param>
      <param name="port">3389</param>
    </connection>
  </authorize>
</user-mapping>
```

#### 3.4.2. 数据库配置 {#h3_3_4_2}

Guacamole支持多种数据库(MySQL, PostgreSQL, or SQL Server), 这里习惯性的使用MySQL. 首先需要下载两个jar文件. 一个是Guacamole jdbc扩展包, 一个是MySQL的Connector包.

同样在release页面下载Guacamole jdbc扩展包, 然后把包里的guacamole-auth-jdbc-mysql-1.2.0.jar文件和schema文件夹放到$GUACAMOLE_HOME/extensions下, 如果没有extensions文件夹需要新建一下.

```bash
# 下载
wget https://mirrors.bfsu.edu.cn/apache/guacamole/1.2.0/binary/guacamole-auth-jdbc-1.2.0.tar.gz
# 解压
tar -xvf guacamole-auth-jdbc-1.2.0.tar.gz
# 移动文件
mv guacamole-auth-jdbc-mysql-1.2.0.jar $GUACAMOLE_HOME/extensions/
mv schema $GUACAMOLE_HOME/extensions/
```

在[MySQL页面](https://dev.mysql.com/downloads/connector/j/)下载Connector包, 选择Platform Indenpent版本. 并将mysql-connector-java-5.1.49-bin.jar文件放到$GUACAMOLE_HOME/lib文件夹下, 如果没有lib文件夹需要新建一下.

```bash
# 下载
wget https://cdn.mysql.com//Downloads/Connector-J/mysql-connector-java-5.1.49.tar.gz
# 解压
tar -xvf mysql-connector-java-5.1.49.tar.gz
# 移动文件
mv mysql-connector-java-5.1.49-bin.jar $GUACAMOLE_HOME/lib/
```

新建一个数据库和用户, 并给用户授权.

```bash
mysql -u root -p
...
# 新建数据库, guacamole_db为数据库名
mysql> CREATE DATABASE guacamole_db;
# 新建用户, user/password为用户名/密码
mysql> CREATE USER 'user'@localhost IDENTIFIED BY 'password';
mysql> CREATE USER 'user'@'127.0.0.1' IDENTIFIED BY 'password';
# 给用户授权, guacamole_db数据库的全部权限
mysql> GRANT ALL PRIVILEGES ON guacamole_db.* TO 'user'@localhost;
mysql> GRANT ALL PRIVILEGES ON guacamole_db.* TO 'user'@'127.0.0.1';
mysql> FLUSH PRIVILEGES;
```

在配置文件里添加数据库连接信息. 这里和文件配置方式一样也会用到$GUACAMOLE_HOME/guacamole.properties文件, 但是只需要配置数据库连接并不需要配置user-mapping.xml远程连接配置文件. 其中user和password就是先前创建的数据库用户密码. MySQL默认本机localhost和3306端口.

```properties
mysql-hostname: localhost
mysql-port: 3306
mysql-database: guacamole_db
mysql-username: user
mysql-password: password
```

运行Guacamole自带的初始化SQL脚本, 脚本先前已经被放在了$GUACAMOLE_HOME/extensions/schema文件夹里, 里面包含了两个文件, 一个是创建表: 001-create-schema.sql, 一个是创建默认用户: 002-create-admin-user.sql, 运行这两个文件.

```bash
cat schema/*.sql | mysql -u root -p guacamole_db
```

## 4. 启动服务 {#h2_4}

至此, 经过上一步的文件配置或者数据库配置就可以启动服务了. 如果是文件配置方式就可以直接用配置文件里设置的用户名密码登录页面开始使用远程连接了, 但是如果是数据库连接方式则需要用默认的guacadmin/guacadmin用户名密码登录页面, 然后新建个自己的用户并将该默认用户删掉, 防止安全问题. 登录后也就可以自行创建各种远程连接设置了.

```bash
# 启动guacd服务
service guacd start

# 启动Tomcat
./tomcat/bin/startup.sh
```

等待服务启动正常后就可以访问nginx配置的站点, 这时候站点也就重定向到了Guacamole客户端.

## 5. 更新历史 {#h2_5}

| Version | Detail | Date |
| ---- | ---- | ---- |
| 1.0 | 初版 | 2020-08-24 |
| 1.1 | 添加依赖库 | 2020-09-03 |