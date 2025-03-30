---
title: "Percentage-Closer Soft Shadows | Real-time Rendering Chapter 7.6"
date: 2025-03-16 00:02:00 +0200
categories: [RTR4,Shadows]
math: true
mermaid: true
toc: true
tip: true
tags: [graphics]     # TAG names should always be lowercase
---
# Percentage-Closer Soft Shadows

In 2005 Fernando  published an influential approach called *percentage-closer soft shadows* (PCSS). It attempts a solution by searching the nearby area on the shadow map to find all possible occluders. The average distance of these occluders from the location is used to determine the sample area width:

$$w_{sample}=w_{light}\frac{d_r-d_o}{d_r}$$

where $d_r$ is the distance of the receiver from the light and $d_o$ the average occluder distance. In other words, the width of the surface area to the sample grows as the average occluder gets farther from the receiver and closer to the light.

If there are no occluders found, the location is fully lit and no further processing is necessary. Similarly, if the location is fully occluded, processing can end. Otherwise, then the area of interest is sampled and the light's approximate contribution is computed. To save on processing costs, the width of the sample area can be used to vary the number of samples taken. Other techniques can be implemented, e.g., using lower sampling rates for distant soft shadows that are less likely to be important.

A drawback of this method is that it needs to sample a fair-sized area of the shadow map to find the occluders. Using a rotated Poisson disk pattern can help hide undersampling artifacts . Jimenez  notes that Poisson sampling can be unstable under motion and finds that a spiral pattern formed by using a function halfway between dithering and random gives a better result frame to frame.

Sikachev et al.  discuss in detail a faster implementation of PCSS using features in SM 5.0, introduced by AMD and often called by their name for it, *contact hardening shadows* (CHS). This new version also addresses another problem with basic PCSS: The penumbra's size is affected by the shadow map's resolution. This problem is minimized by first generating mipmaps of the shadow map, then choosing the mip level closest to a user-defined world-space kernel size. An 8 x 8 area is sampled to find the average blocker depth, needing only 16 $GatherRed()$ texture calls. Once a penumbra estimate is found, the higher-resolution mip levels are used for the sharp area of the shadow, while the lower-resolution mip levels are used
for softer areas.

CHS has been used in a large number of video games , and research continues. For example, Buades et al.  present *separable soft shadow mapping* (SSSM), where the PCSS process of sampling a grid is split into separable parts and elements are reused as possible from pixel to pixel.

One concept that has proven helpful for accelerating algorithms that need multiple samples per pixel is the hierarchical *min/max shadow map*. While shadow map depths normally cannot be averaged, the minimum and maximum values at each mipmap level can be useful. That is, two mipmaps can be formed, one saving the largest zdepth found in each area (sometimes called HiZ ), and one the smallest. Given a texel location, depth, and area to be sampled, the mipmaps can be used to rapidly determine fully lit and fully shadowed conditions. For example, if the texel's z-depth is greater than the maximum z-depth stored for the corresponding area of the mipmap, then the texel must be in shadow-no further samples are needed. This type of shadow map makes the task of determining light visibility much more efficient .

Methods such as PCF work by sampling the nearby receiver locations. PCSS works by finding an average depth of nearby occluders. These algorithms do not directly take into account the area of the light source, but rather sample the nearby surfaces, and are affected by the resolution of the shadow map. A major assumption behind PCSS is that the average blocker is a reasonable estimate of the penumbra size. When two occluders, say a street lamp and a distant mountain, partially occlude the same surface at a pixel, this assumption is broken and can lead to artifacts. Ideally, we want to determine how much of the area light source is visible from a single receiver location.

Several researchers have explored *backprojection* using the GPU. The idea is to treat each receiver's location as a viewpoint and the area light source as part of a view plane, and to project occluders onto this plane. Both Schwarz and Stamminger  and Guennebaud et al.  summarize previous work and offer their own improvements.

Bavoil et al.  take a different approach, using depth peeling to create a multi-layer shadow map. Backprojection algorithms can give excellent results, but the high cost per pixel has (so far) meant they have not seen adoption in interactive applications.
<!--
regex:\[\d+(?:,\s*\d+)*\]
## Lists

### Ordered list

1. Firstly
2. Secondly
3. Thirdly

### Unordered list

- Chapter
  + Section
    * Paragraph

### ToDo list

- [ ] Job
  + [x] Step 1
  + [x] Step 2
  + [ ] Step 3

### Description list

Sun
: the star around which the earth orbits

Moon
: the natural satellite of the earth, visible by reflected light from the sun

## Block Quote

> This line shows the _block quote_.

## Prompts

> An example showing the `tip` type prompt.
{: .prompt-tip }

> An example showing the `info` type prompt.
{: .prompt-info }

> An example showing the `warning` type prompt.
{: .prompt-warning }

> An example showing the `danger` type prompt.
{: .prompt-danger }

## Footnote

Click the hook will locate the footnote[^footnote], and here is another footnote[^fn-nth-2].

## Inline code

This is an example of `Inline Code`.

## Filepath

Here is the `/path/to/the/file.extend`{: .filepath}.

### Dark/Light mode & Shadow

The image below will toggle dark/light mode based on theme preference, notice it has shadows.

![light mode only](/posts/20190808/devtools-light.png){: .light .w-75 .shadow .rounded-10 w='1212' h='668' }
![dark mode only](/posts/20190808/devtools-dark.png){: .dark .w-75 .shadow .rounded-10 w='1212' h='668' }


## Reverse Footnote

[^footnote]: The footnote source
[^fn-nth-2]: The 2nd footnote source
-->
