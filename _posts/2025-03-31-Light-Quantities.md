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
Many of the RGB color values discussed in previous chapters represent intensities and shades of light. In this chapter we will learn about the various physical light quantities measured by these values, laying the groundwork for subsequent chapters, which discuss rendering from a more physically based perspective. We will also learn more about the often-neglected "second half" of the rendering process: the transformation of colors that represent scene linear light quantities into final display colors.

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

Before we get to the next quantity, we need to first introduce the concept of a *solid angle*, which is a three-dimensional extension of the concept of an angle. An angle can be thought of as a measure of the size of a continuous set of directions in a plane, with a value in radians equal to the length of the arc this set of directions intersects on an enclosing circle with radius 1. Similarly, a solid angle measures the size of a continuous set of directions in three-dimensional space, measured in *steradians* (abbreviated "sr"), which are defined by the area of the intersection patch on an enclosing sphere with radius 1 [544]. Solid angle is represented by the symbol $\omega$. In two dimensions, an angle of 2¦Ð radians covers the whole unit circle. Extending this to three dimensions, a solid angle of 4¦Ð steradians would cover the whole area of the unit sphere.

Now we can introduce *radiant intensity*, I, which is flux density with respect to direction?more precisely, solid angle ($\frac{d\Phi}{d\omega}$). It is measured in watts per steradian.

Finally, *radiance*, $L$, is a measure of electromagnetic radiation in a single ray. More precisely, it is defined as the density of radiant flux with respect to both area and solid angle ($d^2\Phi/dAd\omega$). This area is measured in a plane perpendicular to the ray. If radiance is applied to a surface at some other orientation, then a cosine correction factor must be used. You may encounter definitions of radiance using the term "projected area" in reference to this correction factor.

Radiance is what sensors, such as eyes or cameras, measure (see Section 9.2 for more details), so it is of prime importance for rendering. The purpose of evaluating a shading equation is to compute the radiance along a given ray, from the shaded surface point to the camera. The value of $L$ along that ray is the physically based equivalent of the quantity $c_{shaded}$ in Chapter 5. The metric units of radiance are watts per square meter per steradian.

The radiance in an environment can be thought of as a function of five variables (or six, including wavelength), called the *radiance distribution* [400]. Three of the variables specify a location, the other two a direction. This function describes all light traveling anywhere in space. One way to think of the rendering process is that the eye and screen define a point and a set of directions (e.g., a ray going through each pixel), and this function is evaluated at the eye for each direction. Image-based rendering, discussed in Section 13.4, uses a related concept, called the *light field*.

In shading equations, radiance often appears in the form $L_o(x, \vec{d})$ or $L_i(x, \vec{d})$, which mean radiance going out from the point $x$ or entering into it, respectively. The direction vector $\vec{d}$ indicates the ray¡Çs direction, which by convention always points away from $x$. While this convention may be somewhat confusing in the case of $L_i$, since $d$ points in the opposite direction to the light propagation, it is convenient for calculations such as dot products.

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
