---
title: "深夜闲谈 0x01"
date: 2025-02-18T02:06:51+08:00
tags: ["2025-02", "杂谈", "深夜闲谈"]
categories: ["深夜闲谈"]
toc: false
draft: false
---

### 0x01

之前创建这个博客的时候想的是当日记来写的，但是后来不知不觉就写成了某种教程集合……感觉不是很有用的东西就不想放上来了。最近又想起来写博客的目的，遂打算开始一个新的节分类专门放闲扯淡的内容。这一篇是第一篇，所以是0x01.

~~吃完夜宵肚子太饱睡不着来写博客~~

---

### 在PaperMod主题中显示图片的题注

刚刚研究了要怎么在保持当前这个PaperMod主题不变的情况下让帖子中的各个图片的题注（caption）显示出来。之前我一直用的是Markdown的图片语法`![caption](link)`，本来以为这个图片的题注是由配置文件中某个项决定的，查了一圈并没有；退而求其次先研究居中，对照浏览器开发者工具的样式表详情仔细检查PaperMod的模板代码，注意到`assets\css\common\post-single.css`里有一部分与图片格式有关：

```css
.post-content img {
    border-radius: 4px;
    margin: 1rem 0;
}

.post-content img[src*="#center"] {
    margin: 1rem auto;
}
```

往下一滑，发现了和caption有关的一部分样式：

```css
.post-content figure > figcaption {
    color: var(--primary);
    font-size: 16px;
    font-weight: bold;
    margin: 8px 0 16px;
}

.post-content figure > figcaption > p {
    color: var(--secondary);
    font-size: 14px;
    font-weight: normal;
}
```

