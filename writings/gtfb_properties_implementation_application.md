Gammatone滤波器组：性质、实现和应用
===

本文是一篇对Gammatone滤波器组(Gammatone Filter Bank)的介绍，总结了此主题下的几篇重要论文(Darling-1991, Slaney-1993, Ellis-2009, 见末尾的引用部分)。

0. 前言
---

###0.1 Gammatone滤波器组是干什么的？

一种在语音识别、语音分析领域中常用的时频转换方法，作用类似短时傅立叶变换(STFT)，但Gammatone滤波器组(以下简称GTF滤波器组)结合了人耳的听觉特性。它是一种听觉滤波器组(Auditory Filter Bank)。基于GTF的语音识别系统能获得较高的准确度。

###0.2 这篇文章有什么用？

本文主要是因为网上鲜有此类总结性的文章而写。省略了繁杂的推导，着重于应用。如果你在做语音科学或音频处理方面的研究，但愿本文能帮你节省一些读论文的时间。

###0.3 学习这篇文章需要哪些基础知识？

这篇文章并不是面向大众的科普，然而个人完全能够从网络上习得这些基础知识：

* **需要**数字信号处理(DSP)基础，我个人推荐Coursera上[EPFL的DSP课](https://www.coursera.org/course/dsp)，或者CCRMA的[Spectral Audio Signal Processing](https://ccrma.stanford.edu/~jos/sasp/)一书；

* **最好**会使用Matlab或Octave；

* **最好**掌握一些单变量微积分；

* **最好**了解一些音频处理，会很有帮助(例如Coursera上[UPF的ASPMA课](https://www.coursera.org/course/audio))。

###0.4 声明

作者本人买不起Matlab，所以这篇文章中所有的Matlab代码都是在[GNU Octave](http://www.gnu.org/software/octave/)上测试运行的，本人不保证代码不经修改即可在Matlab上执行。

1. 概念 - 什么是滤波器组？
---

顾名思义，一个滤波器组(Filter Bank)就是一组(n个)滤波器，对同一个信号进行滤波，输出n个同步的信号。我们可以给每个滤波器指定不同的响应函数、中心频率、增益、带宽。

假如一个滤波器组中各个滤波器的频率按升序排列，各集中在不同的频率，且滤波器数量足够多，我们可以计算出在不同时间的各个输出信号的短时能量，画成一串功率频谱(Power Spectrum)，或连起来成为声谱图(Spectrogram)。

这张来源于[维基](http://zh.wikipedia.org/wiki/%E6%A2%85%E5%B0%94%E9%A2%91%E7%8E%87%E5%80%92%E8%B0%B1%E7%B3%BB%E6%95%B0)的图片给出了一个滤波器组的频率响应的例子：

<p align="center"><img src="http://i.stack.imgur.com/YUH48.gif" width="400"/><p align="center">(这是一个梅尔滤波器组，由12个三角带通滤波器组成)</p></p>

Gammatone滤波器组生成的声谱图是什么样的？我们先睹为快：

<p align="center"><img src="https://cloud.githubusercontent.com/assets/4531595/5774035/84d7b0de-9da4-11e4-9b2b-496f315db1e3.png" width="450"/><p align="center">
(这是cmu_slt_arctic语料库中的第一句话，使用的GTF滤波器组包含64个滤波器)
</p></p>


2. GTF滤波器的定义与性质
---

在介绍GTF滤波器组前，我们需要先了解GTF滤波器。前者由多个不同参数的后者组成。

本节先给出GTF滤波器的脉冲响应的定义，然后分别观察其时域和频域性质。最后借助matlab代码给出一个基于FIR滤波器的GTF滤波器实现。

###2.1 定义

GTF是一种线性滤波器，其脉冲响应(Impulse Response)如下：

<p align="center"><img src="http://latex.codecogs.com/gif.latex?h(t) = \underbrace{ct^{n - 1} e^{- 2\pi bt}}_{\text{gamma}} \underbrace{cos(2\pi f_0 t + \phi)}_{\text{tone}}, t > 0"/></p>

其中，

* c - 调节比例的常数；
* n - 滤波器级数(order)，一般取4；
* b - 衰减速度，取值为正数，b越大衰减越快，脉冲响应长度越短；
* f0 - 中心频率，当f0 = 0，此时的GTF称为一个基带GTF(Base Band Gammatone Filter)；
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

<p align="center"><img src="https://cloud.githubusercontent.com/assets/4531595/5772052/c3e4f42c-9d85-11e4-8c54-aba2bae95a03.png" width="400"/></p>

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

3. GTF滤波器组
---

本节介绍如何实现GTF滤波器组。首先我们引入临界频带的概念，从而计算出各GTF滤波器的中心频率和ERB。然后介绍GTF滤波器组的两种最常用的IIR实现。最后概括地介绍一种基于短时傅立叶变换(STFT)的快速近似实现。

###3.1 ERB临界频带

临界频带(Critical Band)是人的耳蜗的“滤波器”，在这个带宽内若同时存在两个声调，人对声调的感受会受干扰，产生听觉掩蔽效应(即一个强的音盖过一个弱的音)。临界频带的带宽即是前面提到的ERB。当然，我们的听觉系统比较复杂，在不同频率上的临界频带的带宽是不同的，而且随频率升高而升高，Moore和Glasberg于1990年提出如下经验公式，计算特定频率上的ERB：

<p align="center"><img src="http://latex.codecogs.com/gif.latex?ERB(f) = 24.7 * (0.00437f + 1)"/></p>

其中f单位为Hz，ERB(f)单位亦为Hz。

当我们决定设计一个有N个频道的GTF滤波器组时，如何确定每个频道的中心频率和带宽？假定有一个固定频率间隔sf，通过对<img src="http://latex.codecogs.com/gif.latex?\frac{1}{ERB(f)\cdot sf}" style="height:28px"/>作积分，可以获得一个将频率映射到频道编号(1至N)的函数。其反函数可把频道编号映射到频率：

<p align="center"><img src="http://latex.codecogs.com/gif.latex?f_0(n) = -228.7 + \frac{\frac{fs}{2} + 228.7}{e^{ 0.108 n \cdot sf}}}"/></p>

其中n为频道编号，fs为采样频率。设f为最低频率cf，且编号等于N时(因为耳蜗上频率的分布是颠倒的，所以采取了逆向编号顺序)，解得sf：

<p align="center"><img src="http://latex.codecogs.com/gif.latex?sf = \frac{1}{N}\{9.26[\log(\frac{fs}{2} + 228.7) - \log(cf + 228.7)]\}"/></p>

这样获得sf，再把sf带回f0(n)中，即求得了N个频道的中心频率。然后用ERB(f)计算各个频道的带宽，再代入2.3.1中b = 1.019ERB的关系中，即可获得各个GTF滤波器的参数。

ERB频率计算的matlab实现(这里cf取20)：

```matlab
function CF = make_erb_freqs(fs, n_channel)
    SF = 1 / n_channel * 9.26 * (log(fs + 228.7) - log(20 + 228.7));
    CF = - 228.7 + (fs + 228.7) ./ exp(0.108 * (1:n_channel)' * SF);
end
function ERB = erb_bandwidth(f0)
    B = 24.7 * (0.00437 * f0 + 1);
end
```

测试运行：

```
octave:1> make_erb_freqs(8000, 10)'
ans =

 Columns 1 through 7:

   5570.511   3858.317   2651.639   1801.227   1201.895    779.512    481.836

 Columns 8 through 10:

    272.047    124.198     20.000

octave:2> erb_bandwidth(ans)
ans =

 Columns 1 through 8:

   626.267   441.365   311.054   219.217   154.494   108.881    76.734    54.079

 Columns 9 and 10:

    38.112    26.860
```

###3.2 全极点IIR实现

使用无限脉冲响应(IIR)滤波器产生和FIR滤波器近似的脉冲响应，IIR的阶数较低时，经常仍可以产生不错的近似，如果误差在可以接受的范围内，就能极大地减少计算负担。GTF滤波器作为一种听觉滤波器，在语音识别等应用场合往往没有过高的精度要求，可使用一个全极点滤波器代替FIR滤波器。

1993年Slaney借助Mathematica设计出了一种由四个二阶全极点滤波器级联形成的GTF滤波器近似实现(具体方法是[脉冲响应不变法](https://ccrma.stanford.edu/~jos/pasp/Impulse_Invariant_Method.html)：计算出脉冲响应的拉普拉斯变换，进行部分分式分解，反向拉普拉斯变换回时域，采样，Z变换到频域获得传递函数)。为了减少计算开销，舍弃零点，成为四个传递函数一样的全极点滤波器，如下：

<p align="center"><img src="http://latex.codecogs.com/gif.latex?H_2(z) = \frac{-2T\sin(2f_0\pi T)z^{-1}}{e^{BT}[-4f_0\pi + 8f_0\pi\cos(2f_0\pi T)e^{-BT}z^{-1} - 4f_0\pi e^{-2BT}z^{-2}]}"/></p>

其中，

<p align="center"><img src="http://latex.codecogs.com/gif.latex?B = 2\pi b = 2\pi \cdot 1.019ERB(f)"/></p>

我们可以清楚地看到分子为一阶延迟(![](http://latex.codecogs.com/gif.latex?z^{-1}))，分母为二阶延迟。这个传递函数比较复杂，不过一旦给定了ERB和中心频率，可以计算出固定的滤波器参数，然后就能套用现成的IIR滤波函数。

方便起见，我们可以省略常数项，然后把分母除以<img src="http://latex.codecogs.com/gif.latex?-4f_0\pi"/>。为了确保在中心频率增益为1，我们把f0带进去，计算出<img src="http://latex.codecogs.com/gif.latex?H_2(z)|_{z=e^{2\pi f_0 T}}"/>，然后设定IIR的唯一一个前馈系数为<img src="http://latex.codecogs.com/gif.latex?\frac{1}{|H_2(e^{2\pi f_0 T})|}"/>。

以上是设计单个全极点IIR滤波器的过程。只需将四个同样的IIR滤波器级联，或者说把输入信号用同一个滤波器处理四次，即可形成GTF滤波器的IIR实现。

以下是设计单个二阶全极点IIR滤波器的Matlab代码：

```matlab
% pass_freq和pass_band可以是包含多个频道参数的向量
function [B, A] = make_erb_pass_allpole_cascade( ...
  fs, pass_freq, pass_band)
    T = 1 / fs;
    f0 = pass_freq;
    BW = 1.019 * 2 * pi * pass_band;
    E = exp(BW * T);
    n_channel = length(pass_freq);
    
    A = zeros(n_channel, 3);
    B = zeros(n_channel, 2);
    A(:, 1) = 1;
    A(:, 2) = - 2 * cos(2 * f0 * pi * T) ./ E;
    A(:, 3) = E .^ (-2);
    cz = exp(- 2 * j * pi * f0 * T);
    g = cz ./ (1 + A(:, 2) .* cz + A(:, 3) .* cz .^ 2);
    B(:, 2) = 1 ./ abs(g);
end
```

要设计一个GTF滤波器组只需把3.1中的`make_erb_freqs`函数和`make_erb_bandwidth`函数结合进去即可。这里为保证灵活性没有加入前者：

```matlab
function [B, A] = make_erb_bank_allpole_cascade(fs, bank_freq)
    BW = erb_bandwidth(bank_freq);
    [B A] = make_erb_pass_allpole_cascade(fs, bank_freq, BW);
end
```

最后定义一个很简单的函数即可实现对输入信号的滤波：

```matlab
% B和A都可以是频道数量 * 阶数大小的矩阵
% x是采样数 * 1大小的向量
% y是采样数 * 频道数量大小的矩阵
function y = gtf_allpole(B, A, x)
    n_channel = rows(B);
    y = zeros(length(x), n_channel);
    for i = 1:n_channel
        y(:, i) = filter(B(i, :), A(i, :), x);
        y(:, i) = filter(B(i, :), A(i, :), y(:, i));
        y(:, i) = filter(B(i, :), A(i, :), y(:, i));
        y(:, i) = filter(B(i, :), A(i, :), y(:, i));
    end
end
```

使用范例：

```matlab
[x fs] = wavread('xxx.wav');
n_channel = 64;
c = make_erb_freqs(8000, n_channel);
[B, A] = make_erb_bank_allpole_cascade(fs, c);
Y = gtf_allpole(B, A, x);
```

最后，其实存在一种直接把四个二阶IIR合并成一个八阶IIR的方案，然而对于一些低频率的频道，浮点误差会被放大，造成不稳定性(极点离单位圆太近导致)。正如Slaney-1993中提出的八阶极点-零点实现方式，虽然效率稍高，但在中心频率小于500Hz时会变得不稳定，只能拆成多个低阶IIR使用(有意思的是这个问题最初是由SASP的作者Julius Smith在1995年指出，不由感叹世界真小)。

这种实现方式和其它实现方式的效果对比可在下一小节末尾看到。

###3.3 极点-零点IIR实现

在上一小节的全极点实现方式中，零点由于较为复杂被省略，若加入零点，IIR的误差会大幅降低。这样生成的仍然是四个极点相同的二阶级联IIR滤波器，然而它们的零点不同。由于它们的极点和3.2中的一样，这里只给出零点(传递函数的分母)部分：

<p align="center"><img src="http://latex.codecogs.com/gif.latex?H_n(z) = \frac{B_n(z)}{A(z)}"/></p>

<p align="center"><img src="http://latex.codecogs.com/gif.latex?B_1(z) = -2T + \left(\cos(2f_0\pi T) + \sqrt{3+2^{\frac{3}{2}}}\sin(2f_0\pi T)\right)2Te^{-BT}z^{-1}"/></p>

<p align="center"><img src="http://latex.codecogs.com/gif.latex?B_2(z) = -2T + \left(\cos(2f_0\pi T) - \sqrt{3+2^{\frac{3}{2}}}\sin(2f_0\pi T)\right)2Te^{-BT}z^{-1}"/></p>

<p align="center"><img src="http://latex.codecogs.com/gif.latex?B_3(z) = -2T + \left(\cos(2f_0\pi T) + \sqrt{3-2^{\frac{3}{2}}}\sin(2f_0\pi T)\right)2Te^{-BT}z^{-1}"/></p>

<p align="center"><img src="http://latex.codecogs.com/gif.latex?B_4(z) = -2T + \left(\cos(2f_0\pi T) - \sqrt{3-2^{\frac{3}{2}}}\sin(2f_0\pi T)\right)2Te^{-BT}z^{-1}"/></p>

可以看到，它们唯一不同点在于两个符号的正负不同。为了方便实现可以定义一个函数，给定生成零点的第二项的系数的函数`zero_func`，生成一个(或多个)IIR滤波器的参数。用和全极点滤波器类似的方法，以下Matlab代码生成一个(或多个)极点-零点滤波器的参数，给定`zero_func`函数。

```matlab
function [B, A] = make_erb_pass_polezero_cascade( ...
  fs, pass_freq, pass_band, zero_func)
    T = 1 / fs;
    f0 = pass_freq;
    BW = 1.019 * 2 * pi * pass_band;
    E = exp(BW * T);
    n_channel = length(pass_freq);
    
    B = zeros(n_channel, 2);
    A = zeros(n_channel, 3);
    B(:, 1) = T;
    B(:, 2) = zero_func(T, f0, E);
    A(:, 1) = 1;
    A(:, 2) = - 2 * cos(2 * f0 * pi * T) ./ E;
    A(:, 3) = E .^ (-2);
    cz = exp(- 2 * j * pi * f0 * T);
    g = (T + B(:, 2) .* cz) ./ ...
        (1 + A(:, 2) .* cz + A(:, 3) .* cz .^ 2);
    B(:, 1) ./= abs(g);
    B(:, 2) ./= abs(g);
end
```

类似`make_erb_bank_allpole_cascade`，以下代码根据采样频率和各频道中心频率生成GTF滤波器组的参数：

```matlab
function [B1, B2, B3, B4, A] = make_erb_bank_polezero_cascade( ...
  fs, bank_freq)
    BW = erb_bandwidth(bank_freq);
    f0 = bank_freq;
    zcoef1 = @(T, cf, E) - (T * cos(2 * f0 * pi * T) ./ E ...
                         + sqrt(3 + 2 ^ 1.5) * T * sin(2 * f0 * pi * T) ./ E);
    zcoef2 = @(T, cf, E) - (T * cos(2 * f0 * pi * T) ./ E ...
                         - sqrt(3 + 2 ^ 1.5) * T * sin(2 * f0 * pi * T) ./ E);
    zcoef3 = @(T, cf, E) - (T * cos(2 * f0 * pi * T) ./ E ...
                         + sqrt(3 - 2 ^ 1.5) * T * sin(2 * f0 * pi * T) ./ E);
    zcoef4 = @(T, cf, E) - (T * cos(2 * f0 * pi * T) ./ E ...
                         - sqrt(3 - 2 ^ 1.5) * T * sin(2 * f0 * pi * T) ./ E);
    
    [B1 A] = make_erb_pass_polezero_cascade(fs, bank_freq, BW, zcoef1);
    B2     = make_erb_pass_polezero_cascade(fs, bank_freq, BW, zcoef2);
    B3     = make_erb_pass_polezero_cascade(fs, bank_freq, BW, zcoef3);
    B4     = make_erb_pass_polezero_cascade(fs, bank_freq, BW, zcoef4);
end
```

在执行滤波时，唯一的区别是每个滤波器的零点参数不同：

```matlab
function y = gtf_polezero(B1, B2, B3, B4, A, x)
    n_channel = rows(A);
    y = zeros(length(x), n_channel);
    for i = 1:n_channel
        y(:, i) = filter(B1(i, :), A(i, :), x);
        y(:, i) = filter(B2(i, :), A(i, :), y(:, i));
        y(:, i) = filter(B3(i, :), A(i, :), y(:, i));
        y(:, i) = filter(B4(i, :), A(i, :), y(:, i));
    end
end
```

下图比较了三种方法(FIR, 全极点IIR，极点-零点IIR)实现的GTF滤波器的脉冲响应和频率响应，FIR应当是最准确的；可以看到全极点IIR产生了较大的频谱倾斜：

<p align="center"><img src="https://cloud.githubusercontent.com/assets/4531595/5791640/20482e48-9f1e-11e4-9656-b88fcc7c3186.png" width="900px"/></p>

极点-零点IIR实现的GTF滤波器带来了性能和质量的折衷，所以这是最为常用的实现方式之一。

###3.4 STFT实现

基于FIR或IIR滤波器的实现必须要我们对时域信号进行滤波，尽管后者已经把效率提高了百倍，有时还是不甚满意。在对质量要求不高的场合下，完全可以把这个操作搬到频域进行：把采样过的单位圆代入3.3中的传递函数(Matlab中的`freqz`函数)，获得频率响应，和输入信号的频谱或短时频谱相乘，使用STFT合成，转换回时域。如果我们需要的仅是各频道的短时能量，那么直接求得相乘后频谱的能量即可。具体实现方式和[求得MFCC的过程](http://practicalcryptography.com/miscellaneous/machine-learning/guide-mel-frequency-cepstral-coefficients-mfccs/)类似(只是不需要DCT)，这里就不详述了。

这样做的缺点在于牺牲了高频的时间分辨率——高频的GTF滤波器衰减速度极快，延迟低，然而傅立叶变换把所有频率的时间分辨率都归一了。好在一般的应用不需要那么高的时间分辨率，而STFT实现对效率的提升很明显，特别是对于Matlab这种方便做矩阵计算的语言。在参考文献的Ellis-2009中可以找到一个基于STFT的GTF滤波器组实现。

###3.5 使用GTF滤波器组生成声谱图

这是一个最为简易、粗糙的应用，仅仅为了向你举例如何使用GTF滤波器组：

```matlab

[x fs] = wavread('sound.wav');
n = length(x);
n_channel = 64;

c = make_erb_freqs(8000, n_channel);
[B1, B2, B3, B4, A] = make_erb_bank_polezero_cascade(fs, c);
y = gtf_polezero(B1, B2, B3, B4, A, x);

for i = 1:n_channel
    % 和一个[1 1 1 ... 1]卷积，相当于做移动平均，以求得短时能量
    Y(:, i) = conv(y(:, i) .^ 2, ones(200, 1) / 200);
end

imagesc(log(Y(1:round(n/400):n, :))'); % 在每个输出信号中均匀地取400个采样，获得对数谱，绘制
xlim([0 400]);
ylim([0 n_channel]);
```

如果运行成功，会产生如第一节中所示的声谱图。注意用卷积计算短时能量的方法虽然剩了一点代码，但是带来了极大的计算负担，实际应用中每隔数百采样计算短时能量即可。

A. 参考文献
---

* Darling, A. M. "Properties and implementation of the gammatone filter: a tutorial." Speech Hearing and Language, work in progress (1991): 43-61.

* Ellis, D. P. W. "Gammatone-like spectrograms", web resource. http://www.ee.columbia.edu/~dpwe/resources/matlab/gammatonegram/ (2009).

* Slaney, Malcolm. "An efficient implementation of the Patterson-Holdsworth auditory filter bank." Apple Computer, Perception Group, Tech. Rep 35 (1993): 8.
