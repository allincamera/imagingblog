+++
categories = ["hardware", "software", "system"]
date = "2016-04-06T18:19:37+08:00"
tags = ["sensor", "lens", "algorithm"]
title = "解析力 （1 ）MTF SFR"

+++

## 基本概念
成像系统的解析力一直是摄像头最关键的指标之一。所有用户拿到一张照片的时候首先看到的是照片清楚不清楚，这里的清楚说得就是解析力。但是如何评价一个成像系统的解析力也是大家一直在探讨的问题。目前主流的办法主要有三种TV line检测，MTF检测，和SFR 检测。

### TV line
TV line主要用于主观测试，也有一些读取TV line的软件如HYRes。但是总体来说没有一个具体的标准。大多数公司是以人的读取为标准。不同人的读取，以及状态的不同都会导致读取值的不稳定。而且如ISO12233 chart 实际上我们读出的线对数只能代表读出位置的状况。尤其中心的TV line跨度很大，很难反映一个成像系统

### MTF

MTF是Modulation Transfer Function的英文简称，中文为调制传递函数。是指调制度随空间频率变化的函数称为调制度传递函数。个传递函数最开始是为了说明镜头的能力。在各个摄像头镜头中经常采用MTF描述镜头的MTF曲线，表明镜头的能力。这些曲线是通过理想的测试环境下尽量减少其它系统对镜头的解析力的衰减的情况下测试得出的。但是其实MTF也可以涵盖对整个成像系统的解析力评价。在这里咱们就不多讨论这个问题了，如果有兴趣可以开另外一篇文章讨论。

### SFR
SFR是 spatial frequency response (SFR) 主要是用于测量随着空间频率的线条增加对单一影像的所造成影响。简言之SFR就是MTF的另外一种测试方法。这种测试方法在很大程度上精简了测试流程。SFR的最终计算是希望得到MTF曲线。SFR的计算方法和MTF虽然不同但是在结果上是基本一致的 

## 测量方法

现在我们来看一下传统的MTF是怎么测量出来的，后面我们再针对SFR的原理和MTF的关系进行一些介绍。在以后的文章中我们在介绍一些MTF和SFR测试需要注意的问题。
从上面我们知道MTF是描述不同空间频率下的调制函数。那么什么是空间频率呢？通常，描述频率的单位是赫兹(Hz)，比如50Hz、100MHz之类的。但空间频率的表述习惯用“每毫米线对”。（LP/mm），就是每毫米的宽度内有多少线对。每两条线条之间的距离，以及线条本身的宽度之比是个定值，目前我国分辨率的标板规定，这个定为公因子是20√10≈1.122等比级数。一般MTF的计算离不开线对。下面这个图就是一张不同频率的线对测试图 ，可以看到图卡本身黑色和白色的对比是很清楚的。

![LP Demo1](https://raw.githubusercontent.com/allincamera/imgur/master/resolution_1/lp_demo1.gif)

实际拍摄的时候，就像上图一样频率越高（越细）的线对就越模糊。这就是我们实际拍摄场景中到一定小的纹理的就拍摄不清楚的原因。而MTF的计算就是计算线对间最亮和最暗线对的对比度。计算公式为

> MTF = (最大亮度 - 最小亮度) / (最大亮度 + 最小亮度)

所以MTF的计算不会出现大于1的情况。像下面的图表示的这样，当我们测试了很多不同频率下的MTF值。通过将这些值和空间频率进行一一的对照。通过这条曲线我们就能知道现在的成像系统在什么样的空间频率下的对比度如何。

![LP Demo2](https://raw.githubusercontent.com/allincamera/imgur/master/resolution_1/lp_demo2.jpg)

![LP Demo3](https://raw.githubusercontent.com/allincamera/imgur/master/resolution_1/lp_demo3.jpg)

SFR是怎么测试和计算的呢。首先SFR不需要拍摄不同的空间频率下的线对。它只需要一个黑白的斜边（刀口）即可换算出约略相等于所有空间频率 下的MTF。如何通过一个斜边计算出大家可以去看下ISO12233-2000那篇文档，里面说的很详细。具体的流程如下图。

![Calculate SFR](https://raw.githubusercontent.com/allincamera/imgur/master/resolution_1/sfr_calc.jpg)

其实简单得来说呢，SFR是通过这条斜边的图进行超采样的到一条更加细腻的黑白变换的直线（ESF）。然后通过这条直线求导得到直线的变化率（LSF）。然后对将这个变化率进行FFT变换就能得到各个频率下的MTF的值。这里面的ESF，LSF，都是什么呢? 

![Calculate SFR](https://raw.githubusercontent.com/allincamera/imgur/master/resolution_1/esf_lsf_mtf.jpg)

点扩展函数PSF(Point Spread Function)、线扩展函数LSF(Line Spread Function)和边缘扩展函数ESF(Edge Spread Function)是SFR 计算中的几个重要的概念。点扩展函数PSF是点光源成像后的亮度分布函数，如下图所示，用PSF(X,Y)表示。点扩展函数是中心圆对称的，通常以沿x轴的亮度分布PSF(X,Y)作为成像系统的点扩展函数。

![Calculate SFR](https://raw.githubusercontent.com/allincamera/imgur/master/resolution_1/psf.jpg)

当获取点光源像的亮度分布函数PSF(X,Y)后，对其进行二维傅里叶变换即可得MTF (u，v)。因此，从理论上讲，从PSF也是获取MTF的一个方法。但是，在实际的应用中，由于地面点光源强度很弱，此方法一般较少采用。相对于PSF来说，LSF的能量得到了一定程度的加强。SFR计​算MTF就通过ESF来得到LSF然后进行FFT得到MTF各个频率的值的。这几者之间的关系如下图。

![Calculate SFR](https://raw.githubusercontent.com/allincamera/imgur/master/resolution_1/psf_esf_lsf_mtf.jpg)

说实话光从这几个数学公式还是不好理解为什么ESF可以求出MTF。换一种角度理解LSF就是一条线上（ESF） 的变化的过称。对于任意一条线由黑变白的过程是由不同频率的黑白线对组成。因此可以反过来通过分析一条线得到这些频率下的　（FFT）。当然这只是一种朴素的理解。后面的文章中会有实际使用的MTF和SFR图卡和测试环境和问题进行进一步讨论

![Calculate SFR](https://raw.githubusercontent.com/allincamera/imgur/master/resolution_1/bottom.jpg)
