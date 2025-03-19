---
title: "深夜闲谈 0x02"
date: 2025-03-18T00:50:15+08:00
tags: ["2025-03", "杂谈", "深夜闲谈", "Cloudflare", "Hugo", "PaperMod"]
categories: ["深夜闲谈"]
toc: false
draft: true
---

### 0x02

又到了晚上闲谈时间——

打算重新把自动控制原理捡起来，虽然以前学过但总归是印象不深，现在打算再照着课本复习一遍，顺便把以前学习时的心得体会记录在博客上。其实这门学科可以说是工科课程的代表：用（相对来说）不那么严谨的数学去解决具有强现实意义的工程问题，同一个问题可以有许多方法相互验证，殊途同归，当然还少不了大量充满奇怪参数的经验公式XD

---

### 用Cloudflare Tunnel隐藏RESTful API的源地址

最近研究了Cloudflare Workers，想把之前一直跑在本地小香橙派上的Minecraft服务器白名单自助注册后端迁移上去以避免家宽提供HTTP服务被运营商扫到ban宽带的问题。原本的流程是本地跑着用Python Flask写的简易后端，维护一个白名单列表并与远程的Minecraft服务器通过RCON同步；前端是一个托管在Cloudflare Pages上的静态页面，用户填完信息后点击确定就会发起XHR请求把数据POST到后端API，后端验证Token正确、确实不在白名单里就会执行`whitelist add <username>`把用户添加到白名单。Token是预共享的，本来也只是为了把扫IP的机器人挡在外面，不需要很高的安全性。

但是研究半天Node.js后写好了代码，测试的时候怎么都不行。一查发现Cloudflare Workers不支持发起TCP请求，也就不能直接与Minecraft服务器通信；退而求其次，尝试让Workers与现有的后端通信，Workers本身起一个隐藏源站地址和流量筛选的作用也不行，Workers只能与常见的80 443 8080等端口[^1]通信，而本地的后端为了安全性没开这些端口，用了一个高位的端口提供服务。

最后发现了解决方案：直接使用Cloudflare Tunnel作为反向代理，也就不用暴露本地的后端服务到公网上了。为了使用Tunnel，需要在本地的机器上跑一个`cloudflared`客户端。这个客户端原生支持`armhf`，直接从Github下载deb包后`dpkg -i`安装，再从Tunnel控制台复制带Token的命令在本地运行就可以了。

---

### 将博客从Github Pages迁移到Cloudflare Pages

今天还将博客从Github Pages上面迁移到了Cloudflare Pages上。迁移的过程意外的简单，直接在Cloudflare Pages控制台里指定从Github仓库导入，指定好发布时使用的文件夹就可以了。实际上Pages还能够直接在它的云端运行Hugo生成静态页面，但我实测CF的Hugo版本太低，而且我之前也一直是本地生成后push到Github部署的；于是直接选择部署类型为None，指定好存放静态页面的文件夹后保存部署即可。

之后去Github仓库里把Github Pages自动部署删除并释放CNAME域名，再在CF里将这个域名指派给Cloudflare Pages就可以了。证书什么的CF都会自动处理好，用户只需要点点鼠标就可以启用HTTPS加密。

迁移过来后本来还想着去百度注册一下站点，结果要填一大堆信息（从姓名填到微信QQ号），想想还是算了。

---

### 改变Hugo生成的文章URL格式

之前的文章都是按标题生成URL的（比如<https://blog.sorali.org/2024/08/%E7%A1%AC%E6%94%B9%E8%81%94%E6%83%B3m910q%E5%B0%8F%E4%B8%BB%E6%9C%BA%E5%A2%9E%E5%8A%A0%E7%AC%AC%E4%BA%8C%E4%B8%AAm.2%E6%8E%A5%E5%8F%A3/>），不仅不好复制搜索引擎收录的时候也不太好。于是把每个帖子的源Markdown文件都改成了英文名和连字符的形式，看起来就像这样：<https://blog.sorali.org/articles/2025/03/midnight-chat-0x02/>

另外还加了分类，把帖子放在`content/posts`下，教程放在`content/tutorials`下。然后在`config.yaml`里配置站点文件结构后重新生成站点即可：

```yaml
params:
  mainSections:
    - posts
    - tutorials

permalinks:
  page:
    posts: /articles/:year/:month/:filename/
    tutorials: /tutorials/:year/:month/:filename/
  section:
    posts: /articles/
    tutorials: /tutorials/
```

