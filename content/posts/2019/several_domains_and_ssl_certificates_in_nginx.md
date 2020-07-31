---
title: "Nginx配置多域名SSL证书"
description: ""
date: "2019-11-23"
tags:
  - "nginx"
  - "ssl"
author:
  name: "Vincent Liu"
---

一直以来都只在nginx中使用过单域名以及单个SSL证书的配置. 前些天注册了个新域名并把新域名套上了cloudflare. 于是就想把新老域名都配置上同一个nginx站点, 也顺便对比下套与不套cloudflare的区别. 在网上搜索了下配置其实还是挺简单的, 只要找到原站点的配置文件并把原有的server配置再添加一个同样的并小修改下即可.
<!--more-->

## 原有站点配置文件 {#h2_1}
在nginx的配置文件路径下找到原有站点的配置文件(一般在该路径下/usr/local/nginx/conf/vhost/). 找到类似如下的server配置项.

```nginx
server
    {
        listen 443 ssl http2;
        listen [::]:443 ssl http2;
        server_name www.aaa.com ;
        index index.html index.htm index.php default.html default.htm default.php;
        root  /home/wwwroot/www.aaa.com;

        ssl_certificate /etc/letsencrypt/live/www.aaa.com/fullchain.pem;
        ssl_certificate_key /etc/letsencrypt/live/www.aaa.com/privkey.pem;
        #...
    }
```

## 添加新域名和证书 {#h2_2}
在上述server配置项下再添加一个相同的server配置项并把相应域名改称新域名, 证书路径也改成新域名证书的路径. server中的其他配置, 特别是root的配置千万不要修改.

```nginx
server
    {
        listen 443 ssl http2;
        listen [::]:443 ssl http2;
        server_name www.bbb.com; #更改成新域名
        index index.html index.htm index.php default.html default.htm default.php;
        root  /home/wwwroot/www.aaa.com; #不要修改

        #更改成新域名证书的路径
        ssl_certificate /etc/letsencrypt/live/www.bbb.com/fullchain.pem;
        ssl_certificate_key /etc/letsencrypt/live/www.bbb.com/privkey.pem;
        
        #...
    }
```

## 重启nginx {#h2_3}
修改并保存站点配置文件后直接重启nginx, 然后通过新老域名都可以访问站点了. 另外如果套了cloudflare就需要在cloudflare上设置成full或者full(strict).

## 更新历史 {#h2_4}

| Version | Detail | Date |
| ---- | ---- | ---- |
| 1.0 | 初版 | 2019-11-23 |