---
title: "自动控制原理 - 03 - 线性控制系统的时域分析"
date: 2025-10-20T15:43:12+08:00
tags: ["2025-10", "学习笔记", "控制理论", "自动控制原理", "传递函数", "结构图", "梅逊增益公式"]
categories: ["自动控制原理"]
description: "自动控制原理的学习笔记。本篇主要复习自动控制原理的时域分析法，包括典型一阶系统的时间响应，不同阻尼状态下的典型二阶系统时间响应，高阶系统时间响应的估计，静态/动态误差系数法，以及在时域上做简单反馈/前馈校正的方法。"
toc: true
draft: false
---

从本篇文章开始我们将正式进入控制系统的分析与校正。从时域上对控制系统进行分析符合人类直觉，比起根轨迹法和频域法更有“看得见、摸得着”的感觉；但时域法在对系统参数进行调整以使性能指标朝我们希望的方向进行改变，即系统校正上有所欠缺。即使如此，我们在学习时域法时会接触到许多基本的概念、方法和结论，对后续的控制理论学习都有很大帮助。

## 时域法常用的典型输入信号与性能指标

### 典型输入信号

在时域分析中，我们一般会用到这些典型输入信号：

{{< r2figure
    r2path="control-theory-review/03/lu_table_3-1.png"
    caption="时域分析法中的典型输入信号（54页 表3-1）"
    align=center
>}}

- **单位脉冲函数 $\delta(t)$**：得到的响应即为系统的闭环传递函数。

- **单位阶跃函数 $1(t)$**：又称为位置输入，是工程上最常用的输入信号。由于单位阶跃输入容易实现、对系统稳定能力的要求比较高，故后续的时域分析多以单位阶跃为输入信号展开研究。

- **单位斜坡函数 $t$**：又称为速度输入。

### 性能指标

这部分内容来自卢京潮《自动控制原理》第三章 线性系统的时域分析与校正 3.1.3 系统的常用时域性能指标。

#### 动态性能指标

{{< r2figure
    r2path="control-theory-review/03/lu_figure_3-1.png"
    caption="系统的典型阶跃响应及动态性能指标（54页 图3-1）"
    align=center
>}}

上表是以典型欠阻尼二阶系统的阶跃响应为基础绘制的。当衡量系统的动态性能时，我们一般使用这些指标：

- **上升 (raise) 时间 $t_r$**: 阶跃响应从终值的 $10 \char37$ 上升到终值的 $90 \char37$ 所需的时间；对有振荡的系统，也可定义为从0到第一次达到终值所需的时间。

- **峰值 (peak) 时间 $t_\mathrm{p}$**: 阶跃响应越过终值 $h(\infin)$ 达到第一个峰值所需的时间。

- **调节 (settling) 时间 $t_\mathrm{s}$**: 阶跃响应到达并保持在终值$h(\infin) \pm 5 \char37$ 误差带内所需的最短时间；有时也用终值的 $\pm 2 \char37$ 误差带来定义调节时间。在卢老师的课程中，一般均以终值的 $\pm 5 \char37$ 误差带定义调节时间。

- **超调 (overshoot) 量 $\sigma \char37$**: 峰值 $h(t_\mathrm{p})$ 超出终值 $h(\infin)$ 的百分比，即

$$\sigma \char37 = \frac{h(t_\mathrm{p}) - h(\infin)}{h(\infin)} \times 100 \char37$$

- 延迟 (delay) 时间 $t_d$: 阶跃响应第一次达到终值 $h(\infin)$ 的 $50 \char37$ 所需的时间。

在上述动态性能指标中，工程上最常用的是调节时间 $t_\mathrm{s}$（反映过渡过程的长短，描述
“快”），超调量$\sigma \char37$（反映过渡过程的波动程度，描述“匀”）以及峰值时间 $t_\mathrm{p}$.

#### 稳态性能指标

当我们考虑系统的稳态性能时，实际上我们研究的是系统在达到稳态的情况下，其输出与我们所期望的输出之差值，称为系统的稳态误差。用更精确的语言进行表述：**稳态误差是时间趋于无穷时系统实际输出与理想输出之间的误差，是系统控制精度或抗干扰能力的一种度量。**

应当指出，系统性能指标的确定应根据实际情况而有所侧重。例如，民航客机要求飞行平稳，不允许有超调；歼击机则要求机动灵活，响应迅速，允许有适当的超调；对于一些启动之后便需要长期运行的生产过程（如化工过程等）则往往更强调稳态精度。


