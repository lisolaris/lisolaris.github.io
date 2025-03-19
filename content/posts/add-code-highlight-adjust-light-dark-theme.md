---
title: "为PaperMod主题增加代码语法高亮适应亮/暗模式"
date: 2025-03-19T18:33:26+08:00
tags: ["2025-03", "Hugo", "PaperMod", "JavaScript"]
categories: [""]
toc: false
draft: true
---




既然都改了模板，就打算一改到底了。PaperMod原生的代码块只有黑色背景，在白色主题下看起来相当丑；查了一番后发现Hugo使用的是一个叫做`highlight.js`[^3]的项目，并且可以直接在网页头部导入这个JavaScript：

```javascript
<link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/highlight.js/11.9.0/styles/default.min.css">
<script src="https://cdnjs.cloudflare.com/ajax/libs/highlight.js/11.9.0/highlight.min.js"></script>

<script>hljs.highlightAll();</script>
```

当然这只是对没有应用过`highlight.js`的网页启用代码高亮的例子。对于Hugo和PaperMod，我们还需要做一些魔改；具体来说是首先在`config.yaml`里禁用自带的代码高亮但保留生成网页时对代码块的特殊标签，以便我们应用自己的`highlight.js`：

```yaml
markup:
  highlight:
    codeFences: false
    guessSyntax: false
    noClasses: false

params:
  assets:
    disableHLJS: true # to disable highlight.js
```

之后还需要在PaperMod代码块部分的样式表`themes\PaperMod\assets\css\common\post-single.css`中把和颜色有关的部分注释掉，以及一些切换主题时可能会导致代码块错误地按空格换行的样式：

```css
.post-content code {
    margin: auto 4px;
    padding: 4px 6px;
    font-size: 0.78em;
    line-height: 1.5;
    /* background: var(--code-bg); */
    border-radius: 4px;
}

.post-content pre code {
    /* display: grid; */
    margin: auto 0;
    padding: 10px;
    /* color: rgb(213, 213, 214); */
    /* background: var(--code-block-bg) !important;
    border-radius: var(--radius); */
    overflow-x: auto;
    /* word-break: break-all; */
}
```

