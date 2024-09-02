---
title: "Procedural Texturing | Real-time Rendering Chapter 6.3"
date: 2024-09-01 00:00:00 +0200
categories: [RTR4,Texture]
math: true
mermaid: true
toc: true
tip: true
tags: [graphics]     # TAG names should always be lowercase
---
# Procedural Texturing
Although procedural textures are commonly used in offline rendering applications, image textures are far more common in real-time rendering. This is due to the extremely *high efficiency of the image texturing hardware* in modern GPUs, which can perform many billions of texture accesses in a second. However, GPU architectures are evolving toward *less expensive computation and (relatively) more costly memory access*. These trends have made procedural textures find greater use in real-time applications.

Volume textures are a particularly attractive application for procedural texturing, given the high storage costs of volume image textures. One of the most common is using one or more **noise** functions to generate values. A noise function is often sampled at successive powers-of-two frequencies, called *octaves*. Each octave is given a weight, usually falling as the frequency increases, and the sum of these weighted samples is called a *turbulence* function.

Other procedural methods are possible. For example, a cellular texture is formed by measuring distances from each location to a set of “feature points” scattered through space. Mapping the resulting closest distances in various ways.

When generating a procedural two-dimensional texture, parameterization issues (UV) can pose even more difficulties than for authored textures, where stretching or seam artifacts can be manually touched up or worked around.

Antialiasing procedural textures is both harder and easier than **antialiasing** image textures. On one hand, precomputation methods such as mipmapping are not available, putting the burden on the programmer. On the other, the procedural texture author has “inside information” about the texture content and so can tailor it to avoid aliasing. This is particularly true for procedural textures created by summing multiple noise functions. The frequency of each noise function is known, so any frequencies that would cause aliasing can be discarded, actually making the computation less costly. 