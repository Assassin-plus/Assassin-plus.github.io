---
title: "Light Quantities | Real-time Rendering Chapter 8.1"
date: 2025-03-31 00:02:00 +0200
categories: [RTR4,Light and Colors]
math: true
mermaid: true
toc: true
tip: true
tags: [graphics]     # TAG names should always be lowercase
---
Many of the RGB color values discussed in previous chapters represent intensities and shades of light. In this chapter we will learn about the various physical light quantities measured by these values, laying the groundwork for subsequent chapters, which discuss rendering from a more physically based perspective. We will also learn more about the often-neglected ¡Èsecond half¡É of the rendering process: the transformation of colors that represent scene linear light quantities into final display colors.

# Light Quantities
The first step in any physically based approach to rendering is to *quantify* light in a precise manner. *Radiometry* is presented first, as this is the core field concerned with the physical transmission of light. We follow with a discussion of *photometry*, which deals with light values that are weighted by the sensitivity of the human eye. Our perception of color is a psychophysical phenomenon: the psychological perception of physical stimuli. Color perception is discussed in the section on *colorimetry*. Finally, we discuss the validity of rendering with *RGB* color values.

## Radiometry
*Radiometry* deals with the measurement of electromagnetic radiation. As will be discussed in more detail in Section 9.1, this radiation propagates as waves.

Radiometric quantities exist for measuring various aspects of electromagnetic radiation: overall energy, power (energy over time), and power density with respect to area, direction, or both. These quantities are summarized below.

| Name | Symbol | Units | 
| --- | --- | --- |
| radiant flux | $\Phi$ | W (watts) |
| irradiance | $E$ | W/m? |
| radiant intensity | $I$ | W/sr (watts per steradian) |
| radiance | $L$ | W/(m? sr) |

In radiometry, the basic unit is *radiant flux*, $\Phi$. Radiant flux is the flow of radiant energy over time?power?measured in watts (W).

*Irradiance* is the density of radiant flux with respect to area, i.e., d¦µ/dA. Irradiance is defined with respect to an area, which may be an imaginary area in space, but is most often the surface of an object. It is measured in watts per square meter.

Before we get to the next quantity, we need to first introduce the concept of a *solid angle*, which is a three-dimensional extension of the concept of an angle. An angle can be thought of as a measure of the size of a continuous set of directions in a plane, with a value in radians equal to the length of the arc this set of directions intersects on an enclosing circle with radius 1. Similarly, a solid angle measures the size of a continuous set of directions in three-dimensional space, measured in *steradians* (abbreviated ¡Èsr¡É), which are defined by the area of the intersection patch on an enclosing sphere with radius 1 [544]. Solid angle is represented by the symbol $\omega$. In two dimensions, an angle of 2¦Ð radians covers the whole unit circle. Extending this to three dimensions, a solid angle of 4¦Ð steradians would cover the whole area of the unit sphere.

Now we can introduce *radiant intensity*, I, which is flux density with respect to direction?more precisely, solid angle ($\frac{d\Phi}{d\omega}$). It is measured in watts per steradian.

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
