---
title: "Hugo添加Google Custom Search站内搜索"
description: ""
date: "2019-03-02"
tags:
  - "hugo"
  - "google"
author:
  name: "Vincent Liu"
---

Hugo没有自带站内搜索, 可以使用Google Custom Search Engine来代替.
<!--more-->

## 注册GCSE {#h2_1}
[Google Custom Search官网](https://cse.google.com/cse/)注册帐号, 添加自己网站的链接并记录下cx参数和搜索框代码.

* cx参数

```html
https://cse.google.com/cse?cx=xxx:xxx
```

* 搜索框代码

```html
<script>
  (function() {
    var cx = 'xxx:xxx';
    var gcse = document.createElement('script');
    gcse.type = 'text/javascript';
    gcse.async = true;
    gcse.src = 'https://cse.google.com/cse.js?cx=' + cx;
    var s = document.getElementsByTagName('script')[0];
    s.parentNode.insertBefore(gcse, s);
  })();
</script>
<gcse:search></gcse:search>
```

## 配置文件中添加GCSE参数 {#h2_2}
在config.toml配置文件中添加cx参数

```toml
[Params]
  googleCustomSearch = "xxx:xxx" # Google Custom Search Engine ID
```

打开搜索框插件, 需要主题支持

```toml
[Params.sidebar]
  widgets = ["search", "recent", "categories", "taglist", "social"]
```

## 新建搜索框插件 {#h2_3}
如果主题中没有搜索框插件文件或者不喜欢主题搜索框的样式, 可以直接在Google上设置搜索样式并将相应的搜索框代码嵌入到页面div字段中. 由于mainroad主题中有搜索插件, 所以我选择使用主题提供的搜索框样式, 然后在插件文件中加入GCSE参数就行了.

1. 新建搜索框文件
Hugo根目录下新建搜索框插件文件(./layouts/partials/widgets/search.html), 并将主题目录下相同路径的search.html文件内容拷贝到新文件中.

2. 新增GCSE参数

* action="https://cse.google.com/cse" 表示使用GCSE搜索
* .Site.Params.googleCustomSearch 将config.toml文件中的googleCustomSearch赋值给cx

```html
<div class="widget-search widget">
	<form class="widget-search__form" role="search" method="get" action="https://cse.google.com/cse">
		<label>
			<input class="widget-search__field" type="search" placeholder="{{ T "search_placeholder" }}" value="" name="q" aria-label="{{ T "search_placeholder" }}">
		</label>
		<input class="widget-search__submit" type="submit" value="Search">
		<input type="hidden" name="cx" value="{{ .Site.Params.googleCustomSearch }}" />
	</form>
</div>
```
