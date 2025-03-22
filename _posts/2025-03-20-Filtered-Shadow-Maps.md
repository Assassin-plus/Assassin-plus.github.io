---
title: "Filtered Shadow Maps | Real-time Rendering Chapter 7.7"
date: 2025-03-16 00:02:00 +0200
categories: [RTR4,Shadows]
math: true
mermaid: true
toc: true
tip: true
tags: [graphics]     # TAG names should always be lowercase
---
# Filtered Shadow Maps
One algorithm that allows filtering of the shadow maps generated is Donnelly and Lauritzen¡Çs ***variance shadow map*** (VSM) [368]. The algorithm stores the depth in one map and the depth squared in another map. MSAA or other antialiasing schemes can be used when generating the maps. These maps can be blurred, mipmapped, put in summed area tables [988], or any other method. The ability to treat these maps as filterable textures is a huge advantage, as the entire array of *sampling and filtering* techniques can be brought to bear when retrieving data from them.

We will describe VSM in some depth here, to give a sense of how this process works; also, the same type of testing is used for all methods in this class of algorithm.

To begin, for VSM the depth map is sampled (just once) at the receiver¡Çs location to return an average depth of the closest light occluder. When this average depth $M_1$, called the *first moment*, is greater than the depth on the shadow receiver $t$, the receiver is considered fully in light. When the average depth is less than the receiver¡Çs depth, the following equation is used:

$$p_{max}(t)=\frac{\sigma^2}{\sigma^2+(t-M_1)^2}$$

where $p_{max}$ is the maximum percentage of samples in light, $\sigma^2$ is the variance, $t$ is the receiver depth, and $M_1$ is the average expected depth in the shadow map. The depthsquared shadow map¡Çs sample $M_2$, called the *second moment*, is used to compute the variance:

$$\sigma^2=M_2-M_1^2$$

The value $p_{max}$ is an upper bound on the visibility percentage of the receiver. The actual illumination percentage $p$ cannot be larger than this value. This upper bound is from the one-sided variant of Chebyshev¡Çs inequality. The equation attempts to estimate, using probability theory, how much of the distribution of occluders at the surface location is beyond the surface¡Çs distance from the light. Donnelly and Lauritzen show that for a planar occluder and planar receiver at fixed depths, $p = p_{max}$, so the equation above can be used as a good approximation of many real shadowing situations.

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
