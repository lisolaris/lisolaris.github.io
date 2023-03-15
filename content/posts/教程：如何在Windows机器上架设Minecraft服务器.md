---
title: "教程：如何在Windows机器上架设Minecraft服务器"
date: 2023-03-14T12:53:17+08:00
tags: ["2023-03", "教程", "Minecraft"]
categories: ["如何架设Minecraft服务器"]
toc: true
draft: false
---

本篇是应群友需求，来介绍一下如何在Windows机器上启动Minecraft服务端供内网用户游玩，以使用服务端上才能运行的插件之类的。关于如何让别人远程连接到你的服务器，可以参见下一篇[教程：如何在服务器上使用内网穿透](https://blog.sorali.org/2023/03/%E6%95%99%E7%A8%8B%EF%BC%9A%E5%A6%82%E4%BD%95%E5%9C%A8%E6%9C%8D%E5%8A%A1%E5%99%A8%E4%B8%8A%E4%BD%BF%E7%94%A8%E5%86%85%E7%BD%91%E7%A9%BF%E9%80%8F/)
。  

## 准备工作  

首先呢我们需要准备一个服务端，这里选用[PaperMC](https://papermc.io/)，访问官网后点进右上角Downloads按需下载版本就行了。下载后得到的一个jar文件，先放着。  

![下载PaperMC服务端](https://s3.bmp.ovh/imgs/2023/03/14/083a5e19459dc903.png)

PaperMC属于Bukkit系的服务端，兼容大部分Bukkit和Spigot的插件，同时应用了许多针对原版的优化，令其运行速度大大快于原版vanilla；但代价就是许多原版的~~bug~~特性在Paper端不再可用，需要在高级配置中开启或是安装额外的插件（比如用于造刷沙机的跨世界实体方块更新逻辑在Paper端就需要安装[这个插件](https://github.com/Nats-ji/paper-sand-dupe-unpatched)）。  

需要注意的是官网首页的版本是跟进到最新稳定版的，比如目前是1.19.3，那么1.19.2等不再支持的客户端需要一直往下滑点击More才能看到。此外，更旧的不再支持的大版本服务端（如1.18.2及更早）被归在Legacy里面，点进去后会提示这些版本不再受支持，你提出的任何bug都不会修补云云。  

为了让游戏运行起来，我们还需要Java运行环境。你可以下载[OpenJDK](https://jdk.java.net/19/)或是[Oracle JDK](https://www.oracle.com/java/technologies/downloads)，二者的区别在开源与否。  

此外，从Minecraft 1.17开始，Mojang就要求玩家使用16及更高版本的JRE，因此我推荐你下载最新的JDK。从我个人的使用体验而言，OpenJDK和Oracle JDK并没有察觉到明显的差异，下载哪个都无所谓了……不过我推荐下载zip打包的JDK，可以省去配置环境变量这一步，后续要升级也很方便。  

![OpenJDK Windows版下载](https://s3.bmp.ovh/imgs/2023/03/14/d790612a511af346.png)

## 让服务器跑起来  

### 准备好文件  

现在我们就有了一个.zip的JDK压缩包，一个.jar的Java软件包。我们找个文件夹把它们装起来，解压JDK，看起来像是这样的：  

![文件列表](https://s3.bmp.ovh/imgs/2023/03/14/04cae75d09bef35d.png)

打开得到的jdk文件夹，进入bin目录内，找到java.exe，__按住Shift的同时右键它__，在弹出的菜单中选择『复制文件地址』：  

![复制文件地址](https://s3.bmp.ovh/imgs/2023/03/14/6895b0ce3da56561.png)

如果想运行安装了Forge或者Fabric的Mod服务器，之后我有空会再补充的。这里先介绍原版。

### 启动脚本  

回到刚刚的目录，新建一个文本文档，填入刚刚复制的java.exe的文件地址，在后面加入参数```-Xmx<最大限制内存>M -Xms<最小分配内存>M -Dfile.encoding=utf-8 -Dnative.encoding=utf-8 -jar <服务端文件路径>```，比如像这样：  

```Windows Powershell
chcp 65001
ipconfig /all
"D:\MinecraftServer\jdk-19.0.2\bin\java.exe" -Xmx4096M -Xms768M -Dfile.encoding=utf-8 -Dnative.encoding=utf-8 -jar "D:\MinecraftServer\paper-1.19.3-448.jar"
pause
```

解释一下：  

> + 最大限制内存：允许Java虚拟机使用的最大的内存，可以理解为服务端使用的最大内存量。应当按照你电脑拥有的实际内存来设置。一般有16G内存的电脑，和三五个好友一起玩设置为4096(4GB)已经够用了，人数更多可酌情调整，但也需要更强的CPU性能。  
> + 最小分配内存：至少给Java虚拟机分配这么多内存，合理小于最大限制就可以了。  
> + 服务端文件路径：可以用和复制java.exe的文件路径相同的方法得到，粘贴进去就可以了。注意双引号不要删掉。
> + ```-Dfile.encoding=utf-8 -Dnative.encoding=utf-8```：这两个参数用于指定使用的文本编码，避免控制台日志的中文显示为乱码。  

保存文件，将后缀名改为bat，双击运行一次；第一次的运行会自动下载原版的服务端并在它的基础上修改（为了规避版权问题不能直接提供修改好的服务端），需要一些时间。等它下完开始滚动日志后没一会就会报错停止，这是正常的，因为我们还没有同意最终用户许可协议（同样是版权问题）。  

![第一次运行](https://s3.bmp.ovh/imgs/2023/03/14/aa08d529a007c044.png)

这时在游戏文件夹里已经生成了一堆文件和文件夹，暂时不管。我们打开eula.txt，把最后一行的```false```改成```true```，保存，再次启动服务端，正常运行。

![EULA](https://s3.bmp.ovh/imgs/2023/03/14/e39bf04e4bf76753.png)

弹出防火墙提示的话记得允许放行。

![防火墙](https://s3.bmp.ovh/imgs/2023/03/14/26dfa7f3cd600c9d.png)

这个时候可以看到命令行窗口最后一行日志是```Timings reset```，并有一个```>```命令提示符等待用户输入。以后可以从这里输入Minecraft指令了。

### 在客户端中填入服务器地址  

我们打开游戏，启动多人模式，添加一个服务器为```127.0.0.1:25565```看看能不能正常进去：

![第一次进入](https://s3.bmp.ovh/imgs/2023/03/14/4d11790f64c060ff.png)

打开F3调试选项，左上角可以看到进入了"Paper"服务器。  

### 配置服务端  

在控制台输入```stop```后回车停止服务端。然后打开服务端目录下的server.properties，对服务端进行配置。这里可以参考我的：  

```ini
#Minecraft server properties
#Mon Jan 16 16:16:38 CST 2023
allow-flight=true
allow-nether=true
broadcast-console-to-ops=true
broadcast-rcon-to-ops=true
debug=false
difficulty=normal
enable-command-block=true
enable-jmx-monitoring=false
enable-query=false
enable-rcon=false
enable-status=true
enforce-secure-profile=true
enforce-whitelist=false
entity-broadcast-range-percentage=100
force-gamemode=false
function-permission-level=2
gamemode=survival
generate-structures=true
generator-settings={}
hardcore=false
hide-online-players=false
initial-disabled-packs=
initial-enabled-packs=vanilla
level-name=world
level-seed=
level-type=default
max-chained-neighbor-updates=1000000
max-players=20
max-tick-time=60000
max-world-size=29999984
motd=
network-compression-threshold=256
online-mode=false
op-permission-level=4
player-idle-timeout=0
prevent-proxy-connections=false
previews-chat=false
pvp=true
query.port=25565
rate-limit=0
rcon.password=
rcon.port=25575
require-resource-pack=false
resource-pack=
resource-pack-prompt=
resource-pack-sha1=
server-ip=
server-port=25565
simulation-distance=10
spawn-animals=true
spawn-monsters=true
spawn-npcs=true
spawn-protection=16
sync-chunk-writes=true
text-filtering-config=
use-native-transport=true
view-distance=10
white-list=false
```

对于普通玩家，我们主要关心几个选项：

> + ```server-port```：服务器端口，默认为25565。当使用此端口时其他客户端输入可以省略，但如果冲突的话需要重新设置一个。我推荐设置一个10000~65535的高端口，避免冲突。  
> + ```view-distance```：可视距离，设置的越大性能消耗越高。设置为10（10个区块）就已经够了。  
> + ```online-mode```：是否开启正版验证。如果你的玩家有人在玩盗版（离线验证）就应该关掉。但如果大家玩的都是正版但用的是如HMCL的第三方启动器也建议关掉，因为第三方启动器的登陆状态总有问题，开着的话经常会登不上。  
> + ```motd```：服务器在多人游戏的服务器列表里显示的标题，但是不装插件好像不支持中文。  

其他的选项可以在网上找到参考资料，这里就不赘述了。

写好配置文件后就来到重头戏——如何让别人连接到你？

## 让内网用户连接  

如果你只是想在家/宿舍/校园网内/~~公司/实验室~~里和小伙伴一起玩的话，那么下一篇的内容就不用再看，照本节操作就可以了。反之你可以看看[不在同一个网络内，通过公网连接](https://blog.sorali.org/2023/03/%E6%95%99%E7%A8%8B%E5%A6%82%E4%BD%95%E5%9C%A8windows%E6%9C%BA%E5%99%A8%E4%B8%8A%E6%9E%B6%E8%AE%BEminecraft%E6%9C%8D%E5%8A%A1%E5%99%A8/#%E4%B8%8D%E5%9C%A8%E5%90%8C%E4%B8%80%E4%B8%AA%E7%BD%91%E7%BB%9C%E5%86%85%E9%80%9A%E8%BF%87%E5%85%AC%E7%BD%91%E8%BF%9E%E6%8E%A5)
了解如何让别人远程连接到你的服务器。

我们前面让Windows防火墙放行了服务端，此时理论上和你在同一个网络内（比如说宿舍或者公司的公用路由器下）的用户就可以直接添加你电脑的IP和端口作为客户端加入游戏了；当然，要填对IP才行。

### 在同一个网络内  

这里我们启动刚刚写好的启动服务端的脚本，向上滚动命令行窗口旁边的滚动条直到最顶端，看到第二行的命令```ipconfig /all```产生的输出：

![ipconfig](https://s3.bmp.ovh/imgs/2023/03/14/119cbbaee405462f.png)

找到我们正在用的适配器（一般是以太网或者WLAN，会显示得非常详细，这里我把右边的输出全部隐去了）所对应的IPv4地址，那么就是这个了。让其他玩家在他们的客户端中输入这个IP和之前设置的端口，就可以连上了。

### 在同一个大局域网（如校园网）内  

首先我们需要明确，有些管理严格的学校会禁止校园网内的客户端互相访问，此时你应当跳到下一章按照公网方式连接；但绝大部分学校应该都不会这么做。

但即使是校园网，不同的联网方式操作起来也有所不同：

#### 电脑直接接在网线接口上，通过拨号认证上网  

这种情况你在上节查看IP地址时应该会列出一个PPPoE适配器，让你的朋友填入那个地址即可。但是只有在同一个校园网的玩家才能这么连。

#### 电脑直接接在网线接口上，通过网页认证上网  

实际上我没有使用过这种认证方式……此时按照上一节查看IP地址的方法，直接填入以太网或WLAN适配器的地址即可。

#### 电脑连接到路由器，由路由器拨号上网  

部分学校允许学生使用路由器拨号。此时你需要设置端口转发或DMZ主机，这里以小米路由器为例：  

打开浏览器登陆到路由器后台（对于小米路由器，一般是<http://192.168.31.1>），选择高级设置-端口转发，添加规则：  

![端口转发](https://s3.bmp.ovh/imgs/2023/03/14/6c8363ca429d2e30.png)

![设置规则](https://s3.bmp.ovh/imgs/2023/03/14/81a30ec78ecb1486.png)

各个项目的解释如下：  

> + 名称：随意设置即可。
> + 外部端口：可以填25565，也可以填一个你喜欢的，但这样就需要其他玩家连接时输入这个外部端口，而不是上面设置的端口了。
> + 内部IP地址：填你的电脑的内网IP地址，参见『在同一个网络内』一节。
> + 协议：不用改动，设置为默认的TCP就可以。
> + 内部端口：填之前在配置文件里写的端口。

添加后别忘了把页面划到最下面，有个保存并生效的按钮，按了才算成功的。  

然后打开路由器后台的常用设置-上网设置，可以看到上网信息一栏，记下这个IP地址。  

![上网信息](https://s3.bmp.ovh/imgs/2023/03/14/142975bf2873d1a8.png)

对于校园网内的其他用户，在你设置好端口转发后使用刚刚记下的IP地址和外部端口一般就可以连接到你的服务器了，但如果你的校园网重新拨号导致IP地址变动则需要重新在他们的客户端里填入新的IP，稍微有点麻烦。  

## 不在同一个网络内，通过公网连接

关于这个部分可能需要单独写一篇，因为有很多不同的方法，并且也适用于其他游戏的局域网联机（大概）。请参见下一篇：[教程：如何在服务器上使用内网穿透](https://blog.sorali.org/2023/03/%E6%95%99%E7%A8%8B%EF%BC%9A%E5%A6%82%E4%BD%95%E5%9C%A8%E6%9C%8D%E5%8A%A1%E5%99%A8%E4%B8%8A%E4%BD%BF%E7%94%A8%E5%86%85%E7%BD%91%E7%A9%BF%E9%80%8F/)

## 完成！  

现在你已经设置好了，可以愉快地享受游戏了~  

如果还有什么问题的话，可以[发邮件](mailto:sorali@sorali.org)同我联系！祝你玩得开心:D

欲知后事如何，且听下回分解:D
