---
title: "Nginx强制http跳转https"
description: ""
date: "2019-01-16"
tags:
  - "nginx"
author:
  name: "Vincent Liu"
---

配置nginx强制http请求跳转到https, 同时也可以配置强制跳转www.
<!--more-->

## 强制http跳转https {#h2_1}
修改相应站点的vhost配置文件, 文件路径:/usr/local/nginx/conf/vhost/www.cloudme.com.conf.

```nginx
server
    {
        listen 80;
        #listen [::]:80;
        server_name www.cloudme.com cloudme.com;
        #Redirect http to https
        return 301 https://$host$request_uri;
    }
```

## 强制非www跳转www {#h2_2}
上述修改server配置的方法, 理论上可以做到强制跳转www的. 但是对于WordPress还需要在后台Settings里面把站点配置成https://www.cloudme.com.

```none
Settings->General->WordPress Address (URL)
Settings->General->Site Address (URL)
```

