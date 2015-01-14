Gammatone滤波器组：性质、实现和应用
===

本文是一篇对Gammatone滤波器组(Gammatone filter bank)的介绍，总结了此主题下的几篇重要论文（Darling-1991, Slaney-1993, Ellis-2009, 见末尾的引用部分）。

0. 前言
---

###0.1 Gammatone滤波器组是干什么的？

一种在语音识别、语音分析领域中常用的时频转换方法，作用类似短时傅立叶变换(STFT)，但Gammatone滤波器组(以下简称GTF滤波器组)结合了人耳的听觉特性。它是一种听觉滤波器组(auditory filter bank)。基于GTF的语音识别系统能获得较高的准确度。

###0.2 这篇文章有什么用？

本文主要是因为网上鲜有此类总结性的文章而写。省略了繁杂的推导，着重于应用。如果你在做语音科学或音频处理方面的研究，但愿本文能帮你节省一些读论文的时间。

###0.3 学习这篇文章需要哪些基础知识？

这篇文章并不是面向大众的科普，然而个人完全能够从网络上习得这些基础知识：

* **需要**数字信号处理(DSP)基础，我个人推荐Coursera上[EPFL的DSP课](https://www.coursera.org/course/dsp)，或者CCRMA的[Spectral Audio Signal Processing](https://ccrma.stanford.edu/~jos/sasp/)一书；

* **最好**会使用Matlab或Octave；

* **最好**掌握一些单变量微积分；

* **最好**了解一些音频处理，会很有帮助(例如Coursera上[UPF的ASPMA课](https://www.coursera.org/course/audio))。

1. 概念 - 什么是滤波器组？
---

顾名思义，一个滤波器组(filter bank)就是一组(n个)滤波器，对同一个信号进行滤波，输出n个同步的信号。我们可以给每个滤波器指定不同的响应函数、中心频率、增益、带宽。

假如一个滤波器组中各个滤波器的频率按升续排列，各集中在不同的频率，且滤波器数量足够多，我们可以计算出在不同时间的各个输出信号的短时能量，画成一串功率频谱(power spectrum)，或连起来成为声谱图(spectrogram)。

这张来源于[维基](http://zh.wikipedia.org/wiki/%E6%A2%85%E5%B0%94%E9%A2%91%E7%8E%87%E5%80%92%E8%B0%B1%E7%B3%BB%E6%95%B0)的图片给出了一个滤波器组的频率响应的例子：

<p align="center"><img src="http://i.stack.imgur.com/YUH48.gif" style="width:400px"/><p align="center">(这是一个梅尔滤波器组，由12个三角带通滤波器组成)</p></p>

2. GTF滤波器的定义与性质
---

在介绍GTF滤波器组前，我们需要先了解GTF滤波器。前者由多个不同参数的后者组成。

###2.1 定义

GTF滤波器的脉冲响应(impulse response)如下：

<p align="center"><img src="http://latex.codecogs.com/gif.latex?h(t) = \underbrace{ct^{n - 1} e^{- 2\pi bt}}_{\text{gamma}} \underbrace{cos(2\pi f_0 t + \phi)}_{\text{tone}}, t > 0"/></p>

其中，

* c - 调节比例的常数
* n - 滤波器级数(order)，一般取4
* b - 衰减速度，取值为正数，b越大衰减越快，脉冲响应长度越短
* f0 - 中心频率，当f0 = 0，此时的GTF称为一个基带GTF(base band gammatone filter)
* ɸ - 相位，由于人耳对相位不敏感，可以省略

我们可以看到，时域上GTF就是一个Gamma分布乘上一个余弦信号，所以叫做“Gammatone”。也可以理解成一个余弦信号把这个Gamma分布调制到f0这个频率上。

下图是一个GTF脉冲响应的时域信号：

<p align="center"><img src="https://cloud.githubusercontent.com/assets/4531595/5734961/34821892-9bf9-11e4-9885-fe6d0acc2b8a.png"/></p>

###2.2 频率响应

我们可以想象一下GTF的频率响应的形状：如果把脉冲响应看作一个加窗的余弦波，那么窗的宽度决定了主瓣宽度，所以b决定了GTF的带宽。

下图是一个GTF的频率响应(仅功率部分，对数幅度)：

<p align="center"><img src="https://cloud.githubusercontent.com/assets/4531595/5734963/37624dac-9bf9-11e4-9809-dbea0920143f.png"/></p>

###2.3 ERB带宽和3分贝带宽

###2.4 FIR实现

3. GTF滤波器组
---

###3.1 ERB临界频带

###3.2 全极点IIR实现

###3.3 极点-零点IIR实现

###3.4 STFT实现

(撰写中，未完待续……)

A. 参考文献
---

* Darling, A. M. "Properties and implementation of the gammatone filter: a tutorial." Speech Hearing and Language, work in progress (1991): 43-61.

* Ellis, D. P. W. "Gammatone-like spectrograms", web resource. http://www.ee.columbia.edu/~dpwe/resources/matlab/gammatonegram/ (2009).

* Slaney, Malcolm. "An efficient implementation of the Patterson-Holdsworth auditory filter bank." Apple Computer, Perception Group, Tech. Rep 35 (1993): 8.
