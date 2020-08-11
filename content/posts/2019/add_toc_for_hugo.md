---
title: "Hugo添加文章目录toc"
description: ""
date: "2019-10-30"
tags:
  - "hugo"
  - "toc"
author:
  name: "Vincent Liu"
menu:
  sidebar:
    name: Hugo添加文章目录toc
    identifier: Hugo添加文章目录toc
    parent: 2019
    weight: -1030
---

这几天在原来hugo主题[minimal](https://github.com/calintat/minimal)的基础上尝试加了两个功能, 一个是站内搜索, 一个是文章目录(Table of Contents). 不过都是跟着其他主题修改过来的. 这次先写下文章目录toc的移植修改过程.
<!--more-->

## 1. 添加toc显示变量 {#h2_1}
如果你想始终显示toc的话, 其实可以不用设置这些变量. 但是在小屏幕或者移动设备上, 显示toc的体验并不好, 一是占用屏幕空间, 二是对于可滑动的屏幕也无必要. 所以还是建议大家在config.toml中加上toc的显示控制变量. 后面也可以js上控制是否显示.

```toml
[params]
    toc = true
```

## 2. toc模板和样式 {#h2_2}

Hugo自带的有toc模板, 可以通过{{.Content}}引入这个模板, 具体可以看官方使用[教程](https://gohugo.io/content-management/toc/#template-example-basic-toc).

### 2.1. 新建toc模板 {#h3_2_1}
个人觉得这个默认模板的样式不好设置, 另外的问题就是默认模板从\<h1\>就开始生成目录了, 这个对于喜欢把\<h1\>作为文章标题, 而文章内容从\<h2\>开始的同学包括我来说不是很灵活. 因此决定自己写模板. 于是就来借用了[AllInOne](https://github.com/orianna-zzo/AllinOne)主题的toc模板, 加了点小改动. 在layouts/partials目录下新建toc.html. 这个模板就能生成ul\>li\>a层级的目录结构了, 其中\<ul\>的层级数和\<h\>header数一致.

```html
<!-- toc.html -->
<!-- ignore empty links with + -->
{{ $headers := findRE "<h[1-4].*?>(.|\n])+?</h[1-4]>" .Content }}
<!-- at least one header to link to -->
{{ if ge (len $headers) 1 }}
{{ $h1_n := len (findRE "(.|\n])+?" .Content) }}
{{ $re := (cond (eq $h1_n 0) "<h[2-4]" "<h[1-4]") }}
{{ $renum := (cond (eq $h1_n 0) "[2-4]" "[1-4]") }}

<!--Scrollspy-->
<div class="toc">

    <div class="page-header"><strong>- CATALOG -</strong></div>

    <div id="page-scrollspy" class="toc-nav">

        {{ range $headers }}
        {{ $header := . }}
            {{ range first 1 (findRE $re $header 1) }}
                {{ range findRE $renum . 1 }}
                {{ $next_heading := (cond (eq $h1_n 0) (sub (int .) 1 ) (int . ) ) }}
                    {{ range seq $next_heading }}
                    <ul class="nav">
                    {{end}}
                    {{ $anchorId := (replaceRE ".* id=\"(.*?)\".*" "$1" $header ) }}
                        <li class="nav-item">
                            <a class="nav-link text-left" href="#{{ $anchorId }}">
                            {{ $header | plainify | htmlUnescape }}
                            </a>
                        </li>
                    <!-- close list -->
                    {{ range seq $next_heading }}
                    </ul>
                    {{ end }}
                {{ end }}
            {{ end }}
        {{ end }}

    </div>

</div>
<!--Scrollspy-->

{{ end }}
```

### 2.2. 新建toc样式 {#h3_2_2}
* 主要是设置toc, toc-nav和nav-link的样式, 其中的颜色主要是为了和minimal主题适配, accent的颜色变量就是minimal主题可设置的主颜色. 在自己的css样式文件(例如static/css/custom.css)中添加如下代码.

```css
/* toc style */
.toc {
    position: fixed;
    top: 50%;
    right: 10%;
    width: 10%;
    transform: translateY(-50%);
    /*background-color: #f6f6f6;*/
    /*border: solid 1px #c9c9c9;*/
    border-radius: 5px;
    padding-bottom: 1rem;
}

.toc .page-header {
    margin-top: 1rem;
    margin-bottom: 1rem;
}

.toc-nav ul {
    overflow:hidden;
    white-space:nowrap;
    line-height: 1rem;
}

/* ignore h1 header */
.toc-nav ul ul ul {
    margin-left: 2rem;
}

.toc-nav .nav-link {
    text-overflow:ellipsis;
    overflow:hidden;
    color: #333;
}
```

* 除了设置静态的样式外, 也要设置下目录条目在激活状态下的样式. 当页面滚动到相应目录时, 目录条目就会处于激活状态. 这个功能是由scrollspy实现的, 这个后面再介绍. 现在只要先设置好目录条目在class="active"时的样式就行, 具体的样式就是文字变色, 条目背景变色并且在条目左边生成一条小竖线.

```css
.toc-nav li.active .nav-link {
    background-color: #f6f6f6;
    color: var(--accent);
    border-left: solid 2px var(--accent);
}
```

* 手机, 平板等小屏幕设备上显示toc目录会很拥挤, 体验并不好. 所以在小屏幕设备上将目录隐藏. 如下代码中当屏幕宽度小于1080px时, 将隐藏class="toc"的元素, 并将main元素宽度扩大到100%, 让文章内容宽度适配屏幕宽度.
```css
/* Media Queries */
@media (max-width: 1080px) {
    main {
        max-width: 100%;
    }
    .toc {
        display: none;
    }
}
```

### 2.3. 引入toc模板 {#h3_2_3}
添加好toc模板和样式代码后, 下面就要将toc模板引入到文章模板中. 在文章模板文件(layouts/post/single.html)中\<footer\>元素之前添加toc模板. 当toc=true时才会添加模板, 默认false不添加.

```html
{{ if .Site.Params.toc | default false }}
{{ partial "toc" . }}
{{ end }}
```

## 3. 添加目录跳转 {#h2_3}
经过前面的几个步骤, 文章目录已经可以在文章页面的右侧固定显示了. 如果文章章节标题\<h\>元素的内容为英文的话, 点击目录也会跳转到对应章节. 但是这里还是会有两个问题, 第一个问题是Hugo自动转换\<h\>元素时, 会把内容转换成id属性值, 也就是说如果章节标题是中文的话, id值也是中文. 然而在toc模板中通过Hugo的{{.Content}}内容截取\<h\>章节标题时会将中文字符编码成%xx. toc条目的href属性值也就是%xx类似的编码了, 这样js无法识别, 点击目录条目就会报错. 第二个问题是默认的点击条目跳转章节的动画是直接生硬的跳到对应章节, 没有过渡动画, 视觉效果不是很理想.下面就来一一解决这两个问题.

### 3.1. 中文字符编码 {#h3_3_1}
这里用到decodeURI方法将toc条目元素\<a\>的href属性值%xx形式解码成中文. hash的结果就是该元素的href值, 例如"#章节标题". 但是前面的#号是不需要解码的, 用replace替换成空字符就行了. 

```javascript
/* decode chinese hash */
var target = decodeURI(this.hash.replace(/^#/, ''));
```

### 3.2. 平滑跳转 {#h3_3_2}
上面我们已经可以得到正确的中文字符的hash位置值了, 然后通过该值和章节标题元素的id属性值, 找到对应的章节标题元素. 进而使用offset方法计算出该标题元素与页面顶端的距离. 最后通过animate方法平滑滚动到对应位置. 在自己的js文件中添加如下完整的js代码. 其中navbarHeight变量是页面顶部导航栏的高度, 由于我是固定导航栏, 所以要扣除这个高度的影响. 如果你是自动隐藏或者不固定的导航栏, 可以不用扣除这个值. scrollSpeed是滚动速度, 可以自行调整成自己喜欢的速度.

```javascript
/* scroll to the anchor and scroll spy */
var navbarHeight = 55;
$("#page-scrollspy a.nav-link").on('click', function () {
    /* decode chinese hash */
    var target = decodeURI(this.hash.replace(/^#/, ''));
    $('html,body').animate({scrollTop: $(":header[id='" + target + "']").offset().top - navbarHeight}, scrollSpeed);
    return false;
});
```

## 4. 添加滑动监听 {#h2_4}
最后我们来解决页面的滚动监听功能, 这里会用到bootstrap的scrollspy插件, 这里有比较不错的[scrollspy教程](https://www.runoob.com/bootstrap/bootstrap-scrollspy-plugin.html). 这样页面滚动到章节内容时, toc目录中对应的条目就会变成active状态, 结合前面设置的li.active的样式就能清楚的看出当前章节的位置了. 添加滑动监听有两种方法, 一是在所要监听的元素上设置data-spy和data-target属性, 通过data-target属性值将toc目录组件与被监听元素绑定起来. 二是在js文件中绑定这些元素属性. 个人觉得在html中绑定, 不是很灵活, 而且不好找到元素. 所以选择在js中绑定, 代码如下. 其中offset依旧是顶部导航栏的高度, 这里多加的5px, 是为了调整点击跳转和页面平滑滚两种情况下目录条目active状态的些许误差. 这个值根据实际页面情况调整, 并非固定值.

```javascript
$('body').scrollspy({ target: "#page-scrollspy", offset: navbarHeight+5 });
```

## 5. 遗留问题 {#h2_5}
上面我们虽然通过decodeURI方法解决了目录跳转中的中文编码问题. 但是scrollspy对中文的支持并不好, 如果目录条目href或者文章章节标题id中含有中文字符, 滚动监听将失效, 并会报js错误. 目前没发现好的办法解决这个问题. 所以只能先手动在markdown文件中添加各个标题的id属性, 注意属性值在文章中的唯一性.

```markdown
## title {#id_value}
```

## 6. 更新历史 {#h2_6}

| Version | Detail | Date |
| ---- | ---- | ---- |
| 1.0 | 初版 | 2019-10-30 |
| 1.1 | 小屏幕隐藏目录 | 2019-11-09 |