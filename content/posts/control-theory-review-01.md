---
title: "自动控制原理 - 01 - 一般概念，数学模型与拉普拉斯变换"
date: 2025-03-17T16:43:01+08:00
tags: ["2025-03", "学习笔记", "控制理论", "自动控制原理", "复习", "拉普拉斯变换"]
categories: ["自动控制原理"]
toc: true
draft: true
---

打算开个系列内容记录一下学习自动控制原理时我的一些理解和体会，也顺便在这个过程中再复习一遍经典控制理论。学习进程主要跟着西北工业大学卢京潮老师的录播课程[^1]进行的（相信大部分同学也是看这个课程学习的自控），使用的教材是卢老师主编的《自动控制原理》，后面的文章也会基本按照卢老师教材中的目录安排；当然还有一些其他的图书值得参考，附列在这篇文章的最后。可以自行查阅。

{{< figure
    src="https://z4a.net/images/2025/03/17/cdc3a92b7d69ceb1db93a769d9cbaa1e.md.png"
    caption="卢京潮. 自动控制原理[M]. 第一版. 清华大学出版社, 2013"
    align=center
>}}

## 引言

什么是自动控制？我想从《三体》第三册中摘抄一段：

> 在接下来的IDC委员会会议上，主席拿来了一把他安排人专门制造的伞，与故事中空灵画师送给公主的保护伞一样，是黑色的，有八根伞骨，每根的末端都有一只小石球。真正意义上的伞早就从现代生活中消失了，现代人遮雨使用一种叫避雨器的东西，如小手电筒般大小，向上吹出气流把雨吹开。人们当然知道伞这东西曾经存在，也在影视中见过，但很少有人见过实物。大家好奇地争相摆弄这东西，发现它可以像故事中描写的那样在旋转中借石球的离心力张开，在旋转速度过快或过慢时也能发出相应的声音报警。大家的第一感觉是这样旋转着打伞是件很累的事，公主的奶妈居然能这样打一天伞，很让人佩服。  
> AA也拿过伞旋转着打开，她的手劲比较小，转动的伞面很快垂下来，警示转速过慢的鸟叫声出现了。  
> 从主席把伞第一次打开时，程心就目不转睛地盯着它，现在，她突然指着AA喊道：“别停下！”  
> AA加快了伞的转速，鸟叫声消失了。  
> “再转快些。”程心盯着伞说。  
> AA使尽力气转伞，警示转速过快的风铃声出现了；然后程心又让她转慢些，直到再次出现鸟叫声，就这样反复了几次。  
> “这不是伞！”程心指着旋转中的伞说，“我知道它是什么！”  
> 旁边的毕云峰点点头，“我也知道了。”然后他转向在场的第三个公元人曹彬，“这是一种只有我们三个人才能想到的东西。”  
> “是的。”曹彬看着伞兴奋地说，“即使在我们那个时代，这东西也很陌生了。”  
> 其余的与会者有的看着这三个活着的古人，有的看着伞，全都莫名其妙，但也都兴奋地期待着。  
> “蒸汽机离心调速器。”程心说。  
> “那是什么，一种控制电路？”有人问。  
> 毕云峰摇摇头，“发明那东西的时候还没有电。”  
> 曹彬开始解释：“那是18世纪出现的东西，一种用于调节蒸汽机转速的装置。它主要由两根或四根头部带金属球的悬杆和一根带套筒的转轴组成，就像这把伞，只是伞骨数量要少些。这个装置的转轴由蒸汽机带动旋转，当蒸汽机转速过快时，铁球由于离心力抬起悬杆，带动套筒上升，把与套筒相连的蒸汽门关小，降低蒸汽机转速；蒸汽机转速过低时，离心力的减小使悬臂内合，像伞合上一样，推动套筒下滑，开大蒸汽门增加转速……这是最早的工业自动控制系统。”  

这一段情节安排得非常巧妙：在场的三个公元人都有工科专业背景，学习过自动控制原理这门课程。在他们的时代（也就是我们的时代），这门课程的第一节课还会举出『蒸汽机离心调速器』的例子以说明课程的目标；三个世纪后的人们已经不知道这是个什么东西了。小说中的这个例子很好地说明了自动控制理论的研究目标：对复杂现实系统进行建模并按照规律控制系统中的某个量，使其稳定在某个数值附近，或者按照我们希望的规律变化。

虽然小说中提到的蒸汽机离心调速器在早期的蒸汽机中得到了大量应用，但人们很快就发现把它直接做出来后装上蒸汽机经常会产生振荡现象，即蒸汽机的转速会时快时慢地周期变化。可以想象一下传统不锈钢压力锅盖上面那个泄压阀：它的重量经过设计，当锅内的压力达到一定值时蒸汽就会把这个不锈钢锭顶起来，并顺着钢锭上的开孔放出让它旋转。但生活经验告诉我们它的旋转是震荡的而非匀速旋转，并进一步使得放气声也有规律地变化。

这种振荡促使着人们继续研究控制理论，想找出设计稳定控制系统的普遍规律。但现实中的系统都是以微分方程，甚至高阶偏微分方程描述的，想直接求解这样的一个方程非常困难；好在后来控制科学家提出了不必求解微分方程也能判断系统稳定性的一般方法（劳斯-赫尔维茨稳定判据，与拉普拉斯变换后得到的系统复变方程中零极点的位置有关；奈奎斯特稳定判据，与经典控制理论的频率域方法有关），解决了稳定性的判断问题。

借助于奈奎斯特提出的频率域（复变函数）方法与之前控制科学家们长期研究的时域（微分方程）方法，经典控制理论的基座已经建立起来了。伊文斯（~~消灭人类暴政，世界属于三体！~~）提出的根轨迹法某种程度上可以看作是二者的结合，提供了一个在计算机尚不发达的年代简单且快速地对控制系统进行大致定量分析的方法；再加上对线性离散系统分析时使用的拉普拉斯变换之推广——$z$变换，与对简单非线性系统分析时使用的相平面法和描述函数法，一般大学中开设的『自动控制原理』课程大概就讲到这里了。再往后面的最优控制、状态空间方法，以及鲁棒控制等更加现代的控制理论每个拎出来都能单独作为一门课程，想在这寥寥几篇文章中讲明白实在不太可能（而且我自己也弄得不是很明白，到时估计得当新学一样从头来过）。

## 自动控制的一般概念

上面对这门课程将要讨论的内容大致回顾了一遍，接下来让我们从头看看应该从哪里开始研究控制系统。

### 开环控制和闭环控制

开环控制系统指的是系统的控制作用不受输出影响的控制系统。一个典型的例子是保温电热水壶：不考虑其他因素，则通过电位器旋钮调节电阻大小会改变回路中的电流，从而改变电热丝的热功率，烧热壶中的水。但这个过程没有反馈的参与，水温最后稳定的范围是取决于环境温度、水壶壁的导热效率等因素的。

闭环控制系统则将系统输出量的信息反馈到输入端，得到输入和输出的差值（称作偏差）；利用这个偏差信号对系统进行调节，最终达到减小或消除偏差的目的即为闭环负反馈控制。仍以保温壶举例：我们在壶中放入一个温度传感器并设定一个预期值，根据读取得到的数值调节电位器的阻值，如果水温偏低则调小电位器使热功率上升，加热壶中水，反之则调大电位器使热功率下降，水温降低。

为何最小化偏差就能使系统稳定？我们先来看课本上的一幅图：

{{< figure
    src="https://z4a.net/images/2025/03/20/593c420aa0c329beee4ef16abe3a0667.png"
    caption="典型的反馈控制系统方框图（教材P8 图1.5）"
    align=center
>}}

对于刚刚提到保温壶的例子，其输入量就是我们设定的预期水温，被控对象是水壶的电热丝，输出量是测得水的温度，而测量元件即为刚提到的温度传感器。最小化偏差在这里非常好理解：当输入温度与测得水温有差异时这个信号会输入到控制系统，调节电热丝进行“多退少补”；最后达到稳态时壶中的水温会与设定值完全相同，系统也就停止工作直到热量散失测得水温下降，再次启动调节。这是一个最简单的使用负反馈控制原理的闭环控制系统例子。

