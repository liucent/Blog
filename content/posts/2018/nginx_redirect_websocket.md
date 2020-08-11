---
title: "Nginx配置WebSocket跳转"
description: ""
date: "2018-12-16"
tags:
  - "nginx"
author:
  name: "Vincent Liu"
menu:
  sidebar:
    name: Nginx配置WebSocket跳转
    identifier: Nginx配置WebSocket跳转
    parent: 2018
    weight: -1216
---

Nginx转发WebSocket请求需要添加几个参数来升级http请求为WebSocket请求.
<!--more-->

## 在server中添加需要转发的location参数 {#h2_1}
文件路径:/usr/local/nginx/conf/vhost/www.cloudme.com.conf

```nginx
server{
    #listen
    #ssl
    location = /ray {
        proxy_redirect off;
        proxy_pass http://127.0.0.1:9090;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $http_host;
    }
}
```
