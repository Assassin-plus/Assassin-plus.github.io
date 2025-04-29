---
title: "Illumination | Real-time Rendering Chapter 9.4"
date: 2025-04-22 00:02:00 +0200
categories: [RTR4]
math: true
mermaid: true
toc: true
tip: true
tags: [graphics]     # TAG names should always be lowercase
---
# Illumination

The $L_i(l)$ (incoming radiance) term in the reflectance equation represents light impinging upon the shaded surface point from other parts of the scene. *Global illumination* algorithms calculate $L_i(l)$ by simulating how light propagates and is reflected throughout the scene. These algorithms use the *rendering equation*, of which the reflectance equation is a special case. Global illumination is discussed in Chapter 11. In this chapter and the next, we focus on **local illumination**, which uses the reflectance equation to compute shading locally at each surface point. In local illumination algorithms $L_i(l)$ is given and does not need to be computed.

In realistic scenes, $L_i(l)$ includes nonzero radiance from all directions, whether emitted directly from light sources or reflected from other surfaces. Unlike the directional and punctual lights discussed in Section 5.2, real-world light sources are *area lights* that cover a nonzero solid angle. In this chapter, we use a restricted form of $L_i(l)$ comprised of only directional and punctual lights, leaving more general lighting environments to Chapter 10. This restriction allows for a more focused discussion.

Although punctual and directional lights are non-physical abstractions, they can be derived as approximations of physical light sources. Such a derivation is important, because it enables us to incorporate these lights in a physically based rendering framework with confidence that we understand the error involved.

We take a small, distant area light and define $l_c$ as the vector pointing to its center. We also define the light's color $c_{light}$ as the reflected radiance from a white Lambertian surface facing toward the light $(n = l_c)$. This is an intuitive definition for authoring, since the color of the light corresponds directly to its visual effect.

With these definitions, a directional light can be derived as the limit case of shrinking the size of the area light down to zero while maintaining the value of $c_{light}$ . In this case the integral in the reflectance equation simplifies down to a single BRDF evaluation, which is significantly less expensive to compute: $$ L_o(v) = \pi f(l_c, v) c_{light} (n \cdot l_c) $$

The dot product $(n \cdot l)$ is often clamped to zero, as a convenient method of skipping contributions from lights under the surface: $$ L_o(v) = \pi f(l_c, v) c_{light} (n \cdot l_c)^+ $$

Punctual lights can be treated similarly. The only differences are that the area light is not required to be distant, and $c_{light}$ falls off as the inverse square of the distance to the light. 

In the case of more than one light source, Equation 9.12 is computed multiple times and the results are summed:$$ L_o(v) = \pi \Sigma_{i=1}^n f(l_{c_i}, v) c_{light_i} (n \cdot l_{c_i})^+$$


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
