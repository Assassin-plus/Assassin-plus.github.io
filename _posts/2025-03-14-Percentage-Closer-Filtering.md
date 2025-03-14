---
title: "Percentage-Closer Filtering | Real-time Rendering Chapter 7.5"
date: 2025-03-14 00:02:00 +0200
categories: [RTR4,Shadows]
math: true
mermaid: true
toc: true
tip: true
tags: [graphics]     # TAG names should always be lowercase
---
# Percentage-Closer Filtering

A simple extension of the shadow-map technique can provide pseudo-soft shadows.
This method can also help ameliorate resolution problems that cause shadows to
look blocky when a single light-sample cell covers many screen pixels. The solution is
similar to texture magnification (Section 6.2.1). Instead of a single sample being taken
off the shadow map, the four nearest samples are retrieved. The technique does not
interpolate between the depths themselves, but rather the results of their comparisons
with the surface¡Çs depth. That is, the surface¡Çs depth is compared separately to the
four texel depths, and the point is then determined to be in light or shadow for
each shadow-map sample. These results, i.e., 0 for shadow and 1 for light, are then
bilinearly interpolated to calculate how much the light actually contributes to the
surface location. This filtering results in an artificially soft shadow. These penumbrae
change, depending on the shadow map¡Çs resolution, camera location, and other factors.
For example, a higher resolution makes for a narrower softening of the edges. Still, a
little penumbra and smoothing is better than none at all.

This idea of retrieving multiple samples from a shadow map and blending the
results is called *percentage-closer filtering* (PCF) [1475].Area lights produce soft
shadows. The amount of light reaching a location on a surface is a function of what
proportion of the light¡Çs area is visible from the location. PCF attempts to approx-
imate a soft shadow for a punctual (or directional) light by reversing the process.
Instead of finding the light¡Çs visible area from a surface location, it finds the visibility
of the punctual light from a set of surface locations near the original location. See
Figure 7.22. The name ¡Èpercentage-closer filtering¡É refers to the ultimate goal, to find
the percentage of the samples taken that are visible to the light. This percentage is
how much light then is used to shade the surface.

![fig7.22](/images/fig7.22.png)

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
