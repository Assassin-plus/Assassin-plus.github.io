---
title: "Plane Shadows | Real-time Rendering Chapter 7.1"
date: 2025-01-31 00:02:00 +0200
categories: [RTR4,Shadows]
math: true
mermaid: true
toc: true
tip: true
tags: [graphics]     # TAG names should always be lowercase
---
This chapter focuses on the basic principles of computing shadows, and describes the most important and popular real-time algorithms for doing so. We also briefly discuss approaches that are less popular but embody important principles.

*Occluders* are objects that cast shadows onto *receivers*. Punctual light sources, i.e., those with no area, generate only fully shadowed regions, sometimes called *hard shadows*. If area or volume light sources are used, then *soft shadows* are produced. Each shadow can then have a fully shadowed region, called the *umbra*, and a partially shadowed region, called the *penumbra*. Soft shadows are recognized by their fuzzy shadow edges.

Soft shadows are generally preferable because the penumbrae edges let the viewer know that the shadow is indeed a shadow. Hard-edged shadows usually look less realistic and can sometimes be misinterpreted as actual geometric features, such as a crease in a surface. However, hard shadows are faster to render than soft shadows.

More important than having a penumbra is having any shadow at all. Without some shadow as a visual cue, scenes are often unconvincing and more difficult to perceive.

> It is usually better to have an inaccurate shadow than none at all, as the eye is fairly forgiving about the shadow???s shape. 

In the following sections, we will go beyond these simple modeled shadows and present methods that compute shadows automatically in real time from the occluders in a scene. The first section handles the special case of shadows cast on *planar surfaces*, and the second section covers more general shadow algorithms, i.e., casting shadows onto *arbitrary surfaces*. Both hard and soft shadows will be covered. To conclude, some *optimization techniques* are presented that apply to various shadow algorithms.

# Plane Shadows

A simple case of shadowing occurs when objects cast shadows onto a planar surface.
A few types of algorithms for planar shadows are presented in this section, each with variations in the softness and realism of the shadows.

## Projection Shadows

In this scheme, the three-dimensional object is rendered a second time to create a shadow.
A matrix can be derived that projects the vertices of an object onto a plane. To render the shadow, simply apply this matrix to the objects that should cast shadows on the plane ??, and render this projected object with a dark color and no illumination. In practice, you have to take measures to avoid allowing the projected triangles to be rendered beneath the surface receiving them. One method is to add some *bias* to the plane we project upon, so that the shadow triangles are always rendered in front of the surface.

A safer method is to draw the ground plane first, then draw the projected triangles with the **z-buffer off**, then render the rest of the geometry as usual. The projected triangles are then always drawn on top of the ground plane, as no depth comparisons are made.

If the ground plane has a limit, e.g., it is a rectangle, the projected shadows may fall outside of it, breaking the illusion. To solve this problem, we can use a **stencil buffer**. First, draw the receiver to the screen and to the stencil buffer. Then, with the z-buffer off, draw the projected triangles only where the receiver was drawn, then render the rest of the scene normally.

Another shadow algorithm is to render the triangles into a texture, which is then applied to the ground plane. This texture is a type of *light map*, a texture that modulates the intensity of the underlying surface. As will be seen, this idea of rendering the shadow projection to a texture also allows penumbrae and shadows on curved surfaces. One drawback of this technique is that the texture can become magnified, with a single texel covering multiple pixels, breaking the illusion.

If the shadow situation does not change from frame to frame, i.e., the light and shadow casters do not move relative to each other, this texture can be *reused*. Most shadow techniques can benefit from reusing intermediate computed results from frame to frame if no change has occurred.

All shadow casters must be between the light and the ground-plane receiver. If the light source is below the topmost point on the object, an *antishadow* is generated, since each vertex is projected through the point of the light source. An error will also occur if we project an object that is below the receiving plane, since it too should cast no shadow.

It is certainly possible to explicitly cull and trim shadow triangles to avoid such artifacts. A simpler method, presented next, is to use the existing GPU pipeline to perform projection with clipping.

## Soft Shadows

Soft shadows appear whenever a light source has an area. One way to approximate the effect of an area light is to sample it by using several punctual lights placed on its surface. For each of these punctual light sources, an image is rendered and accumulated into a buffer. The average of these images is then an image with soft shadows. Note that, in theory, any algorithm that generates hard shadows can be used along with this accumulation technique to produce penumbrae. In practice, doing so at interactive rates is usually *untenable* because of the execution time that would be involved.

### Heckbert and Herf's Soft Shadows

Heckbert and Herf use a frustum-based method to produce their shadows. The idea is to treat the light as the viewer, and the ground plane forms the far clipping plane of the frustum. The frustum is made wide enough to encompass the occluders.

A soft shadow texture is formed by generating a series of ground-plane textures.
The area light source is sampled over its surface, with each location used to shade the image representing the ground plane, then to project the shadow-casting objects onto this image. All these images are summed and averaged to produce a ground-plane shadow texture.

> A problem with the sampled area-light method is that it tends to look like what it is: several overlapping shadows from punctual light sources. Also, for n shadow passes, only n + 1 distinct shades can be generated. A large number of passes gives an accurate result, but at an excessive cost. 
> The method is useful for obtaining a (literally) "ground-truth" image for testing the quality of other, faster algorithms.

### More Efficient Soft Shadows

A more efficient approach is to use convolution, i.e., filtering. Blurring a hard shadow generated from a single point can be sufficient in some cases and can produce a semitransparent texture that can be composited with real-world content. However, a uniform blur can be unconvincing near where the object makes contact with the ground.

There are many other methods that give a better approximation, at additional cost. For example, Haines starts with a projected hard shadow and then renders the silhouette edges with gradients that go from dark in the center to white on the edges to create *plausible* penumbrae. However, these penumbrae are not physically correct, as they should also extend to areas inside the silhouette edges. 

Iwanicki draws on ideas from spherical harmonics and approximates occluding characters with ellipsoids to give soft shadows. All such methods have various approximations and drawbacks, but are considerably more efficient than averaging a large set of drop-shadow images.