---
title: "为PaperMod主题增加代码语法高亮适应亮/暗模式"
date: 2025-03-19T18:33:26+08:00
tags: ["2025-03", "Hugo", "PaperMod", "JavaScript"]
categories: [""]
toc: false
draft: false
---

PaperMod主题虽然挺简洁的，但是它原生的代码块只有黑色背景，在白色主题下看起来相当丑，遂想办法解决。

查了一番后发现Hugo使用的是一个叫做`highlight.js`[^1]的项目，并且可以直接在网页头部导入这个JavaScript：

```javascript
<link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/highlight.js/11.9.0/styles/default.min.css">
<script src="https://cdnjs.cloudflare.com/ajax/libs/highlight.js/11.9.0/highlight.min.js"></script>

<script>hljs.highlightAll();</script>
```

当然这只是对没有应用过`highlight.js`的网页启用代码高亮的例子。对于Hugo和PaperMod，我们还需要做一些魔改；具体来说是首先在`config.yaml`里禁用自带的代码高亮但保留生成网页时对代码块的特殊标签，以便我们应用自己的`highlight.js`：

```yaml
markup:
  highlight:
    codeFences: false   # 禁用Hugo自带的代码高亮
    guessSyntax: false
    noClasses: false    #  总是生成<code>标签class里的language-<语言> 属性（由用户编写Markdown时标记），让我们自己导入的highlight.js读取使用

params:
  assets:
    disableHLJS: true   # 禁用PaperMod的代码高亮
```

然后还需要在PaperMod代码块部分的样式表`themes\PaperMod\assets\css\common\post-single.css`中把和颜色有关的部分以及一些切换主题时可能会导致代码块错误地按空格换行的样式注释掉：

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

不过现在碰到了一个问题，怎么检测当前主题是亮色还是暗色模式呢？直接按F12打开开发者工具看看暗色和亮色有什么不同：

```html
...
<body id="top" class="list">
...
```

```html
...
<body id="top" class="list dark">
...
```

注意到切换暗黑模式后，`document.body`的`class`新增了一个`dark`属性，而这部分代码是在PaperMod的`themes\PaperMod\layouts\partials\header.html`文件中实现的：

```html
{{- if (not site.Params.disableThemeToggle) }}
{{- if (eq site.Params.defaultTheme "light") }}
<script>
    if (localStorage.getItem("pref-theme") === "dark") {
        document.body.classList.add('dark');
    }
</script>
{{- else if (eq site.Params.defaultTheme "dark") }}
<script>
    if (localStorage.getItem("pref-theme") === "light") {
        document.body.classList.remove('dark')
    }
</script>
```

实际上还有其他判断是否启用主题切换/检查默认主题之类的逻辑，不过我们先不管它，只要关心`document.body.classList`里是否有`dark`属性就可以了。在使用之前，我们先在`config.yaml`里规定一个flag，用于启用外部的`highlight.js`：

```yaml
params:
  useExternalHLJS: true
```

之后依[为PaperMod主题增加LaTeX支持]({{<siteurl>}}articles/2025/03/midnight-chat-0x02/#%E4%B8%BApapermod%E4%B8%BB%E9%A2%98%E5%A2%9E%E5%8A%A0latex%E6%94%AF%E6%8C%81)的经验，修改站点目录中的`layouts\partials\extend_head.html`，加入这段代码：

```html
{{if site.Params.useExternalHLJS}}
<!-- Enable highlight.js code syntax highlighter -->
<link id="hljs-theme" rel="stylesheet" href=""/>
<script src="https://cdnjs.cloudflare.com/ajax/libs/highlight.js/11.11.1/highlight.min.js" integrity="sha512-EBLzUL8XLl+va/zAsmXwS7Z2B1F9HUHkZwyS/VKwh3S7T/U0nF4BaU29EP/ZSf6zgiIxYAnKLu6bJ8dqpmX5uw==" crossorigin="anonymous" referrerpolicy="no-referrer"></script>
<script>
    document.addEventListener("DOMContentLoaded", function(){
        function SelectThemeAndRefresh(){
            let theme = document.body.classList.contains("dark") ? "dark" : "light";
            document.getElementById("hljs-theme").href = `https://cdnjs.cloudflare.com/ajax/libs/highlight.js/11.11.1/styles/atom-one-${theme}.min.css`;
            hljs.highlightAll();
        }
        SelectThemeAndRefresh();
        const observer = new MutationObserver(mutations => {
            mutations.forEach(mutation => {
                if (mutation.attributeName === "class"){
                    for (let pre of document.getElementsByTagName("pre")){
                        pre.querySelector("code").classList.remove("hljs");
                        pre.querySelector("code").removeAttribute("data-highlighted");
                    }
                    SelectThemeAndRefresh();
                }
            });
        });
        
        observer.observe(document.body, {attributes: true, attributeFilter: ["class"] });
    });
</script>
{{end}}
```

上面的代码结合了一些我之前编写[知乎推荐流优化]({{<siteurl>}}articles/2024/06/optimize-zhihu-web-page-with-additional-javascript/)JavaScript脚本时的经验。这里选用了`Atom One`主题，这是Atom编辑器[^2]默认的配色方案，在cdnjs上亮/暗两种配色css文件的URL恰好只有`light`和`dark`的区别。我们首先封装一个`SelectThemeAndRefresh()`函数，根据`document.body.classList`中是否有`dark`属性决定使用哪一种配色方案的css并随之渲染页面上的所有代码块。

等待页面加载完成后我们在`document.body`上插入一个`MutationObserver`监视器，当它的`class`属性发生变动时即清除所有代码块的已渲染标记（由`highlight.js`在渲染后自动生成），再次调用`SelectThemeAndRefresh()`以新的css主题渲染代码块。由于`MutationObserver` API是JavaScript规范、浏览器实现的，加上博客只是一个静态页面，所以并不会对性能产生多少影响。

这里还使用了cdnjs提供的JavaScript CDN服务，当然也能用其他CDN[^3]或是直接将`css`和`highlight.js`文件下载下来再上传到服务器。因为本站是托管在Cloudflare上的，能访问本站的用户访问<https://cdnjs.cloudflare.com>应该也是比较快的。

保存所有文件并重新生成部署站点，现在点击左上角主页旁切换亮/暗配色的按钮时，文章里的代码块也应当一起跟着变色了。如果在本地测试一切正常，但服务器的静态站点上始终不生效，并且在开发人员工具中看到样式表还是旧的（`.post-content pre code`中仍然存在`background`、`color`等应该已经被注释掉了的属性），很可能是CDN缓存了css之类的静态资源，可以在后台手动清除[^4]：

{{< r2figure
    r2path="code-highlight-adjusting/caching_configuration_cloudflare.png"
    caption="在Cloudflare管理面板中清除域名下的CDN缓存文件"
    align=center
>}}

之后在浏览器中使用`Ctrl+Shift+R`组合键完全刷新页面，应该就能看到效果了。

Enjoy it :D

---

如果对更多关于PaperMod的魔改感兴趣，可以看看我之前写的文章[深夜闲谈 0x02]({{<siteurl>}}articles/2025/03/midnight-chat-0x02/)，我在那里写了如添加$\LaTeX$支持、增加评论功能等的方法。

[^1]:<https://highlightjs.org/>
[^2]:<https://github.com/atom/atom/tree/master/packages/one-light-syntax>
[^3]:可以参见<https://highlightjs.org/#usage>中各个JavaScript CDN的示例
[^4]:<https://developers.cloudflare.com/cache/how-to/purge-cache/purge-by-single-file/>