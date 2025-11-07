---
title: "深夜闲谈 0x03"
date: 2025-10-15T16:19:50+08:00
tags: ["2025-10", "杂谈", "深夜闲谈", "Cloudflare", "Hugo", "PaperMod"]
categories: ["深夜闲谈"]
toc: false
draft: true
---

### 0x03

已经很久没有写过博客了。其实也没什么特别想写的，单纯记录一下最近干了些什么（

---

### 使用 Cloudflare Workers 反向代理 Docker Hub

由于某些神秘原因，在国内是无法直连 Docker Hub 的，并且各个公开镜像站也都消失了。那么如果要访问 Docker Hub 就有两种方式：为 docker 本身配置镜像地址，或为守护进程`dockerd`配置代理。

#### 自建 Docker Hub 反向代理并为 docker 配置

这种方法的原理是国内可以正常访问到 Cloudflare 的相关服务（虽然比较慢），而 Cloudflare 提供了 Workers 这样的云函数计算服务；但这里我们的计算量并不大，更多地是把 Workers 当成一个反向代理服务器使用。

如果不想折腾太多，则只需要在 Cloudflare 上注册开通 Workers, 之后直接将[cloudflare-docker-proxy](https://github.com/ciiiii/cloudflare-docker-proxy)部署到自己账户下的Workers就可以了。

##### 为反向代理的Workers配置其他路由实现复用

使用上面开源项目的默认配置会占用一整个 Workers, 如果我们还想用此域名做其他的事有两种方法：一是在 Cloudflare 后台为 Workers 配置路由，二是直接在 Workers 逻辑中实现不同路由访问不同的功能。第一种方法我暂时还没试过，此处介绍一下第二种方法。

首先我们可以将刚刚项目里的`src/index.js`导入到我们的 Workers 项目中，命名为docker.js;



需要注意的是，对使用免费计划的 Workers, Cloudflare 会限制此 Workers 单次连接能够产生的响应体最大为 100MB[^1], 因此我们自建的这个 docker 反向代理只能缓解有没有的问题，好不好用还是另说（

#### 为守护进程配置代理

如果嫌创建 Cloudflare Workers 太麻烦，在 docker 机器所在的主机或局域网内正好有设备开启了代理服务器功能，那么可以直接设置 docker 使用此代理去连接 Docker Hub.

由 `systemd` 管理运行的服务并不会直接使用当前 shell 环境的代理环境变量，因此需要自己创建相关的配置文件并设置：

```sh
sudo mkdir -p /etc/systemd/system/docker.service.d
sudo vim /etc/systemd/system/docker.service.d/proxy.conf
```

向配置文件中写入代理服务器设置：

```conf
[Service]
Environment="HTTP_PROXY=http://<username>:<password>@<host>:<port>/"
Environment="HTTPS_PROXY=http://<username>:<password>@<host>:<port>/"
Environment="NO_PROXY=localhost,127.0.0.1,.example.com"
```

记得把上面的 `host`, `port` 等替换成你自己的，如果代理服务器没有配置用户名和密码可以直接将 `<username>:<password>@` 删去。

之后需要重载daemon并重启 `dockerd`:

```sh
sudo systemctl daemon-reload
sudo systemctl restart docker
```

这时再进行 `docker pull`, 应该就没有问题了。

### 使用 Cloudflare R2 作为博客图床并自定义 Hugo Shortcode 快速使用

本站之前使用的图床是 <https://z4a.net>, 似乎从几个月前主站就无法访问了 ~~存储在站上的图片也全都消失了~~ 。由于主要的文字页面托管在 Cloudflare 上，此次正好统一将页面和图片都托管在 Cloudflare 上。

[Cloudflare R2](https://developers.cloudflare.com/r2/) 是 Cloudflare 推出的对象存储服务，类似于 AWS S3 与阿里云 OSS, 但价格更加便宜并且有免费计划（免费用户有10GB的存储空间且流量不限），对于博客附图来说还是挺足够的。使用 R2 只需要在 Cloudflare 控制台 Storage & Databases - R2 object storage 中开通并创建一个 bucket 即可。

创建完成后可以从 R2 控制台页面上传文件或是使用 [Cloudflare Wrangler](https://developers.cloudflare.com/workers/wrangler/) 部署工具，或其他任何兼容 S3 API 的操作工具；这里我选择使用 [rclone](https://rclone.org/), 这是一个兼容许多云服务与对象存储提供商的命令行工具。

在使用 rclone 之前，还需要准备一个 API Token 授权访问对应的 bucket. 在 R2 管理页面右下角的账户详情框中可以看到一个 API Token 的管理入口与你账户对应的 S3 API地址：

{{< r2figure
    r2path="midnight-chat/0x03/r2_object_storage_account_details.png"
    caption="Cloudflare R2 API Token 入口"
    align=center
>}}

点进去后创建一个允许读写所有 bucket 的 Token, 得到 `access_key_id` 与 `secert_access_key`; 之后安装 rclone
 并在命令行中运行 `rclone config`, 为这个配置命名（比如就叫做`R2`）后按照指引选择 `Amazon S3 Compliant Storage Providers - Cloudflare R2 storage` 后填入 `endpoint` 为上图中 `S3 API` 的 URL, `access_key_id` 与 `secert_access_key` 为你刚刚创建的即可通过 rclone 操作 R2 中存储的文件了，具体可以参考 [Rclone · Cloudflare R2 docs](https://developers.cloudflare.com/r2/examples/rclone/).

为了组织起文件，我推荐使用目录的形式上传。虽然文件在对象存储中都是平铺在 bucket 中的（也就是不存在目录结构，所有文件都是直接组织在 bucket 的根目录下），但大部分对象存储服务都支持通过在文件名中添加斜杠 `/` 来模拟多层目录；在用 rclone 上传时，它也会自动处理多层目录结构并为文件名添加斜杠。比如，我可以创建一个 blog 目录，用于存放博客中的所有图片，并将这篇文章涉及到的图片放置在 `blog/midnight-chat/0x03/` 目录下：

{{< r2figure
    r2path="midnight-chat/0x03/mutilayer_directories_image.png"
    caption="多层结构放置图片"
    align=center
>}}

然后在 `blog` 目录同级执行

```powershell
rclone copy ./blog R2:blog/blog --update
```

其中 `R2` 是刚刚在 `rclone config` 中填入的配置名。待执行完成后刷新 R2 的控制台，此时应该可以看到已上传到 bucket 内带目录结构的文件了。

为了方便访问 R2, 我们可以为每个 bucket 配置一个子域名。如果已经将域名托管在 Cloudflare, 可以方便地在 bucket 设置中添加一个自定义域名；如果用其他域名服务提供商，可能需要自己去添加一个 CNAME 记录。

然后往下找到 CORS策略，添加 `localhost`、你的博客自定义域名与 `.pages.dev` 的 Pages 域名，可以防止被盗链刷流量：

{{< r2figure
    r2path="midnight-chat/0x03/r2_object_storage_cors_policy.png"
    caption="CORS策略配置"
    align=center
>}}

```json
[
  {
    "AllowedOrigins": [
      "http://localhost:1313",
      "https://blog.sorali.org",
      "http://sorali.pages.dev"
    ],
    "AllowedMethods": [
      "GET"
    ]
  }
]
```

做完以上的准备工作后就可以在博客中通过插入 figure 的方式使用上传到 R2 里的图片了。

```txt
{{</*figure
    src="https://r2.sorali.org/midnight-chat/0x03/mutsumi.png"
    alt="🥒"
    caption="🥒"
    align=center
    width="50%"
*/>}}
```

{{< r2figure
    r2path="midnight-chat/0x03/mutsumi.jpg"
    alt="🥒"
    caption="🥒"
    align=center
    anchor="mutsumi"
    width="50%"
>}}

但是每次都要输入 bucket 的 URL, 总有些不方便。Hugo 提供了自定义 shortcode 的功能，我们可以在站点仓库 `layouts/shortcodes` 目录下创建自定义的 shortcode 来简化操作。这里我们创建一个 `r2figure.html`:

```html
{{- $r2BaseUrl := "https://r2.sorali.org/blog/" -}}
{{- $r2Path := .Get "r2path" -}}
{{- $align := .Get "align" | default "" -}}

{{- $src := printf "%s%s" $r2BaseUrl $r2Path -}}
{{- if $align -}}
  {{- $src = printf "%s#%s" $src $align -}}
{{- end -}}

{{- $caption := .Get "caption" -}}
{{- $link := .Get "link" -}}
{{- $alt := .Get "alt" | default $caption -}}
{{- $class := .Get "class" -}}
{{- $attr := .Get "attr" -}}
{{- $attrlink := .Get "attrlink" -}}
{{- $alt := .Get "alt" | default $caption -}}
{{- $title := .Get "title" | default $caption -}}
{{- $width := .Get "width" -}}
{{- $height := .Get "height" -}}
{{- $loading := .Get "loading" -}}
{{- $decoding := .Get "decoding" | default "async" -}}
{{- $anchor := .Get "anchor" -}}

{{- if $anchor -}}
<span id="{{ $anchor }}"></span>
{{- end -}}
<figure {{ with $align }} class="align-{{$align}}" {{ end }}>
    {{- with $link -}}
        <a id="{{.anchor}}" href="{{ . }}"{{ with $alt }} title="{{ . }}"{{ end }} target="_blank" rel="noopener noreferrer">
    {{- end -}}
    <img src="{{ $src }}"{{ with $width }} width="{{ . }}"{{ end }}{{ with $height }} height="{{ . }}"{{ end }}{{ with $alt }} alt="{{ . }}"{{ end }}{{ with $title }} title="{{ . }}"{{ end }}{{ with $class }} class="{{ . }}"{{ end }}{{ with $loading }} loading="{{ . }}"{{ end }}{{ with $decoding }} decoding="{{ . }}"{{ end }} />
    {{- with $link -}}
        </a>
    {{- end -}}
    {{- with $caption -}}
        <figcaption>
            <p>
            {{- if or $caption $attr -}}
                {{- $caption -}}
                {{- with $attr -}}
                    {{- with $attrlink -}}
                        <a href="{{ . }}" target="_blank" rel="noopener noreferrer">{{ $attr }}</a>
                    {{- else -}}
                        {{- $attr -}}
                    {{- end -}}
                {{- end -}}
            {{- end -}}
            </p>
        </figcaption>
    {{- end -}}
</figure>
```

注意把上面第一行的 `{{- $r2BaseUrl := "https://r2.sorali.org/blog/" -}}` 换成你自己的；之后就可以使用

```txt
{{</*r2figure
    r2path="midnight-chat/0x03/mutsumi.jpg"
    alt="🥒"
    caption="🥒"
    align=center
    anchor="mutsumi"
    width="50%"
*/>}}
```

直接使用相对路径来获取图片了。并且在上面我们添加了一个 `anchor` 参数，允许在插入图片时为图片元素添加一个自定义名称的锚点，这样在文内直接使用 `[文字内容](#mutsumi)` 就可以实现在渲染后的页面上 [点击链接🥒](#mutsumi) 快速定位到图片了。

相比其他的免费图床，使用这种方式虽然稍微麻烦些，但好在不会受图床跑路数据丢失之苦了。

另外注意到一个有趣的事：对 `blog.sorali.org` （我的博客绑定的 Cloudflare Pages 自定义域名）与 `r2.sorali.org` （博客使用的 R2 对象存储 bucket 自定义域名）结果是一样的，也就是说 Cloudflare Pages 和 R2 用的是同一套静态存储容器（大概）。

[^1]: [Limits · Cloudflare Workers docs](https://developers.cloudflare.com/workers/platform/limits/)