于是去搜了下怎么用这个figure，在PaperMod的一篇[FAQ](https://adityatelange.github.io/hugo-PaperMod/posts/papermod/papermod-faq/#centering-image-in-markdown)里找到如何将图片居中的方式；另外这里还提到了怎么把figure设置成居中，并指向Hugo官方的一个[FAQ](https://gohugo.io/shortcodes/figure/)页面。Hugo给出在Markdown帖子中应该这样写：

```txt
{{</*figure
  src="/images/examples/zion-national-park.jpg"
  alt="A photograph of Zion National Park"
  link="https://www.nps.gov/zion/index.htm"
  caption="Zion National Park"
  class="ma0 w-75"
*/>}}
```

Hugo会把这部分渲染成如下的HTML代码：

```html
<figure class="ma0 w-75">
  <a href="https://www.nps.gov/zion/index.htm">
    <img 
      src="/images/examples/zion-national-park.jpg" 
      alt="A photograph of Zion National Park"
    >
  </a>
  <figcaption>
    <p>Zion National Park</p>
  </figcaption>
</figure>
```

这样就能显示出来一幅像下面这样可以点击并且带有题注的图片了：

{{<figure
  src="https://gohugo.io/images/examples/zion-national-park.jpg"
  alt="A photograph of Zion National Park"
  link="https://www.nps.gov/zion/index.htm"
  caption="Zion National Park"
  class="ma0 w-75"
>}}

讨厌的是VSCode会把上面`link`后面的链接标黄提示不符合Markdown语法，之后研究下能不能自定义格式。

---

### 将已有的帖子中的Markdown图片语法转换为Hugo figure语法

前面提到我之前的帖子里用的都是Markdown的图片语法。一个一个手动编辑显然太麻烦，能不能用程序替换，或者退而求其次用手动搜索-替换来实现呢？当然是可以的。回忆一下正则表达式语法，显然有一个子匹配的功能可以实现把形如

```markdown
![微信小程序 HAM模拟考试](https://z4a.net/images/2024/12/27/44cb47a7b13894a7d2846d05cd761cd5.png)
```

替换成

```txt
{{</*figure
    src="https://z4a.net/images/2024/12/27/44cb47a7b13894a7d2846d05cd761cd5.png"
    caption="微信小程序 HAM模拟考试"
    align=center
*/>}}
```

的功能。

正则表达式的子匹配语法将表达式本身作为一个匹配，并且每个括号里的内容视作一个子匹配，在支持的编辑器（比如Notepad++、KDE Kate，顺带一提我不喜欢npp作者喜欢大放厥词的性格，现在已经卸载转向性能稍微差一些的Kate了）可以用`\0`取得表达式的完整匹配，`\1`取得第一个括号对应的子匹配，以此类推。

在VSCode中子匹配不是用`\0` `\1`这样的语法，而是`$0` `$1`，比如下面：

```
^!\[(.+)\]\((.+)(#center)?\)
```

```txt
{ {< figure\n    src="$2"\n    caption="$1"\n    align=center\n>}}
```

`$2`对应第二个`(.+)`匹配到的图片URL，`$1`对应第一个`(.+)`匹配到的图片题注，`\n`用来换行，这样就能把Markdown图片语法转成Hugo支持的HTML标签嵌入语法了。

---

### 改变PaperMod主题的正文宽度

顺便还改了一下主题中用来规定正文宽度的样式，在themes\PaperMod\assets\css\core\theme-vars.css这个表里：

```css
:root {
    --main-width: 1024px;
}
```

把`--main-width`从720px改成了1024px，宽一些更容易看了。

---

### 使用git submodule管理和更新主题

但是我刚刚注意到可能是由于这个主题是我在创建时直接用`git submodule`从PaperMod的仓库里拉取下来的，在commit完push上去之后GitHub似乎会自动尝试向PaperMod的主仓库push我的修改？还是因为刚刚不小心在VSCode里点到了checkout的原因，总之Github Pages的自动部署报错：

```
Run actions/checkout@v4
Syncing repository: lisolaris/lisolaris.github.io
Getting Git version info
Temporarily overriding HOME='/home/runner/work/_temp/be930d38-359c-4a0e-9514-fcfb6eb17a29' before making global git config changes
Adding repository directory to the temporary git global config as a safe directory
/usr/bin/git config --global --add safe.directory /home/runner/work/lisolaris.github.io/lisolaris.github.io
Deleting the contents of '/home/runner/work/lisolaris.github.io/lisolaris.github.io'
Initializing the repository
Disabling automatic garbage collection
Setting up auth
Fetching the repository
Determining the checkout info
/usr/bin/git sparse-checkout disable
/usr/bin/git config --local --unset-all extensions.worktreeConfig
Checking out the ref
Setting up auth for fetching submodules
Fetching submodules
  /usr/bin/git submodule sync --recursive
  /usr/bin/git -c protocol.version=2 submodule update --init --force --depth=1 --recursive
  Submodule 'themes/PaperMod' (https://github.com/adityatelange/hugo-PaperMod.git) registered for path 'themes/PaperMod'
  Cloning into '/home/runner/work/lisolaris.github.io/lisolaris.github.io/themes/PaperMod'...
  Error: fatal: remote error: upload-pack: not our ref 1f90601ef287fd16dd7290695128e946e2aae293
  Error: fatal: Fetched in submodule path 'themes/PaperMod', but it did not contain 1f90601ef287fd16dd7290695128e946e2aae293. Direct fetching of that commit failed.
  Error: The process '/usr/bin/git' failed with exit code 128
```

感觉是因为我错误执行git checkout导致的。如果不行的话估计得删掉重新从PaperMod的仓库里导入一次了……

---

讲到这个，之后还打算把博客从Github Pages迁移到Cloudflare Pages上。在国内访问的话不施魔法似乎CF的更有可能直连上；之前尝试过在CF上部署一个静态网页用来给玩MC的群友提供白名单自助注册功能，效果还不错，而且Cloudflare Pages似乎也支持自动跟踪Github仓库的更新来刷新页面。明天试试看。

现在打算把这篇文章push进仓库，看看Github Pages还能不能正常部署；不行的话真得改bug了。

---

最近还买了张30HX矿卡，动手补上电容现在正常使用中。过程和之前的折腾内容准备补充进前两天新建的那个空帖子里。但现在得先把这个Pages不能部署的问题解决一下（

---

刚刚试图渲染站点的时候Hugo还报错

`深夜闲谈01.md:1:1": got named parameter 'src'. Cannot mix named and positional parameters`

但是又不说对行号我怎么排查问题==

Google一圈都没有和我这个症状一样的问答，想着可能是Hugo版本太老有bug（这时我还用的0.111.2）；在Hugo的Github上找到了最新的release 0.144，两小时前刚发布。现在虽然还是报错但是能正常显示行号了：

`Error: error building site: process: readAndProcessContent: "深夜闲谈0x01.md:111:17": got named parameter 'src'. Cannot mix named and positional parameters`

都把出问题的这一句包裹在代码块里了还尝试着去渲染吗……使用暴力手段（加一个空格破坏shortcode语法），问题解决。

__2025年3月19日：发现Hugo其实提供了让shortcode不被自动匹配的方法，只要将`{{</* */>}}`中的内容用`/**/`包围起来就可以了。__

---

再次尝试渲染，又报错：

`ERROR deprecated: .Site.Social was deprecated in Hugo v0.124.0 and subsequently removed. Implement taxonomy 'social' or use .Site.Params.Social instead.`

这次是模板问题。但我刚刚又在子仓库中执行了`git checkout`，~~陷入了git的陷阱~~

好在问过ChatGPT后说可以很简单地重置子仓库到初始状态：

```sh
git submodule update --init --recursive
```

然后就能让子仓库和其远程仓库同步了：

```sh
git submodule update --remote
```

PaperMod肯定会跟进Hugo的更新，就不用我自己一个一个地方检查哪里不符合新的语法再去改了，问题解决。

---

解决这个问题后Hugo又报错：

`ERROR deprecated: .Site.Social was deprecated in Hugo v0.124.0 and subsequently removed. Implement taxonomy 'social' or use .Site.Params.Social instead.`

这次是`config.yml`中的问题，查阅最新的Hugo文档中有关[如何分页](https://gohugo.io/templates/pagination/)的部分修改配置文件，问题解决。

---

最后push到远程仓库上，Github自动部署成功，问题圆满解决！
