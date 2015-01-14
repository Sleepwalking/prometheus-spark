<script type="text/javascript" src="http://cdn.mathjax.org/mathjax/latest/MathJax.js?config=default"></script>

Gammatone滤波器组：性质、实现和应用
===

本文是一篇对Gammatone滤波器组(Gammatone Filterbank)的介绍，总结了此主题下的几篇重要论文（Darling-1991, Slanney-1993, Ellis-2009, 见末尾的引用部分）。

0. 前言
---

####Gammatone滤波器组是干什么的？

一种在语音识别、语音分析领域中常用的时频转换方法，作用类似短时傅立叶变换(STFT)，但Gammatone滤波器组(以下简称GTF)结合了人耳的听觉特性。它是一种听觉滤波器组(Auditory Filterbank)。基于GTF的语音识别系统能获得较高的准确度。

####这篇文章有什么用？

本文主要是因为网上鲜有此类总结性的文章而写。省略了繁杂的推导，着重于应用。如果你在做语音科学或音频处理方面的研究，但愿本文能帮你节省一些读论文的时间。

####学习这篇文章需要哪些基础知识？

这篇文章并不是面向大众的科普，然而个人完全能够从网络上习得这些基础知识：

* **需要**数字信号处理(DSP)基础，我个人推荐Coursera上[EPFL的DSP课](https://www.coursera.org/course/dsp)，或者CCRMA的[Spectral Audio Signal Processing](https://ccrma.stanford.edu/~jos/sasp/)一书；

* **最好**会使用Matlab或Octave；

* **最好**掌握一些单变量微积分；

* **最好**了解一些音频处理，会很有帮助(例如Coursera上[UPF的ASPMA课](https://www.coursera.org/course/audio))。

1. 什么是滤波器组
---

顾名思义，滤波器组就是一组(n个)滤波器，对同一个信号进行滤波，输出n个同步的信号。

(test if MathJax works with Github)

$$x=\frac{-b\pm\sqrt{b^2-4ac}}{2a}$$

(撰写中，未完待续……)

A. 参考文献
---

* Darling, A. M. "Properties and implementation of the gammatone filter: a tutorial." Speech Hearing and Language, work in progress (1991): 43-61.

* Ellis, D. P. W. "Gammatone-like spectrograms", web resource. http://www.ee.columbia.edu/~dpwe/resources/matlab/gammatonegram/ (2009).

* Slaney, Malcolm. "An efficient implementation of the Patterson-Holdsworth auditory filter bank." Apple Computer, Perception Group, Tech. Rep 35 (1993): 8.
