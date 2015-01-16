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

假如一个滤波器组中各个滤波器的频率按升序排列，各集中在不同的频率，且滤波器数量足够多，我们可以计算出在不同时间的各个输出信号的短时能量，画成一串功率频谱(power spectrum)，或连起来成为声谱图(spectrogram)。

这张来源于[维基](http://zh.wikipedia.org/wiki/%E6%A2%85%E5%B0%94%E9%A2%91%E7%8E%87%E5%80%92%E8%B0%B1%E7%B3%BB%E6%95%B0)的图片给出了一个滤波器组的频率响应的例子：

<p align="center"><img src="http://i.stack.imgur.com/YUH48.gif" style="width:400px"/><p align="center">(这是一个梅尔滤波器组，由12个三角带通滤波器组成)</p></p>

Gammatone滤波器组生成的声谱图是什么样的？我们先睹为快：

<p align="center"><img src="https://cloud.githubusercontent.com/assets/4531595/5774035/84d7b0de-9da4-11e4-9b2b-496f315db1e3.png" style="width:600px"/><p align="center">
(这是cmu_slt_arctic语料库中的第一句话，使用的GTF滤波器组包含64个滤波器)
</p></p>


2. GTF滤波器的定义与性质
---

在介绍GTF滤波器组前，我们需要先了解GTF滤波器。前者由多个不同参数的后者组成。

###2.1 定义

GTF是一种线性滤波器，其脉冲响应(impulse response)如下：

<p align="center"><img src="http://latex.codecogs.com/gif.latex?h(t) = \underbrace{ct^{n - 1} e^{- 2\pi bt}}_{\text{gamma}} \underbrace{cos(2\pi f_0 t + \phi)}_{\text{tone}}, t > 0"/></p>

其中，

* c - 调节比例的常数；
* n - 滤波器级数(order)，一般取4；
* b - 衰减速度，取值为正数，b越大衰减越快，脉冲响应长度越短；
* f0 - 中心频率，当f0 = 0，此时的GTF称为一个基带GTF(base band gammatone filter)；
* ɸ - 相位，由于人耳对相位不敏感，可以省略；
* 这是连续时间下的定义，所以t单位为秒，f0单位为Hz，对于离散时间只需采样即可。

我们可以看到，时域上GTF就是一个Gamma分布乘上一个余弦信号，所以叫做“Gammatone”。也可以理解成一个余弦信号把这个Gamma分布调制到f0这个频率上。

下图是一个GTF脉冲响应的时域信号：

<p align="center"><img src="https://cloud.githubusercontent.com/assets/4531595/5734961/34821892-9bf9-11e4-9885-fe6d0acc2b8a.png"/></p>

###2.2 频率响应

我们可以想象一下GTF的频率响应的形状：如果把脉冲响应看作一个加窗的余弦波，那么窗的宽度决定了主瓣宽度，所以b决定了GTF的带宽：**b越大，窗越小，主瓣宽度越窄，GTF的带宽越窄**。

下图是一个GTF的频率响应(仅功率部分，对数幅度)：

<p align="center"><img src="https://cloud.githubusercontent.com/assets/4531595/5734963/37624dac-9bf9-11e4-9809-dbea0920143f.png"/></p>

因为这是一篇着重于应用的文章，这里省略GTF频率响应的解析表达式推导过程，直接给出该表达式如下：

<p align="center"><img src="http://latex.codecogs.com/gif.latex?H(f) = \frac{c}{2}(n - 1)! (2\pi b)^{-n} [P(f) + P^*(-f)]"/></p>

<p align="center"><img src="http://latex.codecogs.com/gif.latex?\text{where } P(f) = e^{i\phi}[1 + i\frac{(f - f_0)}{b}]^{-n}"/></p>

P(f)与P*(f)可看作正频率和负频率上的两个共轭对称的峰。这符合实数函数的傅立叶变换的对称性。

P(f)和“b越大，带宽越窄”的预测是一致的，且当f = f0时，P(F)有最大值![](http://latex.codecogs.com/gif.latex?e^{i%5cphi})。需注意的是这不是H(f)的最大值，或者说在中心频率处，频率响应不是最大，因为H(f)的值取决于P(f)和P*(-f)的和，这也说明了H(f)的正数部分或负数部分**并非关于f0(和-f0)对称**。

GTF滤波器的功率频谱函数，即![](http://latex.codecogs.com/gif.latex?|H%28f%29|^2)十分复杂，略。有兴趣的读者请参考Darling-1991。但是当**f0相对于b足够大**时，

<p align="center"><img src="http://latex.codecogs.com/gif.latex?H(f) \approx \frac{c}{2}(n - 1)! (2\pi b)^{-n} P(f),\quad \forall f \geq 0"/></p>
<p align="center"><img src="http://latex.codecogs.com/gif.latex?H(f) \approx \frac{c}{2}(n - 1)! (2\pi b)^{-n} P^*(-f),\quad \forall f < 0"/></p>

所以此时，

<p align="center"><img src="http://latex.codecogs.com/gif.latex?|H(f)|^2 \approx [\frac{c}{2}(n - 1)! (2\pi b)^{-n}]^2 [1 + \frac{(|f| - f_0)^2}{b^2}]^{-n}"/></p>

大多情况下我们不需要相位信息，所以这里不介绍相位谱函数的表达式。

###2.3 ERB带宽和3分贝带宽

####2.3.1 GTF的等效矩形带宽
等效矩形带宽(Equivalent Rectangular Bandwidth)指一种矩形带通滤波器的带宽，这中矩形带通滤波器的**高度和某个特定滤波器的功率谱最大值相同**，且**通带功率和这个特定滤波器的功率相同**。

换句话说，给定一个任意的功率谱，可以计算出一个等效矩形滤波器，这个矩形带通滤波器的增益就是给定功率谱的最大值，而该矩形滤波器的功率谱的总和和给定功率谱的总和相同。在这个条件下，可以计算出矩形滤波器的带宽，这就是ERB。

<p align="center"><img src="https://cloud.githubusercontent.com/assets/4531595/5772052/c3e4f42c-9d85-11e4-8c54-aba2bae95a03.png" style="width:350px"/></p>

一图胜过千言万语。上图是一个形象的例子。三角形代表一个三角滤波器，虚线矩形代表它的等效矩形滤波器。它们的高度相等、面积相等。此时矩形滤波器的带宽即为ERB。

GTF滤波器的ERB为，

<p align="center"><img src="http://latex.codecogs.com/gif.latex?ERB = \frac{(2n-2)! 2^{2-2n} \pi}{[(n-1)!]^2}b"/></p>

这很棒，因为对于一个给定的阶数n，ERB仅依赖于b，且关系是线性的。当n为4时，ERB = 0.98175b。我们也可以从一个给定的ERB求出b：

<p align="center"><img src="http://latex.codecogs.com/gif.latex?b \approx 1.019 ERB"/></p>

我们稍后会看到，ERB和人耳的听觉特性有关联。上述转换关系的作用是，我们可以用它方便地针对人耳听觉特性设计出合适的GTF滤波器，这是十分有用的。

####2.3.2 GTF滤波器的3分贝带宽

以信号幅度递减一半(相当于减弱3分贝)为标准，GTF滤波器的带宽近似于，

<p align="center"><img src="http://latex.codecogs.com/gif.latex?B_{3dB} \approx 2b\sqrt{2^{\frac{1}{n}} - 1}"/></p>

相当于

<p align="center"><img src="http://latex.codecogs.com/gif.latex?B_{3dB} \approx 0.87b}"/></p>

由于GTF频率响应的非对称性(见2.2)，我们无法求出准确的3分贝带宽。

###2.4 FIR实现

GTF滤波器的最简易实现方法是按照脉冲响应的定义，在时域生成离散的脉冲响应，作为FIR对输入信号滤波：

```matlab
g = @(t, f0, erb) t .^ 3 .* exp(- 2 * pi * 1.019 * erb * t) .* ...
    cos(2 * pi * f0 * t);
h = g((0:1 / fs:signal_length / fs)', central_freq, band_width);
y = filter(h, 1, x); % y = filter(B, A, x);
y = conv(h, x);      % 相当于和输入信号卷积
```

这里省略了比例常数c，我们将稍后讨论增益如何控制的问题。

由于脉冲响应的衰减很快，可以截取`h`，仅保留值较大的部分，可采用典型的窗函数法设计FIR滤波器。这样的好处是节省计算量，然而对于f0较低的情况，脉冲响应的衰减很慢，计算开销很难避免。对于GTF滤波器，FIR实现方式的计算开销平均比IIR实现慢了两、三个数量级，所以实际应用中FIR采用得较少。

GTF滤波器组
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
