---
title: "Microfacet Theory | Real-time Rendering Chapter 9.7"
date: 2025-05-06 00:02:00 +0200
categories: [RTR4]
math: true
mermaid: true
toc: true
tip: true
tags: [graphics]     # TAG names should always be lowercase
---
# Microfacet Theory

Many BRDF models are based on a mathematical analysis of the effects of microgeometry on reflectance called microfacet theory. This tool was first developed by researchers in the optics community [124]. It was introduced to computer graphics in 1977 by Blinn [159] and again in 1981 by Cook and Torrance [285]. The theory is based on the modeling of microgeometry as a collection of *microfacets*.

Each of these tiny facets is flat, with a single microfacet normal $m$. The microfacets individually reflect light according to the micro-BRDF $f_\mu(l, v, m)$, with the combined reflectance across all the microfacets adding up to the overall surface BRDF. The usual choice is for each microfacet to be a perfect Fresnel mirror, resulting in a specular microfacet BRDF for modeling surface reflection. However, other choices are possible. Diffuse micro-BRDFs have been used to create several local subsurface scattering models [574, 657, 709, 1198, 1337]. A diffraction micro-BRDF was used to create a shading model combining geometrical and wave optics effects [763].

An important property of a microfacet model is the statistical distribution of the microfacet normals $m$. This distribution is defined by the surface¡Çs **normal distribution function**, or NDF. Some references use the term *distribution of normals* to avoid confusion with the Gaussian normal distribution. We will use $$D(m)$$ to refer to the NDF in equations.

The NDF $D(m)$ is the statistical distribution of microfacet surface normals over the microgeometry surface area [708]. Integrating $D(m)$ over the entire sphere of microfacet normals gives the area of the microsurface. More usefully, integrating $D(m)(n \cdot m)$, the projection of $D(m)$ onto the macrosurface plane, gives the area of the macrosurface patch that is equal to 1 by convention, as shown on the left side of Figure 9.31. In other words, the projection $D(m)(n \cdot m)$ is normalized:

$$ \int_{m\in \Theta} D(m)(n \cdot m) dm = 1$$

The integral is over the entire sphere, represented here by $\Theta$, unlike previous spherical integrals in this chapter that integrated over only the hemisphere centered on $n$, represented by $\Omega$. This notation is used in most graphics publications, though some references [708] use $\Omega$ to denote the complete sphere. In practice, most microstructure models used in graphics are heightfields, which means that $D(m) = 0$ for all directions m outside $\Omega$. However, Equation above is valid for non-heightfield microstructures as well.

![Figure 9.31](/images/fig9.31.png)
> Figure 9.31. Side view of a microsurface. On the left, we see that integrating $D(m)(n \cdot m)$, the microfacet areas projected onto the macrosurface plane, yields the area (length, in this side view) of the macrosurface, which is 1 by convention. On the right, integrating $D(m)(v \cdot m)$, the microfacet areas projected onto the plane perpendicular to $v$, yields the projection of the macrosurface onto that plane, which is cos $\theta_o$ or $(v \cdot n)$. When the projections of multiple microfacets overlap, the negative projected areas of the backfacing microfacets cancel out the "extra" frontfacing microfacets.
{: .prompt-info }

More generally, the projections of the microsurface and macrosurface onto the plane perpendicular to any view direction $v$ are equal:

$$ \int_{m\in \Theta} D(m)(v \cdot m) dm = v \cdot n$$

The dot products in the two Equations above are not clamped to 0. The right side of Figure 9.31 shows why. Equations impose constraints that the function $D(m)$ must obey to be a valid NDF.

Intuitively, the NDF is like a histogram of the microfacet normals. It has high values in directions where the microfacet normals are more likely to be pointing. Most surfaces have NDFs that show a strong peak at the macroscopic surface normal $n$. Section 9.8.1 will cover several NDF models used in rendering.

Take a second look at the right side of Figure 9.31. Although there are many microfacets with overlapping projections, ultimately for rendering we care about only the visible microfacets, i.e., the microfacets that are closest to the camera in each overlapping set. This fact suggests an alternative way of relating the projected microfacet areas to the projected macrogeometry area: The sum of the projected areas of the visible microfacets is equal to the projected area of the macrosurface. We can express this mathematically by defining the *masking function* $G_1(m, v)$, which gives the fraction of microfacets with normal $m$ that are visible along the view vector $v$. The integral of $G_1(m, v)D(m)(v \cdot m)^+$ over the sphere then gives the area of the macrosurface projected onto the plane perpendicular to $v$:

$$ \int_{m\in \Theta} G_1(m, v)D(m)(v \cdot m)^+ dm = v \cdot n$$

as shown in Figure 9.32. The dot product in Equation above is clamped to zero. Backfacing microfacets are not visible, so they are not counted in this case. The product $G_1(m, v)D(m)$ is the **distribution of visible normals** [708].



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
