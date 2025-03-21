---
title: "使用额外的JavaScript脚本优化知乎推荐流的浏览体验"
date: 2024-06-26T18:59:24+08:00
tags: ["2024-06", "JavaScript", "知乎", "油猴"]
categories: [""]
toc: true
draft: false
aliases:
  - /2024/06/使用额外的JavaScript脚本优化知乎推荐流的浏览体验/
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

对于每一个卡片，其内部结构是由多层`div`嵌套来确定各个元素所在位置的，并且在属性中会包含此卡片的一些元数据信息。对前段中提到的每个卡片对象，可以在它的子树中搜索属性为`ContentItem`的元素；其中含有的`data-zop`属性以JSON格式存储了此卡片的一些信息，比如作者、问题全文、卡片ID等，另一个属性`data-za-extra-module`中包含了`author_member_hash_id`项，即答案作者的标识符（与用户名的关系类似微信的wxid之于微信号）。  

对每一个知乎用户，可以通过移动端API`https://api.zhihu.com/people/${userId}/profile?profile_new_version=1`来获得此用户的公开信息，例如头像URL、回答数量、关注者数量、答案数量等；其中`${userId}`是前面提到的用户hash id信息。  

## 开始编写  

理清知乎每个卡片的数据关系后，就可以开始编写脚本用于移除不想看到的卡片了。Javascript中提供了被称为**MutationObserver**[^3]的事件监听器，可以针对DOM树中不同的更改执行相应的回调函数。这里我们要监听的是DOM树中子节点发生变化（被添加进来）的事件，这个事件会在每次重载网页或者随着推荐流向下划、新卡片加入到整个首页的推荐流中被触发；由于MutationObserver记录到的并不都是我们想要的添加新卡片的记录，因此要从其中筛选新卡片对象并将其存入数组中等待使用。  

待加入的新卡片数量达到设定的阈值后，就可以对其进行逐一检查了。这个阈值不宜设置得太大，由于知乎推荐流存在预加载的情况（即在用户往下滑到底之前就开始请求新的推荐内容，虽然这部分内容还不会被浏览器渲染出现在窗口里），若设的太大会发生符合条件的卡片在用户眼前消失的情况；经过测试，将新加入的卡片数量阈值设置为5能提供比较好的体验。  

对每个答案卡片，首先检查的是答案作者是否为默认头像（毕竟是本项目的初衷所在）。检查头像时并没有必要将头像的图片下载下来；对于使用默认头像的用户，知乎会直接使用图床里的默认头像URL[^4]，因此只需要比较头像图片的URL中是否包含特定的hash值`abed1a8c04700ba7d72b45195223e0ff`就可以了。对其他的默认头像也可以使用此方法，只需要建立一个默认头像的hash值库并遍历比对即可。  

使用关键词屏蔽问题是知乎盐选会员的功能，但知乎那一堆盐选专享小说实在令人敬谢不敏。油猴为每个用户脚本提供了配置储存的功能，于是可以让用户输入屏蔽词组并对每个卡片所属问题标题进行匹配（直接使用`string.includes()`即可）。  

对于匹配上关键词的问题，通过点击『不感兴趣』按钮可以让知乎的推荐算法了解到你的喜好，少推这类问题。虽然通过抓包可以看到『不感兴趣』功能是通过向一个知乎的RESTful API发送带参数的get请求实现的，但实际上直接通过XHR请求这个API会报HTTP 403，即使带上Cookie做好伪装也不行；于是此功能只能通过模拟点击来实现。  

对于每个答案卡片，底部操作栏最右侧的三点菜单就是一个包含着svg图像（也就是那三个点）的`button`实现，因此可以轻易地用JavaScript获取此对象并模拟点击。点击后出现的悬浮窗是整个网页`body`下的一个小`div`，位置与触发此悬浮窗的卡片绑定。对于预加载的部分这个点击同样有效，只是坐标超出了浏览器窗口范围不会被用户看到；于是只需要监听`document.body`的变化后模拟点击此按钮，并在悬浮窗中点击『不感兴趣』就可以让知乎少推此类内容了。  

{{< figure
    src="https://s3.bmp.ovh/imgs/2024/08/29/1d64fce3f99ff66b.png"
    caption="三点式菜单展开"
    align=center
>}}  

在默认头像和屏蔽词两个方式都检测过后，对符合条件的卡片我们会将其挑选出来。脚本中以全局数组的形式存储了每个符合条件将要被移除的对象；这里是通过在卡片`div`中添加`hidden`属性让用户看不到此卡片，眼不见心不烦。如果选择使用`Element.remove()`实现移除会导致点击知乎顶部『推荐』按钮获取新推荐流时报错，推测此功能是通过将所有卡片移除后再拉取新卡片实现的，用户脚本自己移除会导致remove时找不到对象而报错。  

## 总结  

虽然经过数周实际体验和迭代，这个项目已经到了能用的水平，并且实测在使用脚本多次点击『不感兴趣』后知乎会再也不推送相关内容，但还是有一些问题尚待解决。  

首先是不能对展开评论区里的默认头像用户进行移除：虽然已经编写好了这部分代码，但实际测试时发现对评论的操作并不是十分稳定，并且由于评论区结构是树状交织的（因为有回复和对回复的回复）移除后会显得很奇怪遂暂时作罢；其次在运行时会发生脚本运行在卡片加载完成之前导致检查失败（控制台中出现类似`Uncaught (in promise) TypeError: card.querySelector(...) is null`的报错）。这个问题单纯靠增加运行延迟应该是治标不治本的，还需要寻找一个更好的解决方案。最后，如果刷太久知乎导致脚本请求用户信息API的次数太多，会导致知乎反爬虫机制触发要求人工验证。不过一般简单看个几十条是不会出现的，摸鱼还需慎重（  

如果看到这里的你有更好的方法，还请在[GitHub仓库](https://github.com/lisolaris/ZhihuFeedOptimization)中提出issue，感谢您的不吝赐教！  

[^1]:<https://developer.mozilla.org/zh-CN/docs/Glossary/IIFE>  
[^2]:<https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Strict_mode>  
[^3]:<https://developer.mozilla.org/zh-CN/docs/Web/API/MutationObserver>
[^4]:<https://pica.zhimg.com/v2-abed1a8c04700ba7d72b45195223e0ff_l.jpg>
