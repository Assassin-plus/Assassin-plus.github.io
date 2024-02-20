---
title: "Premultiplied Alphas and Compositing | Real-time Rendering Chapter 5.5.3"
date: 2024-02-17 00:00:00 +0200
categories: [RTR4,Composite]
math: true
mermaid: true
toc: true
tip: true
tags: [graphics]     # TAG names should always be lowercase
---
# Premultiplied Alphas and Compositing
The over operator is also used for blending together photographs or synthetic renderings of objects. This process is called compositing.  The image formed by the alpha channel is sometimes called the **matte**. It shows the silhouette shape of the object.

One way to use synthetic RGBα data is with ***premultiplied alphas*** (also known as ***associated alphas***). That is, the RGB values are multiplied by the alpha value before being used. This makes the compositing over equation more efficient:

$$\mathbf{c_O=c_S'+(1-\alpha_S)c_d}$$

where $\mathbf{c_S'}$ is the premultiplied source channel.Premultiplied alpha also makes it possible to use over and additive blending *without changing the blend state*, since the source color is now added in during blending.

Rendering synthetic images dovetails naturally with premultiplied alphas. An antialiased opaque object rendered over a black background provides premultiplied values by default.

Another way images are stored is with unmultiplied alphas, also known as unassociated alphas or even as the mind-bending term nonpremultiplied alphas.It is best to use premultiplied data whenever filtering and blending is performed, as operations such as linear interpolation *do not work correctly* using unmultiplied alphas. Artifacts such as black fringes around the edges of objects can result.

For image-manipulation applications, an unassociated alpha is useful to mask a photograph without affecting the underlying image’s original data. Also, an unassociated alpha means that the **full precision range** of the color channels can be used. 
> care must be taken to properly convert unmultiplied RGBα values to and from the linear space used for computer graphics computations.
{: .prompt-warning }

Image file formats that support alpha include PNG (unassociated alpha only), OpenEXR (associated only), and TIFF (both types of alpha).
