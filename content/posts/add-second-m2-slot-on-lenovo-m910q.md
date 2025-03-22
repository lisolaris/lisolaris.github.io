---
title: "硬改联想M910q小主机增加第二个M.2接口"
date: 2024-08-29T14:32:33+08:00
tags: ["2024-07", "DIY", "魔改", "MFF", "教程"]
categories: ["教程"]
toc: true
draft: false
aliases:
  - /2024/08/硬改联想m910q小主机增加第二个m.2接口/
---

~~打开VSCode切到博客仓库想写点什么才发现这篇文章还没开始动笔，现在就开始写~~

之前在海鲜市场收了一个联想M910q的MFF小主机，作为家里电脑的换代升级（之前是一台i5-4590的惠普SFF机器，我还挺喜欢这种小型主机的）。之所以买这个型号是因为之前看了许多评测和比较：戴尔和惠普的机器拓展性比较一般；联想同代低一级的M710q芯片组太丐并且没有便宜多少；高一级的M910x又因为加了些用不上的功能而显得太贵了（加了一个PCIE 8x的接口，但由于主机空间和散热受限只能上联想定制的阉割版RX 470显卡；并且为了腾出显卡的空间砍掉了2.5寸SATA硬盘位，然后加了一个M.2 SATA的硬盘位作为补偿）。  

而之所以选择M910q也主要是看中了它能上一个M.2 NVMe硬盘和一个2.5寸SATA硬盘，并且因为主板设计和M910x基本相同，只要做适当改装后能上第二个M.2 SATA或NVMe硬盘的拓展性；同时英特尔在那几年的狂挤牙膏使得只要魔改BIOS就能让200系主板支持6~9代的CPU[^1]（由于BIOS容量有限要支持9代的CPU只能删除6代的微码了，影响不大）。本篇文章的重点即在于如何魔改机器实现上述功能。  

## 改装方案  