当然旧中文格式的文件我也都没删，全部保留了。新的`sitemap.xml`中不会包含这些内容，因此它们应该会慢慢被搜索引擎遗忘（

---

### 让Markdown表格居中显示

最近看Bing搜索后台时发现之前写的那篇[硬改联想M910q小主机增加第二个M.2接口](https://blog.sorali.org/articles/2024/08/add-second-m2-slot-on-lenovo-m910q/)文章还挺多人看的，遂补充了一下，修改了一些段落问题，补充了一个BOM表。但是马上就注意到表格居然和其他正文左对齐了，于是研究了一下怎么让它居中：在`themes\PaperMod\assets\css\extended`目录下新建一个`custom.css`（当然也可以用原本就有的那个`blank.css`，但我比较喜欢自己加），粘贴如下内容：

```css
/* Markdown表格居中 */
table{
  width: auto;
  display: table;
  margin-left: auto;
  margin-right: auto;
}
```

之后重新生成站点就能看到表格已经居中了。

---

### 为PaperMod主题增加$\LaTeX$支持

写自控笔记时必然要使用LaTeX插入公式，但PaperMod主题默认并不支持$\LaTeX$。好在主题支持在生成的网页开头/结尾加入元素用于添加自定义的css/js[^2]。我参考了[kiwamizamurai](https://github.com/kiwamizamurai)写的这篇[教程](https://kiwamizamurai.github.io/posts/2022-03-06/)：

首先在博客仓库`layouts/partials`目录下新建一个文件并命名为`extend_head.html`，填充如下内容：

```html
{{ if or .Params.math .Site.Params.math }}
<link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/katex@0.16.21/dist/katex.min.css" integrity="sha384-zh0CIslj+VczCZtlzBcjt5ppRcsAmDnRem7ESsYwWwg3m/OaJ2l4x7YBZl9Kxxib" crossorigin="anonymous">

<!-- The loading of KaTeX is deferred to speed up page rendering -->
<script defer src="https://cdn.jsdelivr.net/npm/katex@0.16.21/dist/katex.min.js" integrity="sha384-Rma6DA2IPUwhNxmrB/7S3Tno0YY7sFu9WSYMCuulLhIqYSGZ2gKCJWIqhBWqMQfh" crossorigin="anonymous"></script>

<!-- To automatically render math in text elements, include the auto-render extension: -->
<script defer src="https://cdn.jsdelivr.net/npm/katex@0.16.21/dist/contrib/auto-render.min.js" integrity="sha384-hCXGrW6PitJEwbkoStFjeJxv+fSOOQKOPbJxSfM6G5sWZjAyWhXiTIIAmQqnlLlh" crossorigin="anonymous"
    onload="renderMathInElement(document.body);"></script>

<!-- for inline -->>
<script>
document.addEventListener("DOMContentLoaded", function() {
    renderMathInElement(document.body, {
        delimiters: [
            {left: "$$", right: "$$", display: true},
            {left: "$", right: "$", display: false}
        ]
    });
});
</script>
{{ end }}
```

以上代码和kiwamizamurai原帖中的代码有些差异，是因为我从$\KaTeX$（解决这个问题的核心，一个用于实现纯前端渲染LaTeX代码的JavaScript库）的官方库中copy了一份新版本的示例，包含更新的版本号和与之对应的sha384。代码的大致功能是从jsDelivr CDN拉取KaTeX的本体和对应的样式表，并在整个页面加载完成后遍历页面中字符将未渲染的、由`$`或`$$`包围的字符渲染成数学公式。

之后还需要在Hugo的配置中将`params.math`设置为`true`，以启用这个扩展的head：

```yaml
params:
  math: true
```

重新用Hugo生成页面，可以看到$\LaTeX$公式已经被正确渲染了。

---

### 为PaperMod主题增加代码块高亮适应亮/暗模式

本来只是自己嫌丑想加这个功能的，结果弄着弄着发现内容还不少，新开了一个文章讲了。可以移步：<>


---

[^1]:<https://developers.cloudflare.com/fundamentals/reference/network-ports/>
[^2]:<https://github.com/adityatelange/hugo-PaperMod/wiki/FAQs#custom-head--footer>
[^3]:<https://highlightjs.org/>
