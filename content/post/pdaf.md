+++
categories = ["hardware", "software", "system"]
date = "2016-04-07T13:32:07+08:00"
tags = ["sensor", "isp", "lens", "algorithm"]
title = "片上相差式自动对焦原理"
+++

相差式自动对焦与反差式自动对焦是现在手机成像系统中两大主要自动对焦方式。相比反差式自动对焦，相差式自动对焦只需要一次计算，就可以完成对焦。
	
当前比较流行的是片上相差自动对焦（on chip phase detection autofocus）, 在生产sensor的时候，把某些用于相位检测像素遮住左边一半或者右边一半，如下图

![Focus Pixels Array](https://raw.githubusercontent.com/allincamera/imgur/master/cdaf_pdaf/focus_pixel_array.png)

上图只是示意图，各个厂商的半掩模的工艺各有不同，在对IR filter或者microlens的处理上也不相同，但是基本的原理都是让图像形成左右两幅类似人眼的不同光学通路的图像。

这样左右侧的相位检测像素就会产生这样的图像：

![Phase diff in focus pixels](https://raw.githubusercontent.com/allincamera/imgur/master/cdaf_pdaf/focus_pixel_sequence.png)

数字化以后就产生了两个序列。

图像聚焦时，两个序列做互相关运算产生的数值变小，图像离焦时，两个序列做互相关产生的数值变大。如果对相机模组进行校准----针对一个固定图形的高频图像移动镜头，计算互相关运算产生的数值，记录下来成为基准表。在相机工作时，根据实时计算的互相关数值，通过查找基准表，就可以知道当前的离焦程度，从而找到移动方向和移动到什么位置。

数学推导简化起来就是如下公式：

左右两个图像产生的数列做互相关，得到一个对焦函数，可以把相差与镜头的偏移量变成一一对应关系。

![PDAF func](https://raw.githubusercontent.com/allincamera/imgur/master/cdaf_pdaf/pdaf_func.png)

实际工程上计算得到的结果就如下边图中所示，5x5窗口，每个窗口里边的统计数据包括两个部分，高16位是相位差，低16位是置信度。在平坦区域，置信度低，在细节丰富的区域，置信度高（300）。

![PDAF output](https://raw.githubusercontent.com/allincamera/imgur/master/cdaf_pdaf/pdaf_output.png)

通过固定图卡校准可以得到lens 偏移量和相差的对应数组：

> PDAF_Calibration[][2] = {{1,1},{2,3},{3,5},{4,7},{5,9},{6,10},{7,11},{8,12},{9,13},{10,14},{11,15},{12,16},{13,17},{14,18},{15,19},{16,20}, {20,30},{24,40},{28,47},{32,50},{40,70},{48,80},{56,96},{64,110},{80,138},{96,160},{112,180},{128,210}};
	
所以当AF开始工作时，通过实时计算得到相差值，eg： 210， 那么对应移动lens的距离，就是128,如果得到相差值是-210，就移动lens向反方向128个单位。
