---
title: "Hugo添加Google Analytics"
description: ""
date: "2019-02-24"
tags:
  - "hugo"
  - "google"
author:
  name: "Vincent Liu"
---

Hugo Blog添加Google Analytics
<!--more-->

## 注册Google Analytics {#h2_1}
1. 打开[Google Analytics官网](https://analytics.google.com/analytics/web/)注册账户并添加自己的网站域名
2. 打开Google Analytics主页->Admin->Property->Tracking Info->Tracking Code并获取对应域名的Tracking Code

## 设置googleAnalytics参数 {#h2_2}
在config.toml中新建googleAnalytics参数并设置成自己的Tracking Code

```toml
googleAnalytics = "xx-xxxxxxxxx-x" # Enable Google Analytics by entering your tracking id
```

## 新建Google Analytics模板 {#h2_3}
在Hugo站点根目录下新建模板文件(./layouts/_internal/google_analytics_async.html)并添加如下代码.

```html
<!-- Global Site Tag (gtag.js) - Google Analytics -->
<script async src="https://www.googletagmanager.com/gtag/js?id={{ .Site.GoogleAnalytics }}"></script>
<script>
  window.dataLayer = window.dataLayer || [];
  function gtag(){dataLayer.push(arguments);}
  gtag('js', new Date());

  gtag('config', '{{ .Site.GoogleAnalytics }}');
</script>
```

## 引用Google Analytics模板 {#h2_4}
在baseof.html基础模板文件中的head标签尾部添加如下代码, 这样站点发布到非Hugo Server后就会自动引用Google Analytics模板.
```html
<head>
{{- if not .Site.IsServer }}
	{{ template "_internal/google_analytics_async.html" . }}
{{- end }}
</head>
```
