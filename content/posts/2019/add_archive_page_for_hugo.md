---
title: "Hugo添加归档页面"
description: ""
date: "2019-02-19"
tags:
  - "hugo"
author:
  name: "Vincent Liu"
menu:
  sidebar:
    name: Hugo添加归档页面
    identifier: Hugo添加归档页面
    parent: 2019
    weight: -0219
---

Hugo默认并没有归档页面, 但可以单独添加一个归档页面.
<!--more-->

## 新建归档单页面模板 {#h2_1}
1. 在Hugo根目录下新建模板文件(./layouts/archive/single.html)
2. 将主题目录下的默认模板文件内容(./themes/theme_name/layouts/_default/single.html)拷贝到新建的模板文件

## 替换页面展现部分代码 {#h2_2}
将新建的模板文件中的页面展现部分替换成如下代码.

1. 归档目录
  * .Site.Pages "Section" "post" 归档目录设置为content/post
2. 可选归档时间
  * .Pages.GroupByDate "2006" 按年归档
  * .Pages.GroupByDate "2006-01" 按年月归档
3. 折叠展开功能
  * 通过details标签实现

```html
<h2>
  {{ range where .Site.Pages "Section" "post" }}
    {{ range (.Pages.GroupByDate "2006") }}
    <details class="menu" open>
      <summary>{{ .Key }}</summary>
      {{ range .Pages }}
      <article><h3><a href="{{ .Permalink }}" title="{{ .Title }}">{{ .Title }}</a></h3></article>
      {{ end }}
    </details>
    {{ end }}
  {{ end }}
</h2>
```

## 新建归档页面 {#h2_3}
1. 新建一个归档页面./content/archive.md
2. 在文件头添加type字段来引用归档页面模板
3. 在文件头添加menu字段将归档页面链接添加到主目录

```yaml
---
title: "Archive"
type: archive
menu:
  main:
    name: Archive
    weight: 1
---
```
