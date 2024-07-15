---
title: "使用额外的JavaScript脚本优化知乎推荐流的浏览体验"
date: 2024-06-26T18:59:24+08:00
tags: ["2024-06", "JavaScript", "知乎", "油猴"]
categories: [""]
toc: true
draft: true
---

最近刷知乎的时候实在受不了首页给我推荐的低质量内容了，故学习了一番JavaScript，写了一个脚本（[知乎推荐流优化](https://greasyfork.org/zh-CN/scripts/498139-%E7%9F%A5%E4%B9%8E%E6%8E%A8%E8%8D%90%E6%B5%81%E4%BC%98%E5%8C%96)）来改善知乎网页端的浏览体验。现将学习过程记录一二，希望以后能够用上……

~~其实这篇文章已经在编辑器里躺了很久了，一直没有来写完~~

## 确定目标  

首先注意到知乎上质量低劣的键政内容主要是由默认头像用户产生的，因此只要把默认头像（以及满足一定额外条件的）用户的回答卡片隐藏起来就可以了。由于曾用过[知乎美化](https://greasyfork.org/zh-CN/scripts/412212-%E7%9F%A5%E4%B9%8E%E7%BE%8E%E5%8C%96)这样的油猴脚本，了解到油猴脚本应该可以实现所对应的功能，决定开始学习JavaScript。  

## 油猴脚本初探  

参考[维基百科](https://zh.wikipedia.org/wiki/Greasemonkey)上的描述，GreaseMonkey是Firefox的一个附加组件，允许用户在网页上插入自定义的JavaScript脚本以增强网页的功能。但现在已经有了更多更先进的用户脚本管理器，我选择使用的是开源的ViolentMonkey，因为另一个流行的用户脚本管理器Tampermonkey的某个版本在使用前面提到的知乎美化脚本时在某些情况下有严重的性能问题，会拖慢整个浏览器最后不得不关掉重启；另外暴力猴还支持跟踪脚本文件的修改情况，从而使用外部编辑器（我喜欢VSCode）编写脚本。  

打开暴力猴的脚本编辑器，它会自动创建一个模板：  

```javascript  
// ==UserScript==
// @name        New script 
// @namespace   Violentmonkey Scripts
// @match       *://example.org/*
// @grant       none
// @version     1.0
// @author      -
// @description 2024/7/1 19:44:25
// ==/UserScript==
```

脚本最开始的几行是元数据，描述了脚本的各项信息。在模板之外还有一部分如`icon`、`license`、`run-at`、`downloadURL`之类的项目，脚本管理器会通过这些信息来提供更友好的显示；元数据的详情可以参考[TamperMonkey的介绍](https://www.tampermonkey.net/documentation.php?locale=en)。  

绝大部分油猴脚本的主要结构都是这样的：  

```javascript
(function() {
    'use strict';
    // 一系列代码
})();
```

在JavaScript中，有着类似上面这样结构的函数被称之为**立即调用函数表达式**[^1]。这是一种设计模式，旨在创建函数后立刻执行；在脚本管理器中的体现是它会在读取到最后一行的分号后立刻开始执行脚本。

代码中第二行的`'use strict';`指示开启JavaScript的**严格模式**[^2]。严格模式可以使用异常处理并在某种程度上提升了速度。

## 对知乎网页的分析  

知乎主站前端主要是使用React编写的，每项推送内容都被包裹在一个
`<div class="Card TopstoryItem TopstoryItem-isRecommend" tabindex="0">`元素中,可以使用`document.getElementsByClassName()`或者`document.querySelectorAll()`方法获得一个当前页面的所有卡片`div`对象的集合。  

对于每一个卡片，其内部结构是由多层`div`嵌套来确定各个元素所在位置的。

[^1]:<https://developer.mozilla.org/zh-CN/docs/Glossary/IIFE>  
[^2]:<https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Strict_mode>  
