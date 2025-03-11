---
title: "Shadow Mapping | Real-time Rendering Chapter 7.4"
date: 2025-02-25 00:02:00 +0200
categories: [RTR4,Shadows]
math: true
mermaid: true
toc: true
tip: true
tags: [graphics]     # TAG names should always be lowercase
---
# Shadow Mapping

A common z-buffer-based renderer could be used to generate shadows quickly on arbitrary objects. The idea is to render the scene, using the z-buffer, **from the position of the light source** that is to cast shadows. Whatever the light "sees" is illuminated, the rest is in shadow. When this image is generated, only z-buffering is required. Lighting, texturing, and writing values into the color buffer can be turned off.

Each pixel in the z-buffer now contains the z-depth of the object closest to the light source. We call the entire contents of the z-buffer the *shadow map*, also sometimes known as the *shadow depth map* or *shadow buffer*. To use the shadow map, the scene is rendered a second time, but this time with respect to the viewer. As each drawing primitive is rendered, its location at each pixel is compared to the shadow map. If a rendered point is farther away from the light source than the corresponding value in the shadow map, that point is in shadow, otherwise it is not. This technique is implemented by using *texture mapping*. Shadow mapping is a popular algorithm because it is relatively predictable. The cost of building the shadow map is roughly linear with the number of rendered primitives, and access time is constant.

The shadow map can be generated once and reused each frame for scenes where the light and objects are not moving, such as for computer-aided design.

![figure 7.10](</images/2025-03-10_18.02.12.png>)

> Shadow mapping. On the top left, a shadow map is formed by storing the depths to the surfaces in view. On the top right, the eye is shown looking at two locations. The sphere is seen at point $v_a$, and this point is found to be located at texel $a$ on the shadow map. The depth stored there is not (much) less than point $v_a$ is from the light, so the point is illuminated. The rectangle hit at point $v_b$ is (much) farther away from the light than the depth stored at texel $b$, and so is in shadow. On the bottom left is the view of a scene from the light¡Çs perspective, with white being farther away. On the bottom right is the scene rendered with this shadow map.
{: .prompt-info }

When a single z-buffer is generated, the light can "look" in only a particular direction, like a camera. For a distant directional light such as the sun, the light¡Çs view is set to encompass all objects casting shadows into the viewing volume that the eye sees. The light uses an orthographic projection, and its view needs to be made wide and high enough in x and y to view this set of objects. Local light sources need similar adjustments, as possible. If the local light is far enough away from the shadow-casting objects, a single view frustum may be sufficient to encompass all of these. Alternately, if the local light is a spotlight, it has a natural frustum associated with it, with everything outside its frustum considered not illuminated.

If the local light source is inside a scene and is surrounded by shadow-casters, a typical solution is to use a six-view cube, similar to cubic environment mapping. These are called *omnidirectional shadow maps*. The main challenge for omnidirectional maps is avoiding artifacts along the seams where two separate maps meet. Forsyth presents a general multi-frustum partitioning scheme for omnidirectional lights that also provides more shadow map resolution where needed. Crytek sets the resolution of each of the six views for a point light based on the screen-space coverage of each view¡Çs projected frustum, with all maps stored in a texture atlas.


Not all objects in the scene need to be rendered into the light¡Çs view volume. First, only objects that can cast shadows need to be rendered. For example, if it is known that the ground can only receive shadows and not cast one, then it does not have to be rendered into the shadow map.

Shadow casters are by definition those inside the light¡Çs view frustum. This frustum can be augmented or tightened in several ways, allowing us to safely disregard some shadow casters. Think of the set of shadow receivers visible to the eye. This set of objects is within some maximum distance along the light¡Çs view direction. Anything beyond this distance cannot cast a shadow on the visible receivers. Similarly, the set of visible receivers may well be smaller than the light¡Çs original x and y view bounds. Another example is that if the light source is inside the eye¡Çs view frustum, no object outside this additional frustum can cast a shadow on a receiver. Another example is that if the light source is inside the eye¡Çs view frustum, no object outside this additional frustum can cast a shadow on a receiver. Rendering only relevant objects not only can save time rendering, but can also reduce the size required for the light¡Çs frustum and so increase the effective resolution of the shadow map, thus improving quality. In addition, it helps if the light frustum¡Çs near plane is as far away from the light as possible, and if the far plane is as close as possible. Doing so increases the effective precision of the z-buffer.

