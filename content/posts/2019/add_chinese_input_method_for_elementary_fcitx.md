---
title: "Elementary安装fcitx中文输入法"
description: ""
date: "2019-02-16"
tags:
  - "elementary"
  - "linux"
  - "fcitx"
author:
  name: "Vincent Liu"
---

网上的方法都是一次性安装fcitx和fcitx-pinyin然后才配置im-config, 这种办法一直没有成功. 正确的顺序是安装完fcitx之后就得设置im-config, 最后再安装fcitx-pinyin才能成功.
<!--more-->

## 安装fcitx {#h2_1}
打开Terminal并运行如下命令.

```bash
sudo apt install fcitx
```

## 设置fcitx为默认输入法 {#h2_2}
Terminal中运行如下命令, 在弹出的窗口点击两次Yes后选择fcitx并保存.

```bash
sudo im-config
```

## 安装fcitx-pinyin {#h2_3}
Terminal中运行如下命令.

```bash
sudo apt install fcitx-pinyin
```

## 重启机器 {#h2_4}
重启完后, ctrl+space可切换到fictx输入法, 后续点击shift可在中英文输入法之间切换.
