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

Blinn’s original bump mapping method stores two signed values, $b_u$ and $b_v$, at each texel in a texture. These two values correspond to the amount to vary the normal along the $u$ and $v$ image axes. That is, these texture values, which typically are bilinearly interpolated, are used to scale two vectors that are perpendicular to the normal. These two vectors are added to the normal to change its direction. The two values buand bvdescribe which way the surface faces at the point. This type of bump map texture is called an *offset vector bump map or offset map*.

Another way to represent bumps is to use a *heightfield* to modify the surface normal’s direction. Each monochrome texture value represents a height, so in the texture, white is a high area and black a low one (or vice versa). This is a common format used when first creating or scanning a bump map, and it was also introduced by Blinn in 1978. The heightfield is used to derive u and v signed values similar to those used in the first method. This is done by taking the differences between neighboring columns to get the slopes for u, and between neighboring rows for v. A variant is to use a Sobel filter, which gives a greater weight to the directly adjacent neighbors.

## Normal Mapping

A common method for bump mapping is to directly store a *normal map*. The algorithms and results are mathematically identical to Blinn’s methods; only the storage format and pixel shader computations change.

The normal map encodes (x, y, z) mapped to [−1, 1], e.g., for an 8-bit texture the x-axis value 0 represents −1.0 and 255 represents 1.0. The color [128, 128, 255], a light blue, would represent a flat surface for the color mapping shown, i.e., a normal of [0, 0, 1].

The normal map representation was originally introduced as a world-space normal map, which is rarely used in practice. For that type of mapping, the perturbation is straightforward: At each pixel, retrieve the normal from the map and use it directly, along with a light’s direction, to compute the shade at that location on the surface. Normal maps can also be defined in object space, so that the model could be rotated and the normals would then still be valid. However, both worldand object-space representations bind the texture to specific geometry in a particular orientation, which limits texture reuse.

Instead, the perturbed normal is usually retrieved in *tangent space*, i.e., relative to the surface itself. This allows for deformation of the surface, as well as maximal reuse of the normal texture. Tangent-space normal maps also can compress nicely, since the sign of the z-component (the one aligned with the unperturbed surface normal) can usually be assumed to be positive.

Normal mapping can be used to good effect to increase realism。

Filtering normal maps is a difficult problem, compared to filtering color textures. In general, the relationship between the normal and the shaded color is not linear, so standard filtering methods may result in objectionable aliasing. 
> Imagine looking at stairs made of blocks of shiny white marble. At some angles, the tops or sides of the stairs catch the light and reflect a bright specular highlight. However, the average normal for the stairs is at, say, a 45 degree angle; it will capture highlights from entirely different directions than the original stairs. 
When bump maps with sharp specular highlights are rendered without correct filtering, a distracting sparkle effect can occur as highlights wink in and out by the luck of where samples fall.

### Lambertian Surfaces

Lambertian surfaces are a special case where the normal map has an almost **linear effect on shading**. Lambertian shading is almost entirely a dot product, which is a linear operation. Averaging a group of normals and performing a dot product with the result is equivalent to averaging individual dot products with the normals:

$$\mathbf{l}\cdot \(\frac{\Sigma_{j=1}^{n}\mathbf{n}_j}{n}\)=\frac{\Sigma_{j=1}^n(\mathbf{l\cdot n}_j)}{n}$$

Note that the average vector is not normalized before use. Standard filtering and mipmaps almost produce the right result for Lambertian surfaces. The result is not quite correct because the Lambertian shading equation is not a dot product; it is a clamped dot product—max(l · n, 0). The clamping operation makes it nonlinear. This will overly darken the surface for glancing light directions, but in practice this is usually not objectionable. One caveat is that some texture compression methods typically used for normal maps (such as reconstructing the zcomponent from the other two) do not support non-unit-length normals, so using non-normalized normal maps may pose compression difficulties.

In the case of non-Lambertian surfaces, it is possible to produce better results by filtering the inputs to the shading equation as a group, rather than filtering the normal map in isolation.

Finally, it may be useful to derive a normal map from a height map, h(x, y). Care has to be taken at the boundaries of the texture.


Horizon mapping can be used to further enhance normal maps by having the bumps be able to cast shadows onto their own surfaces. This is done by precomputing additional textures, with each texture associated with a direction along the surface’s plane, and storing the angle of the horizon in that direction, for each texel. See Section 11.4 for more information.