![fig7.11](/images/fig7.11.png)

> On the left, the light¡Çs view encompasses the eye¡Çs frustum. In the middle, the light¡Çs far plane is pulled in to include only visible receivers, so culling the triangle as a caster; the near plane is also adjusted. On the right, the light¡Çs frustum sides are made to bound the visible receivers, culling the green capsule.
{: .prompt-info }

One disadvantage of shadow mapping is that the quality of the shadows depends on the resolution (in pixels) of the shadow map and on the numerical precision of the z-buffer. Since the shadow map is sampled during the depth comparison, the algorithm is susceptible to aliasing problems, especially close to points of contact between objects. A common problem is *self-shadow aliasing*, often called "surface acne" or "shadow acne," in which a triangle is incorrectly considered to shadow itself.
This problem has two sources. One is simply the numerical limits of precision of the processor. The other source is geometric, from the fact that the value of a point sample is being used to represent an area¡Çs depth. That is, samples generated for the light are almost never at the same locations as the screen samples (e.g., pixels are often sampled at their centers). When the light¡Çs stored depth value is compared to the viewed surface¡Çs depth, the light¡Çs value may be slightly lower than the surface¡Çs, resulting in self-shadowing.

One common method to help avoid (but not always eliminate) various shadow-map artifacts is to introduce a bias factor. When checking the distance found in the shadow map with the distance of the location being tested, a small bias is subtracted from the receiver¡Çs distance. This bias could be a constant value, but doing so can fail when the receiver is not mostly facing the light. A more effective method is to use a bias that is proportional to the angle of the receiver to the light.
The more the surface tilts away from the light, the greater the bias grows, to avoid the problem. This type of bias is called the *slope scale bias*. Both biases can be applied by using a command such as OpenGL¡Çs glPolygonOffset() to shift each polygon away from the light. Note that if a surface directly faces the light, it is not biased backward at all by slope scale bias. For this reason, a constant bias is used along with slope scale bias to avoid possible precision errors. Slope scale bias is also often clamped at some maximum, since the tangent value can be extremely high when the surface is nearly edge-on when viewed from the light.

Holbert introduced normal offset bias, which first shifts the receiver¡Çs world-space location a bit along the surface¡Çs normal direction, proportional to the sine of the angle between the light¡Çs direction and the geometric normal. This changes not only the depth but also the x- and y-coordinates where the sample is tested on the shadow map. As the light¡Çs angle becomes more shallow to the surface, this offset is increased, in hopes that the sample becomes far enough above the surface to avoid self-shadowing. This method can be visualized as moving the sample to a "virtual surface" above the receiver. This offset is a worldspace distance, so Pettineo recommends scaling it by the depth range of the shadow map. Pesce suggests the idea of biasing along the camera view direction, which also works by adjusting the shadow-map coordinates. Other bias methods are discussed in Section 7.5, as the shadow method presented there needs to also test several neighboring samples.

Too much bias causes a problem called *light leaks* or Peter Panning, in which the object appears to float slightly above the underlying surface. This artifact occurs because the area beneath the object¡Çs point of contact, e.g., the ground under a foot, is pushed too far forward and so does not receive a shadow.

One way to avoid self-shadowing problems is to render only the backfaces to the shadow map. Called *second-depth shadow mapping*, this scheme works well for many situations, especially for a rendering system where hand-tweaking a bias is not an option. The problem cases occur when objects are two-sided, thin, or in contact with one another. If an object is a model where both sides of the mesh are visible, e.g., a palm frond or sheet of paper, self-shadowing can occur because the backface and the frontface are in the same location. Similarly, if no biasing is performed, problems can occur near silhouette edges or thin objects, since in these areas backfaces are close to frontfaces. Adding a bias can help avoid surface acne, but the scheme is more susceptible to light leaking, as there is no separation between the receiver and the backfaces of the occluder at the point of contact. Which scheme to choose can be situation dependent. For example, Sousa et al. found using frontfaces for sun shadows and backfaces for interior lights to work best for their applications.

