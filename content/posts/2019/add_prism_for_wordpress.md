---
title: "WordPress添加Prism代码高亮"
description: ""
date: "2019-01-15"
tags:
  - "wordpress"
  - "prism"
author:
  name: "Vincent Liu"
---

在WordPress中添加和配置Prism代码高亮插件.
<!--more-->

## 下载js和css文件 {#h2_1}
在[PrismJS官网](https://prismjs.com/)DIY自己需要的主题, 语言以及其他插件如行号.

## 上传js和css文件至WP使用主题目录下 {#h2_2}
路径:./wp-content/themes/your_themes_name.

## 添加js和css配置 {#h2_3}
functions.php 文件中添加如下代码.

```php
function add_prism() {
    wp_register_style(
        'prismCSS',
        get_stylesheet_directory_uri() . '/prism.css'
    );
    wp_register_script(
        'prismJS',
        get_stylesheet_directory_uri() . '/prism.js'
    );
    wp_enqueue_style('prismCSS');
    wp_enqueue_script('prismJS');
}
add_action('wp_enqueue_scripts', 'add_prism');
```
