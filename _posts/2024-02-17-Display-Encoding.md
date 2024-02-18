---
title: "Display Encoding | Real-time Rendering Chapter 5.6"
date: 2024-02-17 00:00:00 +0200
categories: [RTR4,Shading]
math: true
mermaid: true
toc: true
tip: true
tags: [graphics]     # TAG names should always be lowercase
---
# Display Encoding
When we calculate the effect of lighting, texturing, or other operations, the values used are assumed to be **linear**.
However, to avoid a variety of visual artifacts, display buffers and textures use **nonlinear encodings** that we must take into account.

In most cases you can tell the GPU to do ***gamma correction*** for you.

## Why Gamma Correction?
Display devices (CRT) exhibit a power law relationship between input voltage and display radiance. As the energy level applied to a pixel is increased, the radiance emitted does not grow linearly but (surprisingly) rises proportional to that level raised to a power greater than one.

**This power function nearly matches the inverse of the lightness sensitivity of human vision.**

The consequence of this fortunate coincidence is that the encoding is roughly *perceptually uniform*. That is, the perceived difference between a pair of encoded values N and N + 1 is roughly constant over the displayable range. Measured as *threshold contrast*, we can detect a difference in lightness of about 1% over a wide range of conditions. 

> This near-optimal distribution of values minimizes **banding artifacts** when colors are stored in **limited-precision display buffers** (Section 23.6). The same benefit also applies to textures, which commonly use the same encoding.
{: .prompt-tip }