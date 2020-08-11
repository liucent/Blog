---
title: "Elementary安装中文输入法"
description: ""
date: "2019-07-14"
tags:
  - "elementary"
  - "linux"
  - "ibus"
  - "fcitx"
author:
  name: "Vincent Liu"
menu:
  sidebar:
    name: Elementary安装中文输入法
    identifier: Elementary安装中文输入法
    parent: 2019
    weight: -0714
---

前段时间安装基于fcitx的输入法, 最近发现其实系统自带的有ibus中文输入法, 现在统一记录下配置两种输入法的方法.
<!--more-->

## 1. 配置ibus输入法 {#h2_1}

### 1.1. 添加中文输入法 {#h3_1_1}
在Applications中打开System Settings, 依次进入Language&Region->Keyboard Settings->Input Method Settings->Input Method页面. 点击Add按钮, 选择Chinese->Intelligent Pinyin. 添加成功后, 点击Save按钮退出.
PS: 第一次进入可能会提示iBus未启动, 点击Yes启动iBus就行了.

### 1.2. 设置iBus为默认输入法 {#h3_1_2}
Terminal中运行如下命令, 在弹出的窗口点击两次Yes后选择iBus并保存.

```bash
sudo im-config
```

### 1.3. 重启机器 {#h3_1_3}
重启完后, 默认就是中文输入法了, 点击shift可在中英文输入法之间切换, 也可以ctrl+space切换到别的输入法.

## 2. 配置fcitx输入法 {#h2_2}

### 2.1. 安装fcitx {#h3_2_1}
打开Terminal并运行如下命令.

```bash
sudo apt install fcitx
```

### 2.2. 设置fcitx为默认输入法 {#h3_2_2}
Terminal中运行如下命令, 在弹出的窗口点击两次Yes后选择fcitx并保存.

```bash
sudo im-config
```

### 2.3. 安装fcitx-pinyin {#h3_2_3}
Terminal中运行如下命令.

```bash
sudo apt install fcitx-pinyin
```

### 2.4. 重启机器 {#h3_2_4}
重启完后, ctrl+space可切换到fictx输入法, 后续点击shift可在中英文输入法之间切换.
