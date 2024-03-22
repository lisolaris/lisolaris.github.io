---
title: "使用LockWorkStation让笔记本的可编程按键发挥作用"
date: 2024-03-22T16:22:45+08:00
tags: ["2024-03", "tips", "HP", "可编程键"]
categories: [""]
toc: true
draft: false
---

总感觉已经许久没有写过blog了……虽然有在给域名续费，但都做的是DDNS方面的用途（顺便推荐一下Cloudflare的DNS解析服务，虽然在国内比较慢但还算方便）。写这个文章主要是想起来之前曾就怎么利用惠普笔记本上自带的F12可编程按键（可以通过系统中的HP Progrommable Key的UWP程序来编辑功能）实现按下即锁定计算机折腾过一番，特来灌水一篇（

~~已经忘记git和hugo该怎么用了，还是临时去查的~~

![在HP Programmable Key中编辑F12键的功能](https://s3.bmp.ovh/imgs/2024/03/22/f455559c421b87f7.png)

## 准备工作

因为并不是多么复杂的程序，所以只需要如果对这部分不感兴趣可以直接拉到文章底部，下载编译好的程序并为编程键设置按下启动即可。

首先可以看看这个API在[MSDN上的文档](https://learn.microsoft.com/en-us/windows/win32/api/winuser/nf-winuser-lockworkstation):它于Windows XP中引入，功能有且只有一个：以类似于Windows键+L的方式锁定工作站（名字都直接写出来了==）。你可以在命令行中使用`rundll32`直接调用到`user32.dll`（对应的头文件为`winuser.h`）中的这个API：

```powershell
rundll32.exe user32.dll,LockWorkStation
```

当然，惠普的这个可编程键并不支持在启动程序时附加参数，也不支持调用bat或者vbs之类的脚本……想要使用可编程键直接锁定电脑只能编译一个程序间接调用这个API了。

## 使用

将下面的代码在Windows平台上使用gcc或msvc编译后，在HP Programmable Key中设置为按下后启动即可。

```C++
#include <Windows.h>
#include <winuser.h>

int main(){
    return(!LockWorkStation());
}
```

使用gcc编译时不需要额外的静态库链接，gcc应当会自动处理与系统自带的`user32.dll`的动态链接：

```powershell
gcc lockstation.cpp -o lockstation.exe
```

这个程序在我的电脑上使用TDM-GCC 10.3.0在Windows 10 专业版 22H2 19045.4170下编译通过并能够正常使用。

编译好的文件可以在这里找到：<https://github.com/lisolaris/lockscreen/releases/tag/alpha>

## 拓展

有关`rundll32`的使用，你可以参考[MSDN](https://learn.microsoft.com/zh-cn/windows-server/administration/windows-commands/rundll32)或是这篇文章[关于利用rundll32执行程序的分析](https://3gstudent.github.io/%E5%85%B3%E4%BA%8E%E5%88%A9%E7%94%A8rundll32%E6%89%A7%E8%A1%8C%E7%A8%8B%E5%BA%8F%E7%9A%84%E5%88%86%E6%9E%90)，挺有趣的。
