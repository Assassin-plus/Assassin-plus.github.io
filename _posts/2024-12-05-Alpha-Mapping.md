---
title: "Alpha Mapping | Real-time Rendering Chapter 6.6"
date: 2024-12-05 00:02:00 +0200
categories: [RTR4,Texture]
math: true
mermaid: true
toc: true
tip: true
tags: [graphics]     # TAG names should always be lowercase
---
# Alpha Mapping

This section discusses the use of *textures with alphas*, noting various limitations and solutions along the way.

## Decal

By assigning an alpha of 0 to a texel, you make it transparent, so that it has no effect. So, by properly setting the decal texture’s alpha, you can replace or blend the underlying surface with the decal. Typically, a clamp corresponder function is used with a transparent border to apply a single copy of the decal (versus a repeating texture) to the surface.

## Making Cutouts
The principle is the same as for decals, except that instead of being flush with an underlying surface, the bush will be drawn on top of whatever geometry is behind it. In this way, using a single rectangle you can render an object with a complex silhouette.

In the case of the bush, if you rotate the viewer around it, the illusion fails, since the bush has no thickness. One answer is to copy this bush rectangle and rotate it 90 degrees along the trunk. The two rectangles form an inexpensive three-dimensional bush, sometimes called a “cross tree”, and the illusion is fairly effective when viewed from ground level.

Combining alpha maps and texture animation can produce convincing special effects, such as flickering torches, plant growth, explosions, and atmospheric effects.

## Rendering objects with alpha maps
Alpha blending allows for fractional transparency values, which enables antialiasing the object edges, as well as partially transparent objects. However, alpha blending requires rendering the blended triangles after the opaque ones, and in back-to-front order.

This problem can be ameliorated in several different ways when rendering.

### alpha testing

The process of conditionally discarding fragments with alpha values below a given threshold in the pixel shader.

This binary visibility test enables triangles to be rendered in any order because transparent fragments are discarded. We normally want to do this for any fragment with an alpha of 0.0. Discarding fully transparent fragments has the additional benefit of saving further shader processing and costs for merging, as well as avoiding incorrectly marking pixels in the z-buffer as visible.

For *cutouts* we often set the threshold value higher than 0.0, say, 0.5 or higher, and take the further step of then ignoring the alpha value altogether, not using it for blending.

Another solution is to perform two passes for each model—one for solid cutouts, which are written to the z-buffer, and the other for semitransparent samples, which are not.

### Alpha testing with mipmapping
When alpha testing is used with mipmapping, the effect can be unconvincing if not handled differently.

#### Correct alpha
Castano presents a simple solution done during mipmap creation that works well. For mipmap level k, the coverage ckis defined as

$$c_k=\frac{1}{n_k}\Sigma_i(\alpha(k,i)>\alpha_t)$$

where $n_k$ is the number of texels in mipmap level $k,\alpha(k, i)$ is the alpha value from mipmap level $k$ at pixel $i$, and $\alpha_t$ is the user-supplied alpha threshold.

Here, we assume that the result of $\alpha (k, i) > \alpha_t$ is 1 if it is true, and 0 otherwise.
Note that $k$ = 0 indicates the lowest mipmap level, i.e., the original image. For each mipmap level, we then find a new mipmap threshold value $\alpha_k$, instead of using $\alpha_t$, such that $c_k$ is equal to $c_0$ (or as close as possible). This can be done using a binary search. Finally, the alpha values of all texels in mipmap level $k$ are scaled by $\alpha_t/\alpha_k$

#### Random alpha
$if (texture.a < random()) discard;$

The random function returns a uniform value in [0, 1], which means that on average this will result in the correct result. For example, if the alpha value of the texture lookup is 0.3, the fragment will be discarded with a 30% chance. This is a form of stochastic transparency with a single sample per pixel. In practice, the random function is replaced with a hash function to avoid temporal and spatial high-frequency noise
$$float hash2D(x,y) { return fract(1.0e4*sin(17.0*x+0.1*y) * (0.1+abs(sin(13.0*y+x)))); }$$

A three-dimensional hash is formed by nested calls to the above function, i.e., $$float hash3D(x,y,z) { return hash2D(hash2D(x,y),z); }$$, which returns a number in [0, 1).

The input to the hash is object-space coordinates divided by the maximum screen-space derivatives (x and y) of the object-space coordinates, followed by clamping. Further care is needed to obtain stability for movements in the z-direction, and the method is best combined with temporal antialiasing techniques. This technique is faded in with distance, so that close up we do not get any stochastic effect at all.

> The advantage of this method is that every fragment is correct on average, while Castano’s method creates a single $α_k$ for each mipmap level. However, this value likely varies over each mipmap level, which may reduce quality and require artist intervention.

> Alpha testing displays ripple artifacts under magnification, which can be avoided by precomputing the alpha map as a distance field

### Alpha to coverage

Alpha to coverage, and the similar feature transparency adaptive antialiasing, take the *transparency value of the fragment and convert this into how many samples* inside a pixel are covered. This idea is like screen-door transparency, but at a subpixel level. 

> Imagine that each pixel has four sample locations, and that a fragment covers a pixel, but is 25% transparent (75% opaque), due to the cutout texture. The alpha to coverage mode makes the fragment become fully opaque but has it cover only three of the four samples. This mode is useful for cutout textures for overlapping grassy fronds, for example. Since each sample drawn is fully opaque, the closest frond will hide objects behind it in a consistent way along its edges. No sorting is needed to correctly blend semitransparent edge pixels, since alpha blending is turned off.

Alpha to coverage is good for antialiasing alpha testing, but can show artifacts when alpha blending. For example, two alpha-blended fragments with the same alpha coverage percentage will use the same subpixel pattern, meaning that one fragment will entirely cover the other instead of blending with it.

For any use of alpha mapping, it is important to understand how **bilinear interpolation affects the color values**.
Imagine two texels neighboring each other: rgbα = (255, 0, 0, 255) is a solid red, and its neighbor, rgbα = (0, 0, 0, 2), is black and almost entirely transparent. What is the rgbα for a location exactly midway between the two texels? Simple interpolation gives (127, 0, 0, 128), with the resulting rgb value alone a “dimmer” red. However, this result is not actually dimmer, it is a full red that has been premultiplied by its alpha. If you interpolate alpha values, for correct interpolation you need to ensure that the colors being interpolated are already premultiplied by alpha before interpolation.

Ignoring that the result of bilinear interpolation gives a premultiplied result can lead to *black edges* around decals and cutout objects. The “dimmer” red result gets treated as an unmultiplied color by the rest of the pipeline and the fringes go to black.

This effect can also be visible even if using alpha testing. The best strategy is to **premultiply before bilinear interpolation is done**. The WebGL API supports this, since compositing is important for webpages. However, bilinear interpolation is normally performed by the GPU, and operations on texel values cannot be done by the shader before this operation is performed. Images are not premultiplied in file formats such as PNG, as doing so would lose color precision. 

These two factors combine to cause black fringing by default when using alpha mapping. One common workaround is to **preprocess cutout images**, painting the transparent, “black” texels *with a color derived from nearby opaque texels*. All transparent areas often need to be repainted in this way, by hand or automatically, so that the mipmap levels also avoid fringing problems. It is also worth noting that premultiplied values should be used when forming mipmaps with alpha values