Note that for shadow mapping, objects must be "watertight" (manifold and closed, i.e., solid; Section 16.3.3), or must have both front- and backfaces rendered to the map, else the object may not fully cast a shadow. Woo proposes a general method that attempts to, literally, be a happy medium between using just frontfaces or backfaces for shadowing. The idea is to render solid objects to a shadow map and keep track of the two closest surfaces to the light. This process can be performed by depth peeling or other transparency-related techniques. The average depth between the two objects forms an intermediate layer whose depth is used as a shadow map, sometimes called a *dual shadow map*. If the object is thick enough, self-shadowing and light-leak artifacts are minimized. Bavoil et al. discuss ways to address potential artifacts, along with other implementation details. The main drawbacks are the additional costs associated with using two shadow maps.

As the viewer moves, the light¡Çs view volume often changes size as the set of shadow casters changes. Such changes in turn cause the shadows to shift slightly from frame to frame. This occurs because the light¡Çs shadow map is sampling a different set of directions from the light, and these directions are not aligned with the previous set. For directional lights, the solution is to force each succeeding shadow map generated to maintain the same relative texel beam locations in world space. That is, you can think of the shadow map as imposing a two-dimensional gridded frame of reference on the whole world, with each grid cell representing a pixel sample on the map. As you move, the shadow map is generated for a different set of these same grid cells. In other words, the light¡Çs view projection is forced to this grid to maintain frame to frame coherence.

## Resolution Enhancement

Similar to how textures are used, ideally we want one shadow-map texel to cover about one image pixel. If we have a light source located at the same position as the eye, the shadow map perfectly maps one-to-one with the screen-space pixels (and there are no visible shadows, since the light illuminates exactly what the eye sees). As soon as the light¡Çs direction changes, this per-pixel ratio changes, which can cause artifacts. The shadow is blocky and poorly defined because a large number of pixels in the foreground are associated with each texel of the shadow map. This mismatch is called *perspective aliasing*. Single shadow-map texels can also cover many pixels if a surface is nearly edge-on to the light, but faces the viewer. This problem is known as *projective aliasing*. Blockiness can be decreased by increasing the shadow-map resolution, but at the cost of additional memory and processing.


There is another approach to creating the light¡Çs sampling pattern that makes it more closely resemble the camera¡Çs pattern. This is done by changing the way the scene projects toward the light. Normally we think of a view as being symmetric, with the view vector in the center of the frustum. However, the view direction merely defines a view plane, but not which pixels are sampled. The window defining the frustum can be shifted, skewed, or rotated on this plane, creating a quadrilateral that gives a different mapping of world to view space. The quadrilateral is still sampled at regular intervals, as this is the nature of a linear transform matrix and its use by the GPU. The sampling rate can be modified by varying the light¡Çs view direction and the view window¡Çs bounds.

There are 22 degrees of freedom in mapping the light¡Çs view to the eye¡Çs. Exploration of this solution space led to several different algorithms that attempt to better match the light¡Çs sampling rates to the eye¡Çs. Methods include *perspective shadow maps* (PSM), *trapezoidal shadow maps* (TSM), and *light space perspective shadow maps* (LiSPSM). Techniques in this class are referred to as *perspective warping* methods.

An advantage of these matrix-warping algorithms is that no additional work is needed beyond modifying the light¡Çs matrices. Each method has its own strengths and weaknesses, as each can help match sampling rates for some geometry and lighting situations, while worsening these rates for others. Lloyd et al. analyze the equivalences between PSM, TSM, and LiSPSM, giving an excellent overview of the sampling and aliasing issues with these approaches. These schemes work best when the light¡Çs direction is perpendicular to the view¡Çs direction (e.g., overhead), as the perspective transform can then be shifted to put more samples closer to the eye.

![fig7.17](/images/fig7.17.png)

> For an overhead light, on the left the sampling on the floor does not match the eye¡Çs rate. By changing the light¡Çs view direction and projection window on the right, the sampling rate is biased toward having a higher density of texels nearer the eye.
{: .prompt-info }


<!--
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