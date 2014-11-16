####Ideal LPF

* Prerequisite: `DTFT`, `integration`
* Origin: [JOS/SASP](https://ccrma.stanford.edu/~jos/sasp/Ideal_Lowpass_Filter.html)

####LPF Design Specifications

* Prerequisite: `ideal LPF`(superficial)
* Origin: [JOS/SASP](https://ccrma.stanford.edu/~jos/sasp/Lowpass_Filter_Design_Specifications.html)

####Optimal Least-Squares Impulse Response Design

> * Summary: the frequency response of filter is optimal in L2 sense.

* Prerequisite: `DTFT`, `energy theorem`, `Ln norm`, `LPF specifications`
* Origin: [JOS/SASP](https://ccrma.stanford.edu/~jos/sasp/Optimal_but_poor_if.html)

####Window Method for FIR Filter Design

> * Summary: "windowing" a theoretically ideal filter impulse response by suitably chosen window function.

* Prerequisite: `DTFT`, `convolution theorem`, `ideal LPF`, `window`
* Origin: [JOS/SASP](https://ccrma.stanford.edu/~jos/sasp/Window_Method_FIR_Filter.html)

####Bandpass Filter Design using Kaiser Window

* Prerequisite: `kaiser window`, `window method for FIR design`
* Origin: [JOS/SASP](https://ccrma.stanford.edu/~jos/sasp/Bandpass_Filter_Design_Example.html)

####Hilbert Transform Design

> * Summary: design a complex bandpass filter which passes positive frequencies and rejects negative frequencies.

* Prerequisite: `window method for FIR design`, `DTFT of real signal`, `linearity of DTFT`， `aliasing`
* Origin: [JOS/SASP](https://ccrma.stanford.edu/~jos/sasp/Hilbert_Transform_Design_Example.html)

####Minimum Phase Filter Design

* Summary: Non-linear phase characteristic, same amplitude response, less delay.
* Prerequisite: `hilbert transform`, `z transform`
* Origin: [dspguru](http://www.dspguru.com/dsp/faqs/fir/properties) [JOS/SASP](https://ccrma.stanford.edu/~jos/sasp/Minimum_Phase_Filter_Design.html)

####Optimal FIR Design

* Summary: Chebyshev FIR Filters, Least-Squares FIR Filters and so on.
* Prerequisite: `linear algebra`, `DTFT`, `LMS`， `linear programming`
* Origin: [JOS/SASP](https://ccrma.stanford.edu/~jos/sasp/Optimal_FIR_Digital_Filter.html)
