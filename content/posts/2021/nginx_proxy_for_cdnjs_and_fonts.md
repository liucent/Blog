---
title: "Nginx反代cdnjs和fonts"
description: ""
date: "2021-03-26"
tags:
  - "nginx"
  - "proxy"
  - "cdnjs"
  - "fonts"
author:
  name: "Vincent Liu"
menu:
  sidebar:
    name: Nginx反代cdnjs和fonts
    identifier: Nginx反代cdnjs和fonts
    parent: 2021
    weight: -0326
---

最近有点不想用公共cdn, 于是就着手用自己手头的VPS里的Nginx反代Cloudflare cdnjs和Google fonts. 方法其实并不难, 特别是服务器上已经有Nginx环境的.
<!--more-->

首先，毕竟是要用https的，所以准备好自己的域名以及ssl证书. 这里假设用cdnjs.yours.com反代cdnjs.cloudflare.com, 用fonts.yours.com反代fonts.googleapis.com和fonts.gstatic.com. 其中fonts.googleapis.com是返回各个字体的css文件, fonts.gstatic.com则是css文件里面标识的下载单个字体文件的域名.

## 1. Nginx反代cdnjs {#h2_1}

假设ssl证书都已经设置好了, 在nginx的配置文件里的http结构添加cache信息, 如果没有http结构则只需添加在server结构外即可.

```nginx
# cache路径, 过期时间, 最大大小
proxy_cache_path  /home/wwwroot/cdnjs.yours.com levels=1:2 keys_zone=cache_cdnjs:20m inactive=1d max_size=100m;
```

在server结构里添加location反代设置.

```nginx
location / {
    # 设置referers白名单, 只有*.yours.com发过来的请求才会做反代, 其余请求直接返回403
    valid_referers none blocked server_names  *.yours.com ;
    if ($invalid_referer) {
        return 403;
    }

    proxy_pass_header Server;
    proxy_set_header Host cdnjs.cloudflare.com;
    proxy_redirect off;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Scheme $scheme;
    proxy_pass https://cdnjs.cloudflare.com;
    proxy_cache cache_cdnjs;

    # cache过期时间1天
    proxy_cache_valid  200 304 1d;
    proxy_cache_key $host$uri$is_args$args;
    expires 1d;
}
```

## 2. Nginx反代fonts {#h2_2}

在nginx的配置文件里的http结构添加cache信息, 如果没有http结构则只需添加在server结构外即可.

```nginx
# cache路径, 过期时间, 最大大小
proxy_cache_path  /home/wwwroot/fonts.yours.com levels=1:2 keys_zone=cache_fonts:20m inactive=1d max_size=100m;
```

反代Google fonts会稍微麻烦点. 因为字体请求流程是这样的:

1. 发送请求到fonts.googleapis.com/css, Google会返回一个css文件. 文件里包含不同字号大小的字体文件路径url(fonts.gstatic.com).
2. 根据不同字号大小的url请求相应的单字体文件.

由此可见我们不仅先要反代请求到fonts.googleapis.com还要把返回的css文件里的fonts.gstatic.com全部替换为我们自己的域名fonts.yours.com.

在server结构里添加location反代设置.

```nginx
# 反代字体请求发到fonts.googleapis.com
location /css {
    valid_referers none blocked server_names  *.yours.com ;
    if ($invalid_referer) {
        return 403;
    }

    # 将返回的css文件里的fonts.gstatic.com替换成自己的域名fonts.yours.com
    sub_filter 'fonts.gstatic.com' 'fonts.yours.com';
    sub_filter_once off;

    sub_filter_types text/css;
    proxy_pass_header Server;
    proxy_set_header Host fonts.googleapis.com;
    proxy_set_header Accept-Encoding '';
    proxy_redirect off;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Scheme $scheme;
    proxy_pass https://fonts.googleapis.com;
    proxy_cache cache_fonts;
    proxy_cache_valid  200 304 1d;
    proxy_cache_key $host$uri$is_args$args;
    expires 1d;
}

location /icon {
    valid_referers none blocked server_names  *.yours.com ;
    if ($invalid_referer) {
        return 403;
    }
    sub_filter 'fonts.gstatic.com' 'fonts.yours.com';
    sub_filter_once off;
    sub_filter_types text/css;
    proxy_pass_header Server;
    proxy_set_header Host fonts.googleapis.com;
    proxy_set_header Accept-Encoding '';
    proxy_redirect off;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Scheme $scheme;
    proxy_pass https://fonts.googleapis.com;
    proxy_cache cache_fonts;
    proxy_cache_valid  200 304 1d;
    proxy_cache_key $host$uri$is_args$args;
    expires 1d;
}

# 反代单个字体文件请求到fonts.gstatic.com
location / {
    valid_referers none blocked server_names  *.yours.com ;
    if ($invalid_referer) {
        return 403;
    }
    proxy_pass_header Server;
    proxy_set_header Host fonts.gstatic.com;
    proxy_redirect off;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Scheme $scheme;
    proxy_pass https://fonts.gstatic.com;
    proxy_cache cache_fonts;
    proxy_cache_valid  200 304 1d;
    proxy_cache_key $host$uri$is_args$args;
    expires 1d;
}
```

## 3. referer白名单 {#h2_3}

原以为上一节代码中的valid_referers写法是只让*.yours.com域名才能反代的白名单. 但是自己浏览器直接访问cdnjs.yours.com请求却是可以被成功反代的. 于是上网查了查原来白名单的写法是这样的.

```nginx
# none: referer为空也加入白名单, 意味着浏览器直接发请求也会成功返回.
# blocked: 非http/https协议也加入白名单, 即其他协议请求也会成功返回.
valid_referers none blocked server_names  *.yours.com ;
```

所以如果只允许自己的域名能返回成功, 那就只需要加上自己的域名即可.

```nginx
valid_referers server_names  *.yours.com ;
```

另外为方便本地调试, 也可以在后面加上localhost. 这样本地起的工程也可以返回成功.

```nginx
valid_referers server_names  *.yours.com localhost ;
```

## 4. 更新历史 {#h2_4}

| Version | Detail | Date |
| ---- | ---- | ---- |
| 1.0 | 初版 | 2021-03-26 |
| 1.1 | 修改错误 | 2021-03-27 |
| 1.2 | 白名单说明 | 2021-03-28 |