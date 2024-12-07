---
title: "Bump Mapping | Real-time Rendering Chapter 6.7"
date: 2024-12-06 00:02:00 +0200
categories: [RTR4,Texture]
math: true
mermaid: true
toc: true
tip: true
tags: [graphics]     # TAG names should always be lowercase
---
# Bump Mapping
This section describes a large family of small-scale detail representation techniques that we collectively call bump mapping. All these methods are typically implemented by modifying the *per-pixel shading routine*. They give a more three-dimensional appearance than texture mapping alone, but **without** adding any additional geometry.

Detail on an object can be classified into three scales: *macro*-features that cover many pixels, *meso*-features that are a few pixels across, and *micro*-features that are substantially smaller than a pixel. These categories are somewhat fluid, since the viewer may observe the same object at many distances during an animation or interactive session.

**Macrogeometry** is represented by vertices and triangles, or other geometric primitives.

**Microgeometry** is encapsulated in the shading model, which is commonly implemented in a pixel shader and uses texture maps as parameters. The shading model used simulates the interaction of a surface’s microscopic geometry, e.g., shiny objects are microscopically smooth, and diffuse surfaces are microscopically rough. The skin and clothes of a character appear to have different materials because they use different shaders, or at least different parameters in those shaders.

**Meso-geometry** describes everything between these two scales. It contains detail that is too complex to efficiently render using individual triangles, but that is large enough for the viewer to distinguish individual changes in surface curvature over a few pixels. The wrinkles on a character’s face, musculature details, and folds and seams in their clothing, are all mesoscale. A family of methods collectively known as *bump mapping* techniques are commonly used for mesoscale modeling. *These adjust the shading parameters at the pixel level in such a way that the viewer perceives small perturbations away from the base geometry, which actually remains flat*. The main distinctions between the different kinds of bump mapping are how they represent the detail features. Variables include the level of realism and complexity of the detail features. For example, it is common for a digital artist to carve details into a model, then use software to convert these geometric elements into one or more textures, such as a bump texture and perhaps a crevice-darkening texture.

## Tangent Frame

The tangent and bitangent vectors represent the axes of the normal map itself in the object’s space, since the goal is to transform the light to be relative to the map.

The matrix, sometimes abbreviated as TBN, transforms a light’s direction (for the given vertex) from world space to tangent space. These vectors do not have to be truly perpendicular to each other, since the normal map itself may be distorted to fit the surface. However, a non-orthogonal basis introduces skewing to the texture, which can mean more storage is needed and also can have performance implications, i.e., the matrix cannot then be inverted by a simple transpose. One method of saving memory is to store just the tangent and bitangent at the vertex and take their cross product to compute the normal. However, this technique works only if the handedness of the matrix is always the same. 

It is still possible to avoid storing the normal in this case if an extra bit of information is stored at each vertex to indicate the handedness. If set, this bit is used to negate the cross product of the tangent and bitangent to produce the correct normal. If the tangent frame is orthogonal, it is also possible to store the basis as a quaternion, which both is more space efficient and can save some calculations per pixel. A minor loss in quality is possible, though in practice is rarely seen.

## Blinn's Methods

