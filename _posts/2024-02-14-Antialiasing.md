---
title: "Aliasing & Antialiasing | Real-time Rendering Chapter 5.4"
date: 2024-02-14 00:00:00 +0200
categories: [RTR4,Shading]
math: true
mermaid: true
toc: true
tip: true
tags: [graphics]     # TAG names should always be lowercase
---
# Aliasing & Antialiasing
We will focus on what currently can be done in real time to alleviate aliasing artifacts.

## Sampling and Filtering Theory
sampling theorem:
For a signal to be sampled properly (i.e., so that it is possible to reconstruct the original signal from the samples), the sampling frequency has to be more than twice the maximum frequency of the signal to be sampled.

> It is impossible to entirely avoid aliasing problems when using point samples to render a scene, and we almost always use point sampling.
{ : .prompt-tip }

However, at times it is possible to know when a signal is band-limited. One example is when *a texture is applied to a surface*. It is possible to compute the frequency of the **texture samples** compared to the **sampling rate of the pixel**. If this frequency is lower than the Nyquist limit, then no special action is needed to properly sample the texture. If the frequency is too high, then a variety of algorithms are used to **band-limit the texture**.

### Reconstruction Filters
The sinc function is the perfect reconstruction filter when the sampling frequency is 1.0 (i.e., the maximum frequency of the sampled signal must be smaller than 1/2). This is useful when resampling the signal (next section). However, the filter width of the sinc is **infinite** and is **negative** in some areas, so it is rarely useful in practice. For applications where negative filter values are undesirable or impractical, filters with no negative lobes (often referred to generically as **Gaussian filters**, since they either derive from or resemble a Gaussian curve) are typically used.
After using any filter, a continuous signal is obtained. However, in computer graphics we cannot display continuous signals directly, but we can use them to **resample the continuous signal to another size**, i.e., either enlarging the signal, or diminishing it. This topic is discussed next.

### Resampling

Resampling is used to magnify or minify a sampled signal. 
Magnification:
Assume the sampled signal is reconstructed as shown in the previous section. Intuitively, since the signal now is perfectly reconstructed and continuous, all that is needed is to **resample** the reconstructed signal at the desired intervals.

Minification:
The frequency of the original signal is too high for the sampling rate to avoid aliasing. Instead it has been shown that a filter using **sinc(x/a)** should be used to create a continuous signal from the sampled one. After that, resampling at the desired intervals can take place.

![picture 0](</images/截屏2024-02-13 23.26.43.png>)