上图中各个元件准确的定义可以参考教材P8。

### 控制系统的分类

这一段基本是书上的内容，参见教材1.6 自动控制系统的分类。

#### 按给定信号形式不同

1. **恒值控制系统**（也称为定值系统或调节系统）：控制输入是恒定值,要求被控量保持给定值不变。例如前面提到的液位控制系统,直流电动机调速系统等。
2. **随动控制系统**（也称为伺服系统）：控制输入是变化规律未知的时间函数,系统的任务是使被控量按同样的规律变化并与输入信号的误差保持在规定范围内。例如函数记录仪,自动火炮系统和飞机——自动驾驶仪系统等。
3. **程序控制系统**：的给定信号按预先编制的程序确定，要求被控量按相应的规律随控制信号变化。机械加工中的数控机床就是典型的例子。

#### 按系统参数是否随时间变化

如果**控制系统的参数在系统运行过程中不随时间变化**，则称之为定常系统或者时不变系统，否则称其为时变系统。

控制系统的参数是什么？从物理意义上看就是系统中各个元件的固有参数，比如电容的容量、弹簧的弹性系数、电机的转矩等。时不变的一个反例就是白炽灯（老式的钨丝灯泡）的电阻。对于一个冷启动的白炽灯，其电阻会随着时间推移灯泡温度上升而缓慢下降，因而它不是一个时不变元件。虽然现实世界中不存在真正的时不变元件，但在某个范围内它们的变化微乎其微，这时我们仍然可以近似地将其作为定常系统来处理。

#### 按系统是否满足叠加原理

什么是叠加原理？我们用$F(x)$来表示一个系统的输出，其中$F$描述了系统本身，$x$是系统的输入。如果对于多个输入线性地叠加在一起形成一个总和的输入$x=x_1+x_2+...$，有

$$F(x_1+x_2+...)=F(x_1)+F(x_2)+... \ ,$$

即总和的输入产生的输出会等于每个输入单独产生的输出之代数和，我们称满足这个规律的系统符合叠加原理（具有叠加性）。