如前所述，联想的这几代机器采用了相近的主板设计，能够通过硬件上补齐元件来恢复M.2插槽的功能。但闲鱼上和大量国内的帖子都只说了要补齐一些电容，焊上插槽即可在进系统后被认出来但无法在BIOS内识别，实现的改装并不完美；在查找改机方法的来源时注意到了Github上的这个项目：[badger707/m920q-dual-NVME](https://github.com/badger707/m920q-dual-NVME)，讲的是如何修改M920系列（使用300系芯片组，支持8/9代CPU）让它支持两个M.2插槽。但是在仓库的issue中还有一个帖子：[Interest in working on the M710q/M910Q series as well?](https://github.com/badger707/m920q-dual-NVME/issues/2) 探讨了在200系芯片组的M910系列上做类似魔改的可行性。在这个帖子中，一位欧洲的网友[ct](https://github.com/its-me-ct)比较了M910q和M910x的主板原理图并找到了此M.2插槽SATA功能的电路部分，并指出了[应当如何补齐元件](https://github.com/badger707/m920q-dual-NVME/issues/2#issuecomment-1837616136)；这里要做的修改主要是补充一组100nF的电容和一些电阻，从0R到1K不等；所需元件和焊接位置可以参见下图。  

|类型|规格|参数|需要量|备注|
|:-:|:-:|:-:|:-:|:-:|
|连接器|M.2 M-KEY|4+5 3.2H|1|记得买固定硬盘用的塑料扣|
|电容|0402|100nF ~ 220nF 50V|8|实测100nF可用|
|电阻|0402|0R|7|或者8个，如果不专门另买33R的电阻|
|电阻|0402|1K ~ 100K|1|实测10K可用|
|电容|0805|10uF 6.3V|2|可选|
|电容|0402|47pF|1|可选|
|电阻|0402|33R|1|可选|

{{< figure
    src="https://z4a.net/images/2024/12/25/287552988-81f812ed-ea7a-435d-8dc4-74975cfcf32c.jpg"
    caption="Github用户its-me-ct在他的issue回复中贴出的原图"
    align=center
>}}

改机所必须的元件只有三种：100nF电容，0R电阻，10K电阻。其中左边要移动位置的10K电阻是这个方案实现完美改装的关键，其作用在于让BIOS识别主板型号，从上面移到下面后BIOS中的主板型号应当会从“Board Version B”变为Board Version C”（虽然我没找到这个主板类型的BIOS项具体在哪个位置），从而让BIOS认为第二个M.2插槽是存在的并可以被选中引导系统。图中标示"0R or 33R"的焊盘可以直接使用0R电阻；可选的两种电容焊盘分别在主板正面SATA接口旁和背面M.2插槽旁起滤波作用。但原作者提示：

> There are two larger footprints for 0805 10uF capacitors across the voltage lines to smooth out ripple. In practice they are not required, because the M.2 SSDs are designed to have all required capacitance locally on the drive. If you want to put "weird" expansion cards in this slot, then maybe install the capacitors, otherwise don't bother.

> 供电线上有两个较大的 0805 封装 10uF 电容焊盘，用于平滑纹波。实际上，这些电容并不是必需的，因为 M.2 SSD 本身已经设计有足够的板载电容。如果你打算在这个插槽中使用一些“特殊”的扩展卡，可以安装这些电容，否则就不必费心了。

**如果确实有必要焊接这两个电容请务必先测量焊盘极性**！钽电容有标线的一端是正极（和圆柱形的电解电容相反），焊接前务必注意，否则上电时轻则烧毁电容，重则连着桥片甚至CPU一起带走！

{{< figure
    src="https://z4a.net/images/2024/12/25/0777dc757bf387f416a990f0222d57f4.png"
    caption="M.2贴片插槽的类型"
    align=center
>}}


背面应使用M.2 M-Key 4+5 pin的插槽，在淘宝上非常便宜只要一块多就可以买一个。~~看GitHub上的讨论国外这个小东西似乎非常贵而且没得卖，以至于改机得从旧板子上拆一个下来~~ 由于每个焊盘间距太小，个人推荐使用小刀头+锡膏+助焊剂使劲堆，最后用吸锡带清理干净的焊法。完成焊接后最好使用万用表二极管档测试相邻pin之间是否短路，以及各个GND引脚是否良好导通（可以在网上找到M.2的引脚定义[^2]，有相当多的GND脚）。

{{< figure
    src="https://z4a.net/images/2025/03/17/QQ20241225190520.jpg"
    caption="应在主板背面焊接的插槽（红框为可选滤波电容的焊盘）"
    align=center
>}}

## 改装成果  

在焊接好和电阻电容后装机，应当可以在BIOS中主菜单-系统概述内看到新出现的M.2 Drive 2和M.2驱动器两项，分别对应的是NVMe和SATA固态。由于M.2 M-KEY中SATA和NVMe（即PCIe）引脚存在复用，因此不能通过转接板之类的手段将二者分离，只能择其一使用。把硬盘装上后即可在BIOS引导菜单中设置从此盘位启动，这里不再赘述。

{{< figure
    src="https://z4a.net/images/2024/12/25/BV1452DK4X156YO3.png"
    caption="接在新焊接的M.2插槽上的Intel 傲腾 M10固态"
    align=center
>}}

## M710系列的方案  

在同一个issue下，还有另一位用户[evil2k2](https://github.com/evil2k2)提供了[联想M710系列的改机方案](https://github.com/badger707/m920q-dual-NVME/issues/2#issuecomment-1925888323)。实际上M710和M910的主板设计非常相似（芯片组都属于同一代，很可能引脚定义也差不多所以联想直接把两个系列的几款机器主板做的基本相同），因此要做的修改也十分相似：  

{{< figure
    src="https://z4a.net/images/2024/12/25/302452107-3df6a5fe-ee5e-4043-9209-4daca722efb3.jpg"
    caption="Github用户evil2k2在他的issue回复中贴出的原图"
    align=center
>}}

但我手头并没有这款机器，因此无法验证这个方案是否可行，仅供参考。  

## 总结  

相比于为了第二个M.2插槽购买贵一百多块的M910x，从淘宝购买上述各种元件样品的价格加在一起不会超过10元，还不用牺牲2.5寸的硬盘位；此外，改装后的插槽支持SATA协议（原生的第一插槽只支持NVMe协议）从而可以让直接插上已有的老硬盘而无需2.5寸转接器，实在是非常超值的选择。

此外，联想还有相同外观的M715q产品[^3]，使用的是AMD Zen平台，能上R5-2400GE。由于知名度相比M710/M910更低，机器本身的价格也更低；但那时的AMD就已经在CPU和GPU性能上把同代的Intel甩开了。如果对魔改上第二条M.2硬盘的需求较低，可以考虑买AMD平台的这个机器。

最后要特别感谢的是ct和evil2k2两位大佬的铺路与无偿分享，正是这种精神使得互联网长盛不衰！

[^1]:<https://www.chiphell.com/forum.php?mod=viewthread&tid=1958822>  
[^2]:<https://web.archive.org/web/20140203000954/http://www.orvem.eu/attachments/article/130/M%202%20introduction.pdf>  
[^3]:<https://www.lenovo.com/gb/en/p/desktops/thinkcentre/m-series-tiny/thinkcentre-m715q-tiny/11tc1mt715q#tech_specs>
