---
title: "Hugo添加Prism代码高亮"
description: ""
date: "2019-02-22"
tags:
  - "hugo"
  - "prism"
author:
  name: "Vincent Liu"
---

Hugo自带的Chroma代码高亮不是很方便, 还是请出Prism.
<!--more-->

## 下载js和css文件 {#h2_1}
在[PrismJS官网](https://prismjs.com/)DIY自己需要的主题, 语言以及其他插件如行号.

## 将js和css文件放到对应目录下 {#h2_2}
1. ./static/css/prism.css
2. ./static/js/prism.js

## 在页面模板文件中添加js和css引用 {#h2_3}
1. 新建默认基础模板文件(./layouts/_default/baseof.html)
2. 将主题中的基础模板文件内容(./themes/theme_name/layouts/_default/baseof.html)拷贝到新文件中
3. 在head字段尾部加入自定义css, 在body字段尾部加入自定义js

```html
<head>
  {{ range .Site.Params.customCSS -}}
  <link rel="stylesheet" href="{{ . | relURL }}">
  {- end }}
</head>
<body>
  {{ range .Site.Params.customJS -}}
  <script src="{{ . | relURL }}"></script>
  {{- end }}
</body>
```

## 配置文件中添加自定义js和css参数 {#h2_4}
在config.toml中添加如下参数, 并设置好js和css文件路径.

```toml
[Params]
  customCSS = ["css/prism.css"] # Include custom CSS files
  customJS = ["js/prism.js"] # Include custom JS files
```

## 文章中代码高亮格式 {#h2_5}
Markdown文件中既可以使用```块标记来调用Prism高亮, 同时也可以使用Prism自己的格式.

1. line-numbers 表示添加行号
2. language-html 设置语言种类

```xml
<pre class="line-numbers"><code class="language-html">your_code</code></pre>
```
