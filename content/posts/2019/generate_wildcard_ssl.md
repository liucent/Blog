---
title: "手动生成SSL通配符证书"
description: ""
date: "2019-06-08"
tags:
  - "ssl"
author:
  name: "Vincent Liu"
---

以前一直使用的LNMP自动生成的证书, 也没在意生成的是不是通配符证书. 前段时间折腾私人网盘时, 竟然触发了Let's Encrypt的申请证书次数限制. 遂来手动生成通配符SSL证书.
<!--more-->

## 安装Certbot工具 {#h2_1}
Certbot是Let's Encrypt官方推荐生成证书的工具. 在Terminal中运行以下命令来安装Certbot.

```bash
wget https://dl.eff.org/certbot-auto
sudo mv certbot-auto /usr/local/bin/certbot-auto
sudo chown root /usr/local/bin/certbot-auto
chmod 0755 /usr/local/bin/certbot-auto
/usr/local/bin/certbot-auto --install-only
```

## 申请通配符SSL证书 {#h2_2}
* 在Terminal中运行以下命令来生成通配符SSL证书(*.ariesme.com).

```bash
certbot-auto certonly --manual --preferred-challenges dns-01 -d *.ariesme.com --server https://acme-v02.api.letsencrypt.org/directory
```

* 过程中会要求检测域名TXT记录, 需要先修改域名DNS TXT记录(_acme-challenge.ariesme.com)成要求的信息. 添加后在另一个Terminal窗口中用以下命令检查TXT记录是否生效. 生效后在原命令窗口中回车继续生成证书.

```bash
dig -t txt _acme-challenge.ariesme.com
```

* 当出现如下信息时就说明生成成功了

```bash
- Congratulations! Your certificate and chain have been saved at:
   /etc/letsencrypt/live/ariesme.com/fullchain.pem
   Your key file has been saved at:
   /etc/letsencrypt/live/ariesme.com/privkey.pem
```

* LNMP新建VHOST时就可以设置上述两个文件路径来使用我们自己手动生成的通配符证书.

##  手动更新证书 {#h2_3}
在Terminal中运行以下命令来更新已有证书(*.ariesme.com). 过程中会要求更新域名TXT记录, 与原申请证书时的步骤一致.

```bash
certbot-auto certonly --renew-by-default --manual --preferred-challenges dns-01 -d *.ariesme.com --server https://acme-v02.api.letsencrypt.org/directory
```

##  优缺点 {#h2_4}
1. 这样手动生成的证书可以给所有二级域名使用, 做到一次生成多次使用.
2. 由于是用_acme-challenge的DNS设置验证的域名所有权, 而且证书的有效期只有90天, 所以每次需要修改DNS来手动更新证书.

## 更新历史 {#h2_5}

| Version | Detail | Date |
| ---- | ---- | ---- |
| 1.0 | 初版 | 2019-06-08 |
| 1.1 | 手动更新证书 | 2019-10-08 |
