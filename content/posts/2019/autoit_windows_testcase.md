---
title: "AutoIt的窗口自动化测试"
description: ""
date: "2019-01-16"
tags:
  - "autoit"
  - "selenium"
author:
  name: "Vincent Liu"
menu:
  sidebar:
    name: AutoIt的窗口自动化测试
    identifier: AutoIt的窗口自动化测试
    parent: 2019
    weight: -0116
---

Selenium自动化测试时会遇到需要调用Windows弹出窗口上传或下载文件的场景. 可以使用AutoIt工具实现.
<!--more-->

## 安装AutoIt {#h2_1}
[AutoIt官网](https://www.autoitscript.com/site/)下载安装即可. 实际上只有编写和调试代码时会用到AutoIt软件工具, 代码写好后运行期间是不需要的. 也就是说代码可以在未安装AutoIt软件的测试机上运行.

## 添加Maven依赖库 {#h2_2}
在Selenium工程的Maven配置文件pom.xml添加. 如需最新的版本可以去[Maven依赖库官网](https://mvnrepository.com/)查找.

```xml
<dependency>
  <groupId>de.openkeyword</groupId>
  <artifactId>autoit</artifactId>
  <version>0.1.13</version>
</dependency>
```

## 定位和操作窗口元素 {#h2_3}
以文件上传窗口为例.

1. 在AutoIt的安装目录下运行Au3Info.exe(64位系统请使用Au3Info_x64.exe)工具, 点击并拖动Finder Tool按钮至弹出框输入文件路径和打开按钮, 并记录下对应的Title和ClassnameNN字段
2. 在Selenium的测试用例java文件中添加如下代码

```java
AutoItX autoItX = new AutoItX();
//Wait for windows opening
autoItX.winWait("Open");
autoItX.sleep(1000);

//Focus on Upload file path field
autoItX.controlFocus(
    "Open",    /*Title(Upload file path field)*/
    "",
    "Edit1"    /*ClassnameNN(Upload file path field)*/
    );
autoItX.sleep(1000);

//Input upload file path
autoItX.controlSend(
    "Open",    /*Title(Upload file path field)*/
    "",
    "Edit1",    /*ClassnameNN(Upload file path field)*/
    imagePath    /*Upload file path*/
    );
autoItX.sleep(1000);

//Click Open button to submit
autoItX.controlClick(
    "Open",    /*Title(Open button)*/
    "",
    "Button1"    /*ClassnameNN(Open button)*/
    );
autoItX.sleep(1000);
```
