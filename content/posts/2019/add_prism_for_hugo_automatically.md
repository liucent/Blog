---
title: "Hugo动态加载prism.js"
description: ""
date: "2019-10-20"
tags:
  - "hugo"
  - "prism"
author:
  name: "Vincent Liu"
menu:
  sidebar:
    name: Hugo动态加载prism.js
    identifier: Hugo动态加载prism.js
    parent: 2019
    weight: -1020
---

以前[文章](https://www.ariesme.com/post/2019/add_prism_for_hugo)记录过怎样在Hugo中添加prism代码高亮的方法. 但是每次添加新的语言或者想换个主题就得重新去官网下载新的js和css文件, 比较繁琐. 所以今天介绍下利用[cdnjs](https://cdnjs.com/)来通过变量动态加载所需语言和主题的方法.
<!--more-->

## 添加语言和主题等变量 {#h2_1}
在config.toml中添加四个变量, 一是是否开启prism, 二是选择的样式主题, 三是选择语言集合, 四是是否开启行号.

```toml
[params]
    prism = true
    prismStyle = "coy" # theme: okaidia, coy...
    prismLanguages = ["bash", "java", "toml", "yaml", "nginx"]
    prismLineNumbers = true
```

## 动态添加js文件 {#h2_2}
在添加js设置的文件中, body元素的结尾处, 添加如下代码. js中包含了基础包, 语言集合以及行号插件. 另外如果prism和prismLineNumbers变量未设置则默认不加载prism和行号.

```html
<!-- prism.js default by false-->
{{ if .Site.Params.prism | default false }}
    <!--  base js  -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/prism/1.17.1/prism.min.js"></script>
    <!--  languages  -->
    {{ range .Site.Params.prismLanguages }}
    <script src="https://cdnjs.cloudflare.com/ajax/libs/prism/1.17.1/components/prism-{{ . }}.min.js"></script>
    {{ end }}
    <!--  line-numbers default by false  -->
    {{ if .Site.Params.prismLineNumbers | default false }}
    <script src="https://cdnjs.cloudflare.com/ajax/libs/prism/1.17.1/plugins/line-numbers/prism-line-numbers.min.js"></script>
    {{ end }}

{{ end }}
```

## 动态添加css文件 {#h2_3}
在添加css设置的文件中, head元素结尾处, 添加如下代码. css中包含了样式主题和行号插件. 另外如果prismStyle和prismLineNumbers变量未设置则默认主题为okaidia和不加载行号.

```html
<!-- theme style -->
{{ if .Site.Params.prism | default false }}
<link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/prism/1.17.1/themes/prism-{{ .Site.Params.prismStyle | default "okaidia" }}.min.css">
{{ end }}
<!-- line numbers -->
{{ if .Site.Params.prismLineNumbers | default false }}
<link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/prism/1.17.1/plugins/line-numbers/prism-line-numbers.min.css">
{{ end }}
```

## 行号简化设置 {#h2_4}
经过以上步骤其实就可以使用prism的功能了. 但是行号设置还是比较复杂, 一是\`\`\`code\`\`\`代码块不支持行号设置, 二是\<pre\>code\</pre\>代码块中要显示行号每次都要添加class="line-numbers". 查看prism官方介绍得知可以在body元素中添加class="line-numbers"就可以让整个页面中的prism代码块都显示行号. 这样我们只要在Hugo的页面模板(single或者baseof等)中稍加改动就可以轻松显示行号并且可以直接让\`\`\`code\`\`\`代码块显示行号, 而不用再在每块代码处添加class行号属性了.

```html
<body {{ if .Site.Params.prismLineNumbers }}class="line-numbers"{{ end }}>
```
