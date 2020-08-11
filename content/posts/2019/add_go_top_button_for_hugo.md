---
title: "Hugo添加返回顶部按钮"
description: ""
date: "2019-03-01"
tags:
  - "hugo"
author:
  name: "Vincent Liu"
menu:
  sidebar:
    name: Hugo添加返回顶部按钮
    identifier: Hugo添加返回顶部按钮
    parent: 2019
    weight: -0301
---

返回顶部按钮虽然看上去小巧，但确实是个很方便的功能. 读者阅读完一些较长文章后能快速返回页面顶部点选其他文章.
<!--more-->

## 1. 基础模板文件中添加返回顶部按钮元素 {#h2_1}
Hugo站点根目录下的基础模板文件(./layouts/_default/baseof.html)的body字段尾部添加如下代码. 如果没有该路径或者该html文件, 可以将主题目录themes下相同路径的baseof.html文件拷贝至此.

1. class="btn" 表示使用主题style.css中的.btn样式, 也可以用自己的样式
2. id="goTopBtn" 按钮的id, 便于在css文件中编写特定的样式, 或者在js文件编写方法
3. onclick="smoothScrollTop()" 点击按钮运行js文件中的smoothScrollTop方法
4. title="Go to top" 表示光标移动到按钮上时显示的tips

```html
<body>
  <button class="btn" onclick="smoothScrollTop()" id="goTopBtn" title="Go to top">TOP</button>
</body>
```

## 2. 按钮自定义样式 {#h2_2}
Hugo站点根目录下的自定义样式文件(./static/css/custom.css)中添加如下代码. 添加css和js文件的方法可以参看[Hugo添加Prism代码高亮](https://www.cloudme.top/post/2019/add_prism_for_hugo/)这篇文章中添加prism.css和prism.js文件的方法.

1. \#name 表示该样式应用于id="name"的元素
2. .name 表示该样式应用于class="name"的元素
3. position: fixed; 表示按钮在页面上的固定位置, 结合bottom和right参数可以将按钮固定在页面左下角
4. z-index: 99; 表示z轴参数, 将按钮置于页面最顶层, 防止被其他元素遮盖
5. cursor: pointer; 表示光标移动到按钮上部时光标指针变成小手

```css
#goTopBtn {
  display: none; /* Hidden by default */
  position: fixed; /* Fixed/sticky position */
  bottom: 25px; /* Place the button at the bottom of the page */
  right: 25px; /* Place the button 30px from the right */
  z-index: 99; /* Make sure it does not overlap */
  border: none; /* Remove borders */
  outline: none; /* Remove outline */
  cursor: pointer; /* Add a mouse pointer on hover */
  padding: 10px; /* Some padding */
  border-radius: 0.3em; /* Rounded corners */
  font-size: 12px; /* Increase font size */
}
```

下面是使用class="btn"后继承于主题中的btn样式部分代码. btn:hover表示光标移动到按钮上部时按钮变色.

```css
/* Button */
.btn {
	padding: 5px 10px;
	font-weight: 700;
	color: #fff;
	white-space: pre-line;
	background: #2a2a2a;
}

.btn:hover {
	color: #fff;
	background: #e64946;
}
```

## 3. 返回顶部行为方法 {#h2_3}

返回顶部的方法可以用javascript实现, 也可以用jquery实现. 下面就分开介绍下.

### 3.1. javascript方法 {#h3_3_1}
Hugo站点根目录下的自定义JavaScript文件(./static/js/custom.js)中添加如下代码. 提供了两种返回方式, 可根据喜好选择其中一种.

1. scrollTop 方法是直接回顶部
2. smoothScrollTop 方法是平滑回滚到顶部
3. scrollTopButton 方法只有向下滚动超过20px后才会显示按钮, 其他情况按钮隐藏


```javascript
// When the user scrolls down 20px from the top of the document, show the button
window.onscroll = function() {scrollTopButton()};

function scrollTopButton() {
  if (document.body.scrollTop > 20 || document.documentElement.scrollTop > 20) {
    document.getElementById("goTopBtn").style.display = "block";
  } else {
    document.getElementById("goTopBtn").style.display = "none";
  }
}

// When the user clicks on the button, scroll to the top of the document
function scrollTop() {
  document.body.scrollTop = 0; // For Safari
  document.documentElement.scrollTop = 0; // For Chrome, Firefox, IE and Opera
}

//Smooth scroll to the top of the document
function smoothScrollTop(){
    var timer  = null;
    cancelAnimationFrame(timer);
    timer = requestAnimationFrame(function fn(){
        var oTop = document.body.scrollTop || document.documentElement.scrollTop;
        if(oTop > 0){
            document.body.scrollTop = document.documentElement.scrollTop = oTop - 50;
            timer = requestAnimationFrame(fn);
        }else{
            cancelAnimationFrame(timer);
        }
    });
}
```

### 3.2. jquery方法 {#h3_3_2}
上面的样式是根据以前主题写的, 现在更新jquery的方法就顺便记录下当前主题配套的返回按钮样式, css代码如下, 其中返回按钮元素id="gotoTop".

```css
/*gotoTop button*/
#gotoTop {
    position:fixed;
    right:0;
    opacity:.8;
    visibility:hidden;
    bottom:25px;
    margin:0 25px 0 0;
    z-index:99;
    transition:.35s;
    transform:scale(0);
    transition:all .5s
}
#gotoTop.visible {
    opacity:.8;
    visibility:visible;
    transform:scale(1)
}
#gotoTop.visible a:hover {
    outline:none;
    opacity:.9;
    background:var(--accent)
}
#gotoTop a {
    outline:none;
    text-decoration:none;
    border:0;
    display:block;
    width:46px;
    height:46px;
    background-color:#212121;
    opacity:.8;
    transition:all .3s;
    border-radius:50%;
    text-align:center;
    font-size:26px
}
body #gotoTop a {
    outline:none;
    color:#fff
}
#gotoTop a:after {
    outline:none;
    content:"\f106";
    font-family:fontawesome;
    position:relative;
    display:block;
    top:50%;
    -webkit-transform:translateY(-55%);
    transform:translateY(-55%);
    color:#ffffff
}
```

同以前一样, 将返回按钮添加在\<footer\>元素之前.

```html
<!--  goto Top button  -->
<div id="gotoTop"><a href="#"></a></div>
```

jquery代码部分主要是用animate方法. 按钮的显示与隐藏用visible属性实现.

```javascript
/* Back to Top button behaviour */
    var pxShow = 100;
    var scrollSpeed = 500;
    $(window).scroll(function() {
        if ($(window).scrollTop() >= pxShow) {
            $("#gotoTop").addClass('visible');
        } else {
            $("#gotoTop").removeClass('visible');
        }
    });

    $('#gotoTop a').on('click', function() {
        $('html, body').animate({
            scrollTop: 0
        }, scrollSpeed);
        ;
        return false;
    });
```

## 4. 更新历史 {#h2_4}

| Version | Detail | Date |
| ---- | ---- | ---- |
| 1.0 | 初版 | 2019-03-01 |
| 1.1 | 添加jquery方法 | 2019-11-02 |
