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
| irradiance | $E$ | W/m |
| radiant intensity | $I$ | W/sr (watts per steradian) |
| radiance | $L$ | W/(m * sr) |

In radiometry, the basic unit is *radiant flux*, $\Phi$. Radiant flux is the flow of radiant energy over time?power?measured in watts (W).

*Irradiance* is the density of radiant flux with respect to area, i.e., $d\Phi/dA$. Irradiance is defined with respect to an area, which may be an imaginary area in space, but is most often the surface of an object. It is measured in watts per square meter.

Before we get to the next quantity, we need to first introduce the concept of a *solid angle*, which is a three-dimensional extension of the concept of an angle. An angle can be thought of as a measure of the size of a continuous set of directions in a plane, with a value in radians equal to the length of the arc this set of directions intersects on an enclosing circle with radius 1. Similarly, a solid angle measures the size of a continuous set of directions in three-dimensional space, measured in *steradians* (abbreviated "sr"), which are defined by the area of the intersection patch on an enclosing sphere with radius 1 . Solid angle is represented by the symbol $\omega$. In two dimensions, an angle of $2\pi$ radians covers the whole unit circle. Extending this to three dimensions, a solid angle of $4\pi$ steradians would cover the whole area of the unit sphere.

Now we can introduce *radiant intensity*, I, which is flux density with respect to direction?more precisely, solid angle ($\frac{d\Phi}{d\omega}$). It is measured in watts per steradian.

Finally, *radiance*, $L$, is a measure of electromagnetic radiation in a single ray. More precisely, it is defined as the density of radiant flux with respect to both area and solid angle ($d^2\Phi/dAd\omega$). This area is measured in a plane perpendicular to the ray. If radiance is applied to a surface at some other orientation, then a cosine correction factor must be used. You may encounter definitions of radiance using the term "projected area" in reference to this correction factor.

Radiance is what sensors, such as eyes or cameras, measure (see Section 9.2 for more details), so it is of prime importance for rendering. The purpose of evaluating a shading equation is to compute the radiance along a given ray, from the shaded surface point to the camera. The value of $L$ along that ray is the physically based equivalent of the quantity $c_{shaded}$ in Chapter 5. The metric units of radiance are watts per square meter per steradian.

The radiance in an environment can be thought of as a function of five variables (or six, including wavelength), called the *radiance distribution* . Three of the variables specify a location, the other two a direction. This function describes all light traveling anywhere in space. One way to think of the rendering process is that the eye and screen define a point and a set of directions (e.g., a ray going through each pixel), and this function is evaluated at the eye for each direction. Image-based rendering, discussed in Section 13.4, uses a related concept, called the *light field*.

In shading equations, radiance often appears in the form $L_o(x, \vec{d})$ or $L_i(x, \vec{d})$, which mean radiance going out from the point $x$ or entering into it, respectively. 

> The direction vector $\vec{d}$ indicates the ray's direction, which by convention always points away from $x$. While this convention may be somewhat confusing in the case of $L_i$, since $d$ points in the opposite direction to the light propagation, it is convenient for calculations such as dot products.
{: .prompt-warning }

An important property of radiance is that it is not affected by distance, ignoring atmospheric effects such as fog. In other words, a surface will have the same radiance regardless of its distance from the viewer. The surface covers fewer pixels when more distant, but the radiance from the surface at each pixel is constant.

Most light waves contain a mixture of many different wavelengths. This is typically visualized as a *spectral power distribution* (SPD), which is a plot showing how the light's energy is distributed across different wavelengths. Clearly, human eyes make for poor spectrometers. We will discuss color vision in detail in Section 8.1.3. All radiometric quantities have spectral distributions. Since these distributions are densities over wavelength, their units are those of the original quantity divided by nanometers. For example, the spectral distribution of irradiance has units of watts per square meter per nanometer.

Since full SPDs are unwieldy to use for rendering, especially at interactive rates, in practice radiometric quantities are represented as RGB triples. In Section 8.1.3 we will explain how these triples relate to spectral distributions.

## Photometry

Radiometry deals purely with physical quantities, without taking account of human perception. A related field, *photometry*, is like radiometry, except that it weights everything by the sensitivity of the human eye. The results of radiometric computations are converted to photometric units by multiplying by the *CIE photometric curve*, a bell-shaped curve centered around 555 nm that represents the eye's response to various wavelengths of light.

> The full and more accurate name is the "CIE photopic spectral luminous efficiency curve." The word "photopic" refers to lighting conditions brighter than 3.4 candelas per square meter?twilight or brighter. Under these conditions the eye's cone cells are active. There is a corresponding "scotopic" CIE curve, centered around 507 nm, that is for when the eye has become dark-adapted to below 0.034 candelas per square meter, i.e. a moonless night or darker. The rod cells are active under these conditions.
{: .prompt-info }

![fig8.4](/images/fig8.4.png)

The conversion curve and the units of measurement are the only difference between the theory of photometry and the theory of radiometry. Each radiometric quantity has an equivalent metric photometric quantity. The Table below shows the names and units of each. The units all have the expected relationships (e.g., lux is lumens per square meter). Although logically the lumen should be the basic unit, historically the candela was defined as a basic unit and the other units were derived from it. In North America, lighting designers measure illuminance using the deprecated Imperial unit of measurement, called the foot-candle (fc), instead of lux. In either case, illuminance is what most light meters measure, and it is important in illumination engineering.

| Radiometric Quantity: Units | Photometric Quantity: Units |
| --- | --- |
| radiant flux : W | luminous flux : lumen (lm) |
| irradiance : $W/m^2$ | illuminance : lux (lx) |
| radiant intensity : $W/sr$ | luminous intensity : candela (cd) |
| radiance : $W/m^2/sr$ | luminance : candela per square meter (cd/m2= nit) |

Luminance is often used to describe the brightness of flat surfaces. For example, high dynamic range (HDR) television screens' peak brightness typically ranges from about 500 to 1000 nits. In comparison, clear sky has a luminance of about 8000 nits, a 60-watt bulb about 120,000 nits, and the sun at the horizon 600,000 nits.

## Colorimetry

In Section 8.1.1 we have seen that our perception of the color of light is strongly connected to the light's SPD (spectral power distribution). We also saw that this is not a simple one-to-one correspondence. *Colorimetry* deals with the relationship between spectral power distributions and the perception of color.


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
