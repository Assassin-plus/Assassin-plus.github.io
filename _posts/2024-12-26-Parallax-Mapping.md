---
title: "Parallax Mapping | Real-time Rendering Chapter 6.8"
date: 2024-12-06 00:02:00 +0200
categories: [RTR4,Texture]
math: true
mermaid: true
toc: true
tip: true
tags: [graphics]     # TAG names should always be lowercase
---
# Parallax Mapping
A problem with bump and normal mapping is that the bumps never shift location with the view angle, nor ever block each other.

It would be better to have the bumps actually affect which location on the surface is rendered at each pixel.

*Parallax* refers to the idea that the positions of objects move relative to one another as the observer moves. As the viewer moves, the bumps should appear to have heights. The key idea of *parallax mapping* is to take an educated **guess** of what should be seen in a pixel by examining the height of what was found to be visible.

For parallax mapping, the bumps are stored in a heightfield texture. When viewing the surface at a given pixel, the heightfield value is retrieved at that location and used to *shift* the texture coordinates to retrieve a different part of the surface. The amount to shift is based on the height retrieved and the angle of the eye to the surface.

The heightfield values are either stored in a separate texture, or packed in an unused color or alpha channel of some other texture.

> care must be taken when packing unrelated textures together, since this can negatively impact compression quality.

The heightfield values are scaled and biased before being used to shift the coordinates. The scale determines how high the heightfield is meant to extend above or below the surface, and the bias gives the “sea-level” height at which no shift takes place. 

Given a texture-coordinate location $p$, an adjusted heightfield height $h$, and a normalized view vector $v$ with a height value $v_z$ and horizontal component $v_{xy}$, the new parallax-adjusted texture coordinate $p_{adj}$ is

$$p_{adj}=p+\frac{h\cdot v_{xy}}{v_z}$$

Note that unlike most shading equations, here the space in which the computation is performed matters—the view vector needs to be in tangent space.

> Though a simple approximation, this shifting works fairly well in practice if the bump heights change relatively slowly. Nearby neighboring texels then have about the same heights, so the idea of using the original location’s height as an estimate of the new location’s height is reasonable. 

> However, this method falls apart at shallow viewing angles. When the view vector is near the surface’s horizon, a small height change results in a large texture coordinate shift. The approximation fails, as the new location retrieved has little or no height correlation to the original surface location.

Offset limiting. The idea is to limit the amount of shifting to never be larger than the retrieved height.
The equation is then
$$p_{adj}'=p+h\cdot v_{xy}$$
Note that this equation is faster to compute than the original. Geometrically, the interpretation is that the height defines a radius beyond which the position cannot shift.

. At shallow angles, the offset becomes limited in its effect. Visually, this makes the bumpiness lessen at shallow angles, but this is much better than random sampling of the texture. Problems also remain with texture swimming as the view changes, or for stereo rendering, where the viewer simultaneously perceives two viewpoints that must give consistent depth cues. 

Even with these drawbacks, parallax mapping with offset limiting costs just a few additional pixel shader program instructions and gives a considerable image quality improvement over basic normal mapping.

## Parallax Occlusion Mapping

What we want is what is visible at the pixel, i.e., where the view vector first intersects the heightfield. One way is to use ray marching along the view vector until an (approximate) intersection point is found. This work can be done in the pixel shader where height data can be accessed as textures.

These types of algorithms are called ***parallax occlusion mapping*** (POM) or *relief mapping* methods, among other names. The key idea is to first test a fixed number of heightfield texture samples along the projected vector. More samples are usually generated for view rays at grazing angles, so that the closest intersection point is not missed. Each three-dimensional location along the ray is retrieved, transformed into texture space, and processed to determine if it is above or below the heightfield. Once a sample below the heightfield is found, the amount it is below, and the amount the previous sample is above, are used to find an intersection location. The location is then used to shade the surface, using the attached normal map, color map, and any other textures.

Multiple layered heightfields can be used to produce overhangs, independent overlapping surfaces, and two-sided reliefmapped impostors; The heightfield tracing approach can also be used to have the bumpy surface cast shadows onto itself.

One problem with all the methods is that the illusion breaks down along the silhouette edges of objects, which will show the original surface’s smooth outlines. The key idea is that the triangles rendered define which pixels should be evaluated by the pixel shader program, not where the surface actually is located. In addition, for curved surfaces, the problem of silhouettes becomes more involved. 