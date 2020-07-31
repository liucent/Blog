---
title: "WordPress关闭自动更新和版本控制"
description: ""
date: "2018-12-17"
tags:
  - "wordpress"
author:
  name: "Vincent Liu"
---

WordPress安装完成后,需要将不必要的自动更新和自动保存功能关闭.
<!--more-->

## 关闭自动更新 {#h2_1}
wp-config.php 文件中添加如下代码.

```php
//Disable automatic update
define('AUTOMATIC_UPDATER_DISABLED', true);
```

## 关闭自动更新提示 {#h2_2}
functions.php 文件中添加如下代码.

```php
//Disable automatic update notification
add_filter('pre_site_transient_update_core',create_function('$a',"return null;"));
add_filter('pre_site_transient_update_plugins',create_function('$a',"return null;"));
add_filter('pre_site_transient_update_themes',create_function('$a',"return null;"));
remove_action('admin_init','_maybe_update_core');
remove_action('admin_init','_maybe_update_plugins');
remove_action('admin_init','_maybe_update_themes');
```

## 关闭文章自动保存 {#h2_3}
wp-config.php 文件中添加如下代码.

```php
//Disable revisions and autosave per day
define('WP_POST_REVISIONS', false);
define('AUTOSAVE_INTERVAL', 36000);
```
