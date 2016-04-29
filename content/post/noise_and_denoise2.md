+++
categories = ["software"]
date = "2016-04-29T17:14:30+08:00"
tags = ["software", "algorithm"]
title = "图解噪声与去噪 之二：从『均值滤波』到『BM3D』"

+++

**本文系微信公众号《大话成像》，知乎专栏《大话成像 all in camera》原创文章，转载请注明出处。**

上一篇讲过了temporal noise和Fix patter noise的分离，通过多帧平均可以去除掉temporal noise，并分离出FPN，在这篇将介绍如何去除FPN。

在信号处理教科书中，介绍过很多经典的图像去噪方法，主要的是针对随机噪声的，对于图像中非随机噪声，比如sensor本身的物理缺陷导致的hot pixel，weak pixel 或是dead pixel，一般称之为impulse noise，对于impulse noise有单独的处理方法，因为他们不属于随机噪声。

随机噪声也就是在比图像的真实信号或高或低的不确定变化。

![random noise](https://raw.githubusercontent.com/allincamera/imgur/master/avg2bm3d/random_noise.png)

如果中间的虚线视作真实信号，红色和蓝色的曲线代表随机噪声叠加后的信号，如果虚线定义为0，那么所有随机噪声求和应为0 ，在统计学上叫零和噪声。由于零和噪声的这种特点，均值滤波可以降低图像的噪声。

![random noise average value](https://raw.githubusercontent.com/allincamera/imgur/master/avg2bm3d/random_noise_average_value.png)

如图所示，浅蓝色的线代表红蓝线求均值以后的信号，波动的幅度明显减小了，也就是噪声降低了。

## 均值滤波与变换域去噪

教科书里讲图像去噪声，第一个提到的就是均值滤波，在图像处理中，就是当前像素的值用周围n个像素的均值来代替。

![local average formula](https://raw.githubusercontent.com/allincamera/imgur/master/avg2bm3d/local_average_formula.png)

在实际信号处理中，就是用一个n x n 的模版 A 对图像进行卷积，比如：

![local average filter](https://raw.githubusercontent.com/allincamera/imgur/master/avg2bm3d/local_avg_filter.png)

当前像素就是处在矩阵中心的像素，它的值等于周围所有像素的值包括它自身取均值。

这样的一个基本均值滤波，它可以去掉噪声，但同时也会把图像搞模糊，比如当前的像素正好是一个图案的边缘，左边是白色的，像素值是200，右边是黑色的，像素值是10，做完均值滤波，（200+200+200+10+10+10+10+10+10）/9 = 74， 这样图像的细节就被模糊掉了。于是人们就对这种均值滤波进行了一些改进，比如增加图像边缘方向的判断，红线的方向上相邻像素的数值差不多，所以在做均值的时候只把这个方向的两个像素计算在内。这样既去掉了一些噪声，又保持了锐度。

![pixel array](https://raw.githubusercontent.com/allincamera/imgur/master/avg2bm3d/pixel_array.png)

这样由于做均值的像素变少了，去噪的效果不太好，于是有人想出一种none local mean的方法，也就是做均值的像素不再是领域的像素，扩大些范围找相似的，然后再做均值。        

![none local mean value](https://raw.githubusercontent.com/allincamera/imgur/master/avg2bm3d/none_local_mean.png)

绿色的部分和中心要处理的部分很相似，求均值的时候就把这些部分算进去，而红色的部分不相似，去均值的时候就排除这些部分。很容易想象，搜索的范围越大，计算量越大。

这些方法都是从空间的角度去思考如何去噪，也就是所谓的spatial noise reduction，这条路子能想的方法也都做得差不多了，于是有人换个角度想问题，就有了变换域做去噪的方法。通过数学变换，在变换域上把信号和噪声分离，然后把噪声过滤掉，剩下的就是信号。如下图，

没有噪声的信号看起来比较光滑：

![signal no noise](https://raw.githubusercontent.com/allincamera/imgur/master/avg2bm3d/signal_wo_noise.png)

带噪声的信号就会有些毛刺：

![signal with noise glitch](https://raw.githubusercontent.com/allincamera/imgur/master/avg2bm3d/signal_with_noise_glitch.png)

把带噪声的信号变换到一个域（比如频域，小波域等等)

![signal noise trans](https://raw.githubusercontent.com/allincamera/imgur/master/avg2bm3d/signal_noise_trans.png)

高于一个阈值的部分就是噪声，设一个截止值，把高于截止值的部分去掉

![signal noise trans threshold](https://raw.githubusercontent.com/allincamera/imgur/master/avg2bm3d/signal_noise_trans_threshold.png)

再做反变换，就得到了干净的信号。

![signal trans back](https://raw.githubusercontent.com/allincamera/imgur/master/avg2bm3d/signal_trans_back.png)

从频率上可以把噪声分为高中低频噪声，用这种变换域的方法就可以把不同频率的噪声分离，然后有效的去掉。像傅立叶变换，小波变换都是比较常见的变换域方法。

## BM3D去噪

简单来说，BM3D融合了spatial denoise和tranform denoise，可以得到最高的峰值信噪比。它先吸取了NLM中的计算相似块的方法，然后又融合了小波变换域去噪的方法。这是芬兰Tampere工业大学在2007年发表的论文里提出的算法。（了解NOKIA的人就知道Tampere这个城市就是NOKIA最早起源的地方）。

具体算法如下：

第一步，搜索相似块，然后把相似的块grouping成一个个3D stack。（图像本身是2D，变成stack就成了3D）

![BM3D Step1](https://raw.githubusercontent.com/allincamera/imgur/master/avg2bm3d/bm3d1_wm.png)

第二步，把这些3D stack进行变换域

![BM3D Step2](https://raw.githubusercontent.com/allincamera/imgur/master/avg2bm3d/bm3d2_wm.png)

第三步，3D 协同滤波----听着很邪乎，细节需要自行wiki

![BM3D Step3](https://raw.githubusercontent.com/allincamera/imgur/master/avg2bm3d/bm3d3_wm.png)

第四步，反变换以及blending

![BM3D Step4](https://raw.githubusercontent.com/allincamera/imgur/master/avg2bm3d/bm3d4_wm.png)

红色模块里提到的在变换域中thresholding的设定，有自适应计算的方法，但在工程中用的效果比较好的是噪声模型法，关于噪声模型，我们会在下一篇中进行介绍。

**本文系微信公众号《大话成像》，知乎专栏《大话成像 all in camera》原创文章，转载请注明出处。**
