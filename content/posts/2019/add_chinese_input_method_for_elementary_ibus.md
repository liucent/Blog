---
title: "Elementary安装iBus中文输入法"
description: ""
date: "2019-07-14"
tags:
  - "elementary"
  - "linux"
  - "ibus"
author:
  name: "Vincent Liu"
---

前段时间记录了如何安装基于fcitx的输入法, 最近发现其实系统自带的有ibus中文输入法, 并且只需要简单的配置下就行了.
<!--more-->

## 添加中文输入法 {#h2_1}
在Applications中打开System Settings, 依次进入Language&Region->Keyboard Settings->Input Method Settings->Input Method页面. 点击Add按钮, 选择Chinese->Intelligent Pinyin. 添加成功后, 点击Save按钮退出.
PS: 第一次进入可能会提示iBus未启动, 点击Yes启动iBus就行了.

## 设置iBus为默认输入法 {#h2_2}
Terminal中运行如下命令, 在弹出的窗口点击两次Yes后选择iBus并保存.

```bash
sudo im-config
```

## 重启机器 {#h2_3}
重启完后, 默认就是中文输入法了, 点击shift可在中英文输入法之间切换, 也可以ctrl+space切换到别的输入法.
