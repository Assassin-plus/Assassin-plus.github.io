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


> The *display transfer function* describes the relationship between the digital values in the display buffer and the radiance levels emitted from the display. For this reason it is also called the *electrical optical transfer function* (EOTF). The display transfer function is part of the hardware, and there are different standards for computer monitors, televisions, and film projectors. There is also a standard transfer function for the other end of the process, image and video capture devices, called the optical electric transfer function (OETF).

When encoding linear color values for display, our goal is to cancel out the effect of the display transfer function, so that whatever value we compute will emit a corresponding radiance level. To maintain this connection, we apply the inverse of the display transfer function to cancel out its nonlinear effect. This process of nullifying the display’s response curve is also called ***gamma correction***. When decoding texture values, we need to apply the display transfer function to generate a linear value for use in shading.

![picture 1](</images/ß截屏2024-02-18 21.08.31.png>)