齐次性的概念可以参考[齐次性到底描述了什么? - lyounger的回答 - 知乎](https://www.zhihu.com/question/25552461/answer/34618178)。或者我们回到高数课堂，看看微分方程那一章的内容：

{{< figure
    src="https://z4a.net/images/2025/03/21/943e77578b810d6f75056d5736001277.png"
    caption="一阶齐次线性微分方程 同济版高等数学教材第八版 P307"
    align=center
>}}

简单地说，当一个微分方程满足从最高阶到零阶之间都不缺项，即形如

$$a_n\frac{\mathrm{d}^n y}{\mathrm{d} x^n} + a_{n-1}\frac{\mathrm{d}^{n-1} y}{\mathrm{d} x^{n-1}} + a_{n-2}\frac{\mathrm{d}^{n-2} y}{\mathrm{d} x^{n-2}} + ... + a_1\frac{\mathrm{d} y}{\mathrm{d} x} + a_0 y = 0 \ ,$$

则称其为常系数齐次线性微分方程，其中$a_i$为常数。

线性系统总是能用这样一个常系数线性微分方程来描述。它具有齐次性和叠加性，系统的稳定性与初始状态及外作用无关；这部分基础知识的介绍可以参见[卢老师的课程](https://www.bilibili.com/video/BV1F34y1h7so?t=606.9&p=5)。这也是我们能够用拉普拉斯变换把系统微分方程转化为代数方程求解的基础。接下来要研究的大部分都是线性时不变系统。

如果控制系统中含有一个或一个以上非线性元件，这样的系统就属于非线性控制系统。非线性系统不满足叠加原理，系统响应与初始状态和外作用都有关。虽然实际物理系统都具有某种程度的非线性，但在一定范围内通过合理简化（可以理解为泰勒展开取低阶项），大量物理系统都可以足够准确地用线性系统来描述。

#### 按系统中的信号是连续的还是离散的

如果系统中各部分的信号都是连续函数形式的模拟量，则这样的系统就称为连续系统，接下来要讨论的大部分系统都属于连续系统。如果系统中有一处或几处的信号是离散信号（脉冲序列或数码），则这样的系统就称为离散系统（包括采样系统和数字系统）。计算机控制系统就是离散控制系统的典型例子。

#### 按系统输入/输出信号的数量

按照系统输入信号和输出信号的数目，可将系统分为单输入-单输出（SISO）系统和多输入-多输出（MIMO）系统。

单输入-单输出系统通常称为单变量系统，这种系统只有一个输入（不包括扰动输入）和一个输出。多输入-多输出系统通常称为多变量系统，有多个输入或多个输出。单变量系统可以视为多变量系统的特例。

在描述多输入-多输出系统时，我们通常会使用状态空间方程，这部分知识可以参考现代控制理论中的相关内容。接下来要讨论的系统基本都是SISO系统，涉及到MIMO的情形时我们会把每个输入和输出拆分出来研究它们各自的影响。由于线性时不变系统满足叠加原理，因此对于某个特定的输出端口，所有输入经过系统后产生的输出叠加在一起即为这个端口最终会得到的输出。

### 控制系统的性能目标

对于一个控制系统，我们一般从稳、准、快三个方面评价其性能。

#### 稳定性

稳定性指的是系统重新恢复平衡状态的能力。任何一个能够正常工作的控制系统，首先必须是稳定的。稳定是对自动控制系统的最基本要求。

由于闭环控制系统有反馈作用，控制过程有可能出现振荡或发散。回到之前在引言中提到的压力锅泄压阀的例子：它泄压时的振动正是来源于压力锅-泄压阀系统自身的不稳定性。更具体地，这个系统可以被简化为带有阻尼与弹性恢复力的二阶系统。在临界状态时蒸汽流速会随阀门开度增加而显著增大，使系统进入负阻尼状态；但是泄压阀中限位器（钢锭内卡住泄压孔的弹片，防止压力过大把泄压阀吹走）的存在引入了非线性因素，最终二者共同作用形成了一个稳定的振荡。

{{< figure
    src="https://z4a.net/images/2025/03/21/c0fb18b1c9116c928e5d010ec56b3559.png"
    caption="不同系统的单位阶跃响应过程 《自动控制原理》 卢京潮 P15"
    align=center
>}}

不稳定的系统无法使用。

#### 准确性

准确性是对系统稳态（静态）性能的要求。对一个稳定的系统而言，当过渡过程结束后，系统输出量的实际值与期望值之差称为稳态误差，它是衡量系统控制精度的重要指标。稳态误差越小，表示系统的准确性越好，控制精度越高。

#### 快速性

快速性是对系统过渡过程（动态）性能的要求。描述系统动态性能可以用平稳性和快速性加以衡量。平稳是指系统由初始状态过渡到新的平衡状态时，具有较小的过调（超调量，指第一次调节时系统输出值超过稳态值的多少）和振荡性；快速是指系统过渡到新的平衡状态所需要的调节时间较短。动态性能是衡量系统质量高低的重要指标。

## 控制系统的数学模型

### 从元部件特性建立系统线性微分方程

在研究元部件特性时，我们主要研究的是与这个元部件相关的物理量和时间的函数关系。自动控制原理中主要研究的是电路与机械两种系统的分析和校正。

#### 电路分析

可以先看看这个视频[【动态系统的建模与分析】2_电路系统建模_基尔霍夫定律](https://www.bilibili.com/video/BV13t411t7aR/)来了解在自动控制原理课程中典型的电路分析过程；如果要深入学习电路分析，可以参考电子工业出版社 『国外电子与通信教材系列』的《工程电路分析》[^2]，虽然厚但是讲的非常清晰透彻。

先来看裴润老师教材中总结的常用电路元件的微分方程描述：

{{< figure
    src="https://z4a.net/images/2025/03/21/0725f10ab4f196c080c38fcc3b4ffac1.png"
    caption="电阻、电容、电感两端电压电流的关系 《自动控制原理（上册）》 裴润 P12"
    align=center
>}}

对于电路，在分析时还需要用到基尔霍夫电路定律及复阻抗分析方法：

> **所有进入某节点的电流的总和等于所有离开这节点的电流的总和。**

> **沿着闭合回路所有器件两端的电势差（电压）的代数和等于零。**

|元件|阻抗|物理意义|
|:-:|:-:|:-:|
|电阻|$Z_R = R$|纯实数，不随频率变化|
|电容|$Z_C = \frac{1}{j\omega C} = -\frac{j}{\omega C}$|电容的阻抗随频率增加而减小|
|电感|$Z_L = j\omega L$|电感的阻抗随频率增加而增加|

如果将上表中的$j\omega$换成$s$，我们就得到了各个元件参数的复频域（$s$域）表示。实际上在学习信号与系统课程时会有一个小节专门讲解使用拉普拉斯变换进行电路分析：这种方法的本质是利用拉普拉斯变换将描述电路系统的常微分方程变换为$s$域上的代数方程，求解后再使用拉普拉斯反变换回时域得到结果。

我们可以用网孔法/节点电压法结合复阻抗来计算电路中各个元件两端的电压和流经它的电流。

##### 网孔法

1. **定义网孔电流**：

- **网孔**：指不包含其他回路的最小封闭回路。
- **网孔电流**：在每个网孔中，假设一个顺时针或逆时针的电流，并假设所有网孔电流都互不相关（实际上可能有部分重叠）。

2. **应用基尔霍夫电压定律（KVL）**：对于每个网孔，列出 KVL 方程：

$$\sum V = 0, $$

即网孔中所有元件的电压降之和等于零。

3. **分析共用电阻**：如果一个电阻 **仅属于一个网孔**，则其电压降等于该网孔电流乘以电阻值（欧姆定律）；若其 **被两个网孔共用**，则其电压降等于该电阻的电流（即两个网孔电流之差）乘以电阻值。

4. **求解线性方程组**：列出所有独立的KVL方程后，解方程组，求出所有网孔电流。

##### 节点电压法

1. **选择参考节点**：选择一个电路中的参考节点（接地节点）并将参考节点的电压定义为**0V**，通常选择电压最低或者连接支路最多的点。

2. **标注节点电压**：对非参考节点（其余所有节点）计算其相对于参考节点的电位差；

3. **应用基尔霍夫电流定律（KCL）**：对每个未知节点列出 KCL 方程：

$$\sum_{k=1}^{n}{i_k} = 0, $$

即进入节点的电流 = 离开节点的电流。之后使用欧姆定律（$I = \frac{U}{R}$， 其中$R$指的是复阻抗），将电流用电压表示出来。

4. **求解方程**：将每个节点的电流-电压关系综合形成线性方程组，解出节点电压；如果需要，可以用节点电压求解电路中的电流。

#### 机械系统分析

同样地，裴润老师已经为我们总结了常见的机械系统要素：

{{< figure
    src="https://z4a.net/images/2025/03/21/dce52a0aad8547161ccd97787bbdc2a1.png"
    caption="机械系统中的基本要素 《自动控制原理（上册）》 裴润 P13"
    align=center
>}}

让我们来明确一下机械系统中运动物体所遵循的物理定律。

- 做直线运动的物体要遵循牛顿第二定律，即

$$\sum F = m \frac{\mathrm{d}^2 x}{\mathrm{d} t^2}, $$

式中$F$为物体所受到的力，$m$为物体的质量，$x$为物体在此过程中的线位移，$t$为物体运动过程所经历的时间。

回顾一下高中物理知识，我们知道物体受到的力是满足矢量分解/叠加原理的。也就是说，我们在分析受力时可以将物体受到的所有力都计算出其在运动轴向上的分力，并用牛顿第二定律$F= m a = m \ddot{x}$将二者联系起来。

- 转动的物体要遵循牛顿转动定律，即

$$\sum T = J \frac{\mathrm{d}^2 \theta}{\mathrm{d} t^2}, $$

式中$T$为物体所受到的力矩，$J$为物体的转动惯量[^3]，$\theta$为物体在此过程中的角位移，$t$为物体运动过程所经历的时间。

我们可以注意到转动力矩的表达式十分类似于牛顿第二定律。转动惯量是物体的一个固有属性，与转动轴和质量有关：

&emsp;&emsp;对于质点， 其转动惯量$J = m r^2$，其中$m$是其质量，$r$是质点和旋转轴的垂直距离。

&emsp;&emsp;对于刚体，可将其视作无限个质点之和并用积分计算其转动惯量：$J = \int{\rho r^{2}} \mathrm{d}V$，其中$\rho$是物体的密度，$\mathrm{d}V$是体积微元。

- 实际运动中的物体总是会受到摩擦力的阻碍：做线运动的物体要受到摩擦力$F_c$的作用，可以表示为

$$F_c = F_B + F_f = f \frac{\mathrm{d} x}{\mathrm{d} t} + F_f, $$

式中$F_B$为粘性摩擦力，与运动的速度成正比，$f$是粘性摩擦系数；$F_f$为恒值摩擦力，又称作库仑摩擦力。

- 对于转动的物体，摩擦力体现为摩擦力矩$T_c$

$$T_c = T_B + T_f = K_c \frac{\mathrm{d} \theta}{\mathrm{d} t} + T_f, $$

类似地，式中$T_B$是粘性摩擦力矩，与转动的叫速度成正比，粘性阻尼系数$K_c$是其比例系数；$T_f$为恒值摩擦力矩。

对机械系统中的每个部件分析其各物理量之间的关系后加总，即可得到系统输入与输出之间的关系。

### 傅里叶级数与傅里叶变换

在学习拉普拉斯变换之前，有必要先回顾一下傅里叶变换。

#### 连续时间周期信号的傅里叶级数展开

连续时间傅里叶级数是一种将任意**连续时间周期信号**表示成无穷多个周期函数的线性组合，可以表示为复指数形式和三角函数形式如下：

$$x(t) = \sum_{k = - \infty}^{+ \infty} a_k e^{j \omega_0 t} = \sum_{k = 0}^{+ \infty} B_k \cos(j \omega_0 t) + C_k \sin(j \omega_0 t). $$

其中$\omega_0 = \frac{2 \pi}{T_0}$是输入信号的角频率，$T_0$是输入信号的周期。$k = 0$的项被称为直流分量，$k = 1$为基波分量，$k = N$的分量被称为$N$次谐波分量。

在复指数形式中，系数$a_k$的值可以利用三角函数的正交性计算：

$$a_k = \frac{1}{T_0} \int_{0}^{T_0} x(t) e^{-j \omega_0 t} \mathrm{d} t$$

这部分的证明可以参见胡浩基老师的课程：[函数的正交分解](https://www.bilibili.com/video/av343424609?p=17)。

#### 非周期信号的傅里叶变换导出

对于**非周期信号**，我们可以将其视作周期为无限大的周期信号来应用傅里叶级数展开。这时x(t)的周期$T$满足$T \to + \infty$，则对应有$\frac{2 \pi}{T} = \omega_0 \to 0$。此时的$a_k$的计算转变为

$$a_k = \frac{1}{T_0} \int_{\infty}^{+ \infty} x(t) e^{-j \omega_0 t} \mathrm{d} t , $$

我们定义

$$X(j \omega) = \int_{\infty}^{+ \infty} x(t) e^{-j \omega_0 t} \mathrm{d} t , $$

则$a_k$可以表示为

$$a_k = \frac{1}{T_0} X(j \omega_0) . $$

将其代回到$x(t)$的傅里叶级数展开式，有

$$x(t) = \sum_{k = - \infty}^{+ \infty} \frac{1}{T_0} X(j \omega_0) e^{j \omega_0 t} = \frac{1}{2 \pi} \sum_{k = - \infty}^{+ \infty} \omega_0 X(j \omega_0) e^{j \omega_0 t} . $$

当$\omega_0 \to 0$时，根据黎曼积分的定义将$\omega$视作积分微元，则上式可以表示为

$$x(t) = \frac{1}{2 \pi} \int_{- \infty}^{+ \infty} X(j \omega_0) e^{j \omega_0 t} \mathrm{d} \omega$$

于是我们得到了**傅里叶变换对**

$$X(j \omega) = \int_{\infty}^{+ \infty} x(t) e^{-j \omega t} \mathrm{d} t , $$
$$x(t) = \frac{1}{2 \pi} \int_{- \infty}^{+ \infty} X(j \omega) e^{j \omega t} \mathrm{d} \omega . $$

这里的$X(j \omega)$表示的是将时域信号$x(t)$展开为三角函数信号之叠加时，对应频率为$\omega$的复指数信号$e^{j \omega t}$的“复幅度”。当然，上面的过程只极简略地说明了$x(t)$与$X(j \omega)$是如何互相转换的，它们之间详细的物理意义请参考胡浩基老师的课程。

#### 傅里叶变换的收敛条件

并不是所有的$x(t)$都能够通过傅里叶变换得到其频谱。具体来说，输入的时域函数$x(t)$必须满足**狄利克雷条件**，其傅里叶变换才是收敛的：

1. $x(t)$绝对可积，即$\int_{- \infty}^{+ \infty} | x(t) | \mathrm{d} t < + \infty$；

2. 在任意有限区间内，$x(t)$只有有限个最大值和最小值；

3. 在任意有限区间内，$x(t)$只有有限个不连续点，并且在每个不连续点上信号都是有限值。

#### 使用傅里叶变换解微分方程

为什么说傅里叶变换和微分方程的求解有关？让我们回到高数课堂，回忆一下二阶线性常系数**非齐次**微分方程的求解。一般来说，二阶常系数非齐次微分方程可以写成如下形式：

$$y^{\prime \prime} + p y^\prime + q y = f(x) , $$

其中$p$，$q$是常数。

一般来说，我们特别关心$f(x)$的两种形式：
$$f(x) = e^{\lambda x} P_m (x) , $$

$$f(x) = e^{\lambda x} [P_l (x) \cos(\omega x) + Q_n (x) \sin(\omega x)] , $$

其中$P_m (x)$、$P_l (x)$与$Q_n (x)$分别是$m$、$l$、$n$次的多项式；这些特定$f(x)$形式对应的非齐次特解是有公式可循的，并且与齐次方程$y^{\prime \prime} + p y^\prime + q y = 0$的特征根强相关，即使在高于二阶的情形下也是如此。

可惜的是现实中需要解决的实际物理问题大部分都没有这么好的形式。但我们自然会想，如果能通过某种数学变换将任意函数都转化成上面两种特殊形式，那就很容易可以求解这样的微分方程了。傅里叶所做的工作正式如此：他提出的傅里叶变换能将任意的函数变换成三角函数形式（前提是满足狄利克雷条件），那么求解这样的微分方程就变成了简单的分解$f(x)$成谐波组合形式 $\to$ 求解三角函数形式的非齐次方程 $\to$ 把所有分解结果对应的非齐次方程的解相加，当分解后保留的谐波次数足够多（类似于泰勒展开保留高次项），最终就能以足够精度得到原始微分方程的解了。

### 拉普拉斯变换

狄利克雷条件限制了傅里叶变换所适用的对象，具体来说是要求$x(t)$的积分$\int_{- \infty}^{+ \infty} | x(t) | \mathrm{d} t$必须是收敛的。但这样好的性质并不是很常见，不如说现实世界中碰到的$x(t)$绝大部分都会发散；有没有办法绕过这个限制呢？

### 线性时不变系统的微分方程求解

前面已经提到线性时不变系统可以用一个常系数线性微分方程描述。在高等数学中我们已经学习过用于求解高阶微分方程的特征方程法，即假设解的形式为$y=e^{\lambda t}$并代入原方程进行求解；但这种方式求解起来非常繁琐，并且对于不同的输入每次都要解对应的方程。

### 运动的模态

线性微分方程的解由齐次方程的通解和输入信号对应的特解组成。通解反映的是系统的自由响应（即零输入响应，系统在没有外界输入的情况下自发产生的输出）。如果微分方程的特征根是单实根$\lambda_1, \lambda_2, ...$，则把函数$e^{\lambda_1 t}, e^{\lambda_2 t}, ...$称为该微分方程所描述运动的模态,也叫振型。

如果特征根中有多重根$\lambda$，则模态是具有$t e^{\lambda t}, t^2 e^{\lambda t}, ...$形式的函数。

如果特征根中有共轭复根$\lambda = \sigma + j \omega$，则其共轭复模态$e^{\sigma + j \omega}$、$e^{\sigma - j \omega}$可写成实函数模态$e^{\sigma t} \sin(\omega t)$、$e^{\sigma t} \cos(\omega t)$。

每一种模态可以看成是线性系统自由响应中最基本的运动形态，线性系统的自由响应就是其相应模态的线性组合。通过模态分析，可以了解系统的运动特性。

## 参考书

以下列出的是我个人学习时使用过的一些自控教材，可供参考：

1. 胡寿松, 姜斌, 张绍杰. 自动控制原理[M]. 第八版. 科学出版社, 2024：国内最普遍流行的控制理论教材，涵盖了经典控制理论和现代控制理论两个方面。这本书很厚很大很全面，但对卢老师的视频课程不是十分适配，可以作为卢老师那本薄书（相对）的补充。

2. 裴润, 宋申民. 自动控制原理[M]. 第二版. 哈尔滨工业大学出版社, 2023：分上下册，上册为经典控制理论，下册为现代控制理论。第一版正文中的符号错误之多让人怀疑出版时有没有校对，好在我买的是二手书有前辈学长（学姐？）的笔记勘误看起来稍微好些，不知第二版有无修正；此外可能哈工大做控制的教授从苏联一脉相承，书中使用的闭环系统误差定义在输出端（期望输出-实际输出）而不是同美帝及其他国内教材一样定义在输入端（实际输入-实际输出），在后续列写闭环系统误差方程等章节也和其他教材有显著不同。

3. 吴麒, 王诗宓. 自动控制原理[M]. 第二版. 清华大学出版社, 2006：看网上说是清华本校用的教材，最大的特色是自控和现控穿插着讲。比如第一章先介绍传统时域微分方程——拉普拉斯变换——复域代数方程，紧接着马上开始介绍状态变量和状态空间方程。对于初学者不是很友好~~也有可能是我太笨了~~，但是频域分析的部分感觉写的不错。上下册加在一起比胡寿松那本还厚。另外这本书还用了数十页的篇幅讲了国内外控制理论的研究发展史，可以很清晰地看到这门学科的发展脉络。

4. 王天威. 控制之美（卷1）——控制理论从传递函数到状态空间[M]. 第二版. 清华大学出版社, 2022：B站一位做机器人控制的UP主[DR_CAN](https://space.bilibili.com/230105574)出的书，实际上初学者可能看不太明白，需要在学习完经典+现代控制理论，或至少学完经典控制理论后再作阅读会得到对控制理论的更统一的认识。他的视频讲的非常简明易懂，很适合新学或者复习。

以上图书都可以很容易地在[Z-Library](https://z-lib.fm/)上找到下载，当然如果有能力还是购买一本实体书支持一下各位老师。

## 后记

我个人觉得自动控制原理，或者说控制理论这个学科具有非常典型的工科特征：用数学方法解决物理问题。通过对现实问题抽象建模，在适当时候采取一些近似方法，得到的结论可以很好地吻合实际结果并指导现实生产生活，是这种方法的最好

[^1]:<https://www.bilibili.com/video/av811394492/>
[^2]:Hayt W H, Jr, EKemmerly J, etal. 工程电路分析[M]. 第八版. 电子工业出版社, 2012.
[^3]:<https://zh.wikipedia.org/wiki/%E8%BD%89%E5%8B%95%E6%85%A3%E9%87%8F>
