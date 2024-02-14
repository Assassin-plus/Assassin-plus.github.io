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
{: .prompt-tip }

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

## Screen-Space Antialiasing

Algorithms discussed in this section are screen based, i.e., that they operate only on the output samples of the pipeline.
> There is no one best antialiasing technique, as each has different advantages in terms of quality, ability to capture sharp details or other phenomena, appearance during movement, memory cost, GPU requirements, and speed.
{: .prompt-tip }

The general strategy of screen-based antialiasing schemes is to use a sampling pattern for the screen and then weight and sum the samples to produce a pixel color.

### Supersampling

- Antialiasing algorithms that compute more than one *full sample per pixel* are called supersampling (or oversampling) methods. (**FSAA/SSAA**)
- A sampling method related to supersampling is based on the idea of the **accumulation buffer**. Instead of one large offscreen buffer, this method uses a buffer that has the same resolution as the desired image, but with more bits of color per channel. *The additional costs of having to re-render the scene a few times per frame and copy the result to the screen makes this algorithm costly for real-time rendering systems.*
    > The accumulation buffer used to be a separate piece of hardware. It was supported directly in the OpenGL API, but was deprecated in version 3.0. On modern GPUs the accumulation buffer concept can be implemented in a pixel shader by using a higher-precision color format for the output buffer.
    {: .prompt-tip }

- Additional samples are needed when phenomena such as object edges, specular highlights, and sharp shadows cause abrupt color changes. 
    > Aliasing of object edges still remains as a major sampling problem. It is possible to use analytical methods, where object edges are detected during rendering and their influence is factored in, but these are often *more expensive and less robust* than simply taking more samples. However, GPU features such as **conservative rasterization** and **rasterizer order views** have opened up new possibilities.
    {: .prompt-warning }

> Techniques such as supersampling and accumulation buffering work by generating samples that are fully specified with **individually computed** shades and depths. *The overall gains are relatively low and the cost is high*, as each sample has to run through a pixel shader.
{: .prompt-tip }

### MSAA

- **Multisampling antialiasing (MSAA)** lessens the high computational costs by computing the surface’s shade once per pixel and sharing this result among the samples.
![picture 1](</images/截屏2024-02-14 23.35.47.png>)
    Pixels may have, say, four (x,y) sample locations per fragment, each with their own color and z-depth, but the pixel shader is evaluated **only once for each object fragment** applied to the pixel. 
    If all MSAA positional samples are covered by the fragment, the shading sample is evaluated at the center of the pixel. If instead the fragment covers fewer positional samples, the shading sample’s position can be **shifted** to better represent the positions covered. Doing so avoids shade sampling off the edge of a texture, for example. This position adjustment is called **centroid sampling** or centroid interpolation and is done <u>automatically by the GPU</u>, if enabled. 
    Centroid sampling avoids off-triangle problems but can cause **derivative computations** to return incorrect values.

MSAA is faster than a pure supersampling scheme because the fragment is shaded only once. It focuses effort on sampling the fragment’s pixel **coverage** at a higher rate and **sharing** the computed shade. It is possible to save more memory by further de- coupling sampling and coverage, which in turn can make antialiasing faster still—the less memory touched, the quicker the render.

- EQAA (Enhanced Quality Antialiasing) is a technique that uses a combination of MSAA and a programmable resolve filter to improve the quality of the final image. It is a feature of some AMD GPUs.
  > EQAA’s “2f4x” mode stores two color and depth values, shared among four sample locations. The colors and depths are no longer stored for particular locations but rather saved in a table. Each of the four samples then needs just one bit to specify which of the two stored values is associated with its location. The coverage samples specify the contribution of each frag- ment to the final pixel color. 
    {: .prompt-tip }
  > If the number of colors stored is exceeded, a stored color is evicted and its samples are marked as unknown. These samples do not contribute to the final color. For most scenes there are relatively few pixels containing three or more visible opaque fragments that are radically different in shade, so this scheme performs well in practice.
    {: .prompt-tip }