## 一阶系统的时域分析

一阶系统的时域分析相对比较简单。我们可以从系统的典型结构图出发进行分析：

{{< r2figure
    r2path="control-theory-review/03/lu_figure_3-2.png"
    caption="一阶系统典型结构图（55页 图3-2）"
    align=center
>}}

系统的传递函数为

$$\mathit{\large \Phi (s)} = \frac{K}{s + K} = \frac{1}{Ts+1}\ ,$$

可以解得系统特征根[^1] $\lambda = -\dfrac{1}{T} \ .$

上式中 $T=\dfrac{1}{K}$ 称为一阶系统的时间常数，在分析一阶系统时有重要意义。

对于一阶系统，在复频域上计算其单位阶跃响应为

$$
C(s)
= \mathit{\Phi (s)} \cdot R(s)
= \frac{1}{Ts+1} \frac{1}{s}
= \frac{1}{s} - \frac{1}{s + \frac{1}{T}}
\ ,
$$

通过拉普拉斯变换即得到时域响应

$$h(t) = \mathscr{L}^{-1}[C(s)] = 1-e^{- \frac{t}{T}} .$$

### 一阶系统的动态性能指标

一阶系统的单位阶跃响应如下图所示，是一条单调的指数上升曲线。

{{< r2figure
    r2path="control-theory-review/03/lu_figure_3-3.png"
    caption="一阶系统的单位阶跃响应（56页 图3-3）"
    align=center
>}}

由调节时间 $t_\mathrm{s}$ 的定义可得

$$h(t_\mathrm{s}) = 1 - e^{\frac{t_\mathrm{s}}{T}} = 0.95 \ ,$$

可以解得

$$t_\mathrm{s} = 2.9957T \approx 3T \ .$$

从上图也可以验证这个结论。图3-3表明，可以用时间常数 $T$ 描述一阶系统的响应特性。时间常数 $T$ 是一阶系统的重要特征参数。$T$ 越小，系统极点越远离虚轴，过渡过程越快[^2]。

类似地，可以讨论一阶系统的脉冲响应和斜坡响应，并将一阶系统对不同典型信号的响应汇总如下：

{{< r2figure
    r2path="control-theory-review/03/lu_table_3-2.png"
    caption="一阶系统典型输入响应（56页 表3-2）"
    align=center
>}}

从表中容易看出，**系统对某一输人信号的微分/积分的响应，等于系统对该输人信号响应的微分/积分。** 这是线性定常系统的重要性质，**对任意阶线性定常系统均适用。**

## 二阶系统的时域分析

二阶系统分析是本章的重点。相比一阶系统，二阶系统引入的额外极点会使得系统不再稳定地趋于收敛，而是依阻尼比不同产生震荡甚至发散；下面我们来详细讨论这一点。

典型的二阶系统结构图如图3-6 (a)所示，其中 $K, T_0$ 为环节参数。

{{< r2figure
    r2path="control-theory-review/03/lu_figure_3-6.png"
    caption="常见二阶系统结构图（58页 表3-6）"
    align=center
>}}

系统的闭环传递函数为

$$\large \varPhi (s) = \frac{K}{T_0 s^2 + s + K} \ ,$$

化为标准形式

$$
\begin{alignedat}{2}
\varPhi(s) &= \frac{\omega_\mathrm{n}^2}{s^2 + 2\zeta\omega s + \omega_\mathrm{n}^2} &\qquad& (\text{首1型})
\\[1em]
\varPhi(s) &= \frac{K}{T_0 s^2 + s + K} &\qquad& (\text{尾1型})
\end{alignedat}
$$

式中 $\displaystyle T = \sqrt{\frac{T_0}{K}}, \quad \omega_\mathrm{n} = \frac{1}{T} = \sqrt{\frac{K}{T_0}}, \quad \zeta = \frac{1}{2} \sqrt{\frac{1}{K T_0}} \quad .$

以上的 $\zeta, \omega_\mathrm{n}$ 分别称为系统的阻尼比和无阻尼自然频率，是二阶系统重要的特征参数。二阶系统的首1标准型传递函数常用于时域分析中，频域分析时则常用尾1标准型。

二阶系统闭环特征方程为

$$D(s) = s^2 + 2\zeta\omega s + \omega_\mathrm{n}^2 = 0 \ ,$$

对应的特征根为

$$\lambda_{1, 2} = -\zeta \omega_\mathrm{n} \pm \mathrm{j} \omega_\mathrm{n} \sqrt{1 - \zeta^2} \qquad (0 < \zeta < 1),$$

上式为欠阻尼情况的系统特征根。当改变二阶系统的阻尼比 $\zeta$ 时，特征根在 $s$ 平面上的分布随之改变，系统特性会有比较明显的变化。我们将在后面讨论这一点。

### 欠阻尼二阶系统的动态过程分析

一般教材都会先介绍过阻尼和临界阻尼的二阶系统分析，再来用长篇幅介绍欠阻尼的情况；但作为复习，我们先研究欠阻尼二阶系统是如何分析的，再考虑临界阻尼和过阻尼这样的特殊情况。

先来看一张重要的图：

{{< r2figure
    r2path="control-theory-review/03/lu_figure_3-10.png"
    caption="欠阻尼二阶系统的极点表示（62页 图3-10）"
    align=center
>}}

我们刚刚分析了二阶系统的闭环特征方程并解出了一对共轭的特征根。参照上图可以看到，欠阻尼二阶系统的特征根总是在 $s$ 平面的虚轴左侧（即具有负实部），其中

- $\sigma = \zeta \omega_\mathrm{n}$: 衰减系数，决定了系统单位阶跃响应收敛的速度；

- $\omega_d = \omega_\mathrm{n} \sqrt{1 - \zeta^2}$: 有阻尼振荡频率；

- $\omega_\mathrm{n}$: 无阻尼振荡频率，是系统的固有特性参数，若令 $\zeta = 0$ 并保持其他参数不变，系统将会做频率为 $\omega_\mathrm{n}$ 的等幅正弦振荡。

- $\beta$: 阻尼角，在后续的根轨迹分析/频率域分析中有重要作用。

#### 欠阻尼二阶系统的单位阶跃响应

对于欠阻尼二阶系统，其单位阶跃响应的拉普拉斯变换为

$$
\begin{aligned}
C(s)
&= \varPhi(s) R(s)
= \frac{\omega_\mathrm{n}^2}{s^2 + 2 \zeta \omega_\mathrm{n} s + \omega_\mathrm{n}^2} \frac{1}{s}
= \frac{1}{s} - \frac{s + 2 \zeta \omega_\mathrm{n}}{(s + \zeta \omega_\mathrm{n})^2 + (1 - \zeta^2) \omega_\mathrm{n}^2}
\newline
&= \frac{1}{s} - \frac{s + \zeta \omega_\mathrm{n}}{(s + \zeta \omega_\mathrm{n})^2 + (1 - \zeta^2) \omega_\mathrm{n}^2} - \frac{\zeta}{\sqrt{1 - \zeta^2}} \cdot \frac{\sqrt{1 - \zeta^2} \omega_\mathrm{n}}{(s + \zeta \omega_\mathrm{n})^2 + (1 - \zeta^2) \omega_\mathrm{n}^2}
\end{aligned}
\ ,
$$

反变换得到时域的阶跃响应[^3]

$$
\begin{aligned}
h(t)
&= 1 - \mathrm{e}^{-\zeta \omega_\mathrm{n} t} \cos(\sqrt{1 - \zeta^2} \omega_\mathrm{n} t) - \frac{\zeta}{\sqrt{1 - \zeta^2}} \mathrm{e}^{-\zeta \omega_\mathrm{n} t} \sin(\sqrt{1 - \zeta^2} \omega_\mathrm{n} t)
\newline
&= 1 - \frac{\mathrm{e}^{-\zeta \omega_\mathrm{n} t}}{\sqrt{1 - \zeta^2}} \left [ \sqrt{1 - \zeta^2} \cos(\sqrt{1 - \zeta^2} \omega_\mathrm{n} t) - \frac{\zeta}{\sqrt{1 - \zeta^2}} + \zeta \sin(\sqrt{1 - \zeta^2} \omega_\mathrm{n} t) \right ]
\newline
&= 1 - \frac{\mathrm{e}^{-\zeta \omega_\mathrm{n} t}}{\sqrt{1 - \zeta^2}} \sin \left ( \sqrt{1 - \zeta^2} \omega_\mathrm{n} t + \arctan \frac{\sqrt{1 - \zeta^2}}{\zeta} \right )
\newline
&= 1 - \frac{\mathrm{e}^{-t \cot \beta}}{\sin \beta} \sin \left ( \omega_d t + \beta \right ) 
\ ,
\end{aligned}
$$

当改变阻尼比 $\zeta$ 时，时域响应也随之变化如下：

{{< r2figure
    r2path="control-theory-review/03/lu_figure_3-11.png"
    caption="典型二阶系统的单位阶跃响应（63页 图3-11）"
    align=center
>}}

系统单位阶跃响应同时取决于阻尼比 $\zeta$ 和无阻尼振荡频率 $\omega_\mathrm{n} \ ,$ 其响应曲线位于两条包络线 $1 \pm \frac{1}{\sqrt{1 - \zeta^2}} \mathrm{e}^{-\zeta \omega_\mathrm{n} t}$ 之间，如下图所示。包络线收敛速率取决于 $\zeta \omega_\mathrm{n}$ （特征根实部之模），响应的阻尼振荡频率取决于 $\sqrt{1 - \zeta^2} \omega_\mathrm{n}$ （特征根虚部）。响应的初始值 $h(0) = 0 \ ,$ 初始斜率 $h^\prime (0) = 0 \ ,$ 终值 $h(\infin) = 1 \ .$

{{< r2figure
    r2path="control-theory-review/03/lu_figure_3-11.png"
    caption="欠阻尼二阶系统单位阶跃响应及包络线（63页 图3-12）"
    align=center
>}}

二阶系统的单位脉冲响应为

$$
\begin{aligned}
k(t) &= h^\prime (t) = \mathscr{L}^{-1} \left [ \frac{\omega_\mathrm{n}}{\sqrt{1 - \zeta^2}} \cdot \frac{\sqrt{1 - \zeta^2} \omega_\mathrm{n}}{(s + \zeta \omega_\mathrm{n})^2 + (1 - \zeta^2) \omega_\mathrm{n}^2} \right]
\newline
&= \frac{\omega_\mathrm{n}}{\sqrt{1 - \zeta^2}} \mathrm{e}^{-\zeta \omega_\mathrm{n} t} \sin(\sqrt{1 - \zeta^2} \omega_\mathrm{n} t)
\newline
&= \frac{\omega_\mathrm{n}}{\sin \beta} \mathrm{e}^{-t \cot \beta} \sin(\omega_d t)
\ .
\end{aligned}
$$

#### 欠阻尼二阶系统的动态性能指标

##### 峰值时间 $t_\mathrm{p}$

令 $h^\prime (t) = k(t) = 0$ 可得方程

$$\sin (\sqrt{1 - \zeta^2} \omega_\mathrm{n} t) = 0 \ ,$$

即

$$\sqrt{1 - \zeta^2} \omega_\mathrm{n} t = 0, \pi, 2 \pi, 3 \pi, ...$$

峰值时间指的是第一次达到峰值使 $h^\prime (t) = 0$ 的时刻，故取上式右边为 $\pi$ 可得欠阻尼二阶系统的峰值时间计算公式

$$t_\mathrm{p} = \frac{\pi}{\sqrt{1 - \zeta^2} \omega_\mathrm{n}} = \frac{\pi}{\omega_\mathrm{d}}$$


[^1]: 可以看回 [上篇文章的结尾]({{<siteurl>}}articles/2025/03/control-theory-review-02/#%E9%97%AD%E7%8E%AF%E4%BC%A0%E9%80%92%E5%87%BD%E6%95%B0%E4%B8%8E%E7%B3%BB%E7%BB%9F%E7%9A%84%E7%89%B9%E5%BE%81%E6%A0%B9) 回顾一下系统特征根的意义。

[^2]: 当 $T$ 越小时，$e^{- \frac{t_\mathrm{s}}{T}}$ 的指数部分的绝对值就会越大，对应使 $e^{- \frac{t_\mathrm{s}}{T}}$ 收敛速度更快。

[^3]: 我个人比较喜欢把 $\mathrm{e}^{-\zeta \omega_\mathrm{n} t}$ 记作 $\mathrm{e}^{-t \cot \beta} \ ,$ $\sqrt{1 - \zeta^2} \omega_\mathrm{n}$ 记作 $\omega_d \ ,$ 因为比较容易记住 :D
