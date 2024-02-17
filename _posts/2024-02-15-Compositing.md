---
title: "Transparency, Alpha, Compositing | Real-time Rendering Chapter 5.5"
date: 2024-02-14 00:00:00 +0200
categories: [RTR4,Shading]
math: true
mermaid: true
toc: true
tip: true
tags: [graphics]     # TAG names should always be lowercase
---
# Transparency, Alpha, Compositing
There are many different ways in which **semitransparent** objects can allow light to pass through them. For rendering algorithms, these can be roughly divided into:
- **Light-based** effects are those in which the object causes light to be attenuated or diverted, causing other objects in the scene to be lit and rendered differently. 
- **View-based** effects are those in which the semitransparent object itself is being rendered.

In this section we will deal with the simplest form of view-based transparency, in which the semitransparent object acts as an **attenuator** of the **colors** of the objects behind it. 
> More elaborate view- and light-based effects such as *frosted glass*, the bending of light (*refraction*), attenuation of light due to the *thickness* of the transparent object, and *reflectivity* and *transmission* changes due to the viewing angle are discussed in later chapters.
{: .prompt-tip }

### Screen-door Transparency
One of the simplest ways to give the illusion of transparency is to use a **screen-door transparency**. The idea is to render the transparent triangle with a **pixel-aligned checkerboard fill pattern**. That is, every other pixel of the triangle is rendered, thereby leaving the object behind it partially visible. 
-  A major drawback of this method is that often only one transparent object can be convincingly rendered on one area of the screen.
-  Also, the 50% checkerboard is limiting. Other larger pixel masks could be used to give other percentages, but these tend to create detectable patterns。
-  Advantage: simplicity. Transparent objects can be rendered at any time, in any order, and no special hardware is needed.
> This same idea is used for antialiasing edges of *cutout* textures, but at a subpixel level, using a feature called ***alpha to coverage*** (Section 6.6).
{: .prompt-tip }

### Stochastic Transparency
Stochastic transparency uses *subpixel screen-door masks* combined with stochastic sampling. 
A reasonable, though noisy, image is created by using *random stipple patterns* to represent the alpha coverage of the fragment.
- A **large** number of *samples* per pixel is needed for the result to look reasonable, as well as a sizable amount of *memory* for all the subpixel samples. 
- What is appealing is that **no blending** is needed, and antialiasing, transparency, and any other phenomena that *creates partially covered pixels* are covered by a single mechanism.

### Alpha Blending
Most transparency algorithms blend the transparent object’s color with the color of the object behind it. When an object is rendered on the screen, an RGB color and a z-buffer depth are associated with each pixel. Another component, called alpha (α), can also be defined for each pixel the object covers. 
Alpha is a value describing the degree of opacity and coverage of an object fragment for a given pixel.
> A pixel’s alpha can represent either opacity, coverage, or both, depending on the circumstances.
{: .prompt-tip }

## Blending Order
### Over Operator
To make an object appear transparent, it is rendered on **top** of the existing scene with an alpha of less than 1.0. Each pixel covered by the object will receive a resulting RGBα (also called RGBA) from the *pixel shader*. Blending this fragment’s value with the *original pixel color* is usually done using the **over** operator, as follows:
$$\mathbf{c_O} = \alpha_S\mathbf{c_S} + (1-\alpha_S)\mathbf{c_d}$$
where $c_s$ is the color of the transparent object (called the *source*), $α_s$ is the object’s alpha, $c_d$ is the pixel color before blending (called the *destination*), and $c_o$ is the resulting color due to placing the transparent object **over** the existing scene.
Transparency done this way works, in the sense that we perceive something as trans- parent whenever the objects behind can be seen through it. Using over simulates the real-world effect of a **gauzy fabric**. The view of the objects behind the fabric are partially obscured—the fabric’s threads are opaque. In practice, loose fabric has an alpha coverage that varies with angle.
> The over operator is less convincing simulating other transparent effects, most notably viewing through **colored glass** or **plastic**. A red filter held in front of a blue object in the real world usually makes the blue object look dark, as this object reflects little light that can pass through the red filter. (*physical transmittance* is discussed in Sections 14.5.1 and 14.5.2.)
{: .prompt-tip }
### Additive Blending
$$\mathbf{c_O}=\alpha_S\mathbf{c_S}+\mathbf{c_d}$$
This blending mode can work well for glowing effects such as lightning or sparks that do not attenuate the pixels behind but instead only brighten them.
For *several layered* semitransparent surfaces, such as smoke or fire, additive blending has the effect of **saturating** the colors of the phenomenon

### Depth Peeling
To render transparent objects properly, we need to draw them *after* the opaque objects. This is done by rendering all opaque objects first with blending **off**, then rendering the transparent objects with over turned **on**.
> In theory we could always have over on, since an opaque alpha of 1.0 would give the source color and hide the destination color, but doing so is more **expensive**, for no real gain.
{: .prompt-tip }

A limitation of the *z-buffer* is that only **one** object is stored per pixel. If several transparent objects overlap the same pixel, the z-buffer alone cannot hold and later resolve the effect of all the visible objects. When using over the transparent surfaces at any given pixel generally need to be rendered in **back-to-front** order. Not doing so can give incorrect perceptual cues.

#### Sort by Centroid
One way to achieve this ordering is to sort individual objects by, say, the distance of their centroids along the view direction. This rough sorting can work reasonably well, but has a number of problems under various circumstances.
- the order is just an approximation, so objects classified as more distant may be in front of objects considered nearer.
- Objects that **interpenetrate** are impossible to resolve on a per-mesh basis for all view angles, short of breaking each mesh into separate pieces.

because of its simplicity and speed, as well as needing no additional memory or special GPU support, performing a rough sort for transparency is still commonly used.

> If implemented, it is usually best to **turn off** z-depth replacement when performing transparency. That is, the z-buffer is still tested normally, but surviving surfaces do not change the z-depth stored; *the closest opaque surface’s depth is left intact*. 
> In this way, all transparent objects will at least appear in some form, versus suddenly appearing or disappearing when a camera rotation changes the sort order.
{: .prompt-tip }

Other techniques can also help improve the appearance, such as drawing each transparent mesh twice as you go, first rendering backfaces and then frontfaces. (Further Reading)

