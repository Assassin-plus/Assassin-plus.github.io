---
title: "Aliasing & Antialiasing | Real-time Rendering Chapter 5.4"
date: 2024-02-14 00:00:00 +0200
categories: [RTR4,AA]
math: true
mermaid: true
toc: true
tip: true
tags: [graphics]     # TAG names should always be lowercase
---
# Aliasing & Antialiasing

We will focus on what currently can be done in real time to alleviate aliasing artifacts.

## Sampling and Filtering Theory

sampling theorem:
For a signal to be sampled properly (i.e., so that it is possible to reconstruct the original signal from the samples), the sampling frequency has to be more than twice the maximum frequency of the signal to be sampled.

> It is impossible to entirely avoid aliasing problems when using point samples to render a scene, and we almost always use point sampling.
> {: .prompt-tip }

However, at times it is possible to know when a signal is band-limited. One example is when *a texture is applied to a surface*. It is possible to compute the frequency of the **texture samples** compared to the **sampling rate of the pixel**. If this frequency is lower than the Nyquist limit, then no special action is needed to properly sample the texture. If the frequency is too high, then a variety of algorithms are used to **band-limit the texture**.

### Reconstruction Filters

The sinc function is the perfect reconstruction filter when the sampling frequency is 1.0 (i.e., the maximum frequency of the sampled signal must be smaller than 1/2). This is useful when resampling the signal (next section). However, the filter width of the sinc is **infinite** and is **negative** in some areas, so it is rarely useful in practice. For applications where negative filter values are undesirable or impractical, filters with no negative lobes (often referred to generically as **Gaussian filters**, since they either derive from or resemble a Gaussian curve) are typically used.

After using any filter, a continuous signal is obtained. However, in computer graphics we cannot display continuous signals directly, but we can use them to **resample the continuous signal to another size**, i.e., either enlarging the signal, or diminishing it. This topic is discussed next.

### Resampling

Resampling is used to magnify or minify a sampled signal.

Magnification:
Assume the sampled signal is reconstructed as shown in the previous section. Intuitively, since the signal now is perfectly reconstructed and continuous, all that is needed is to **resample** the reconstructed signal at the desired intervals.

Minification:
The frequency of the original signal is too high for the sampling rate to avoid aliasing. Instead it has been shown that a filter using **sinc(x/a)** should be used to create a continuous signal from the sampled one. After that, resampling at the desired intervals can take place.

![picture 0](/images/2024-02-1_23.26.43.png)

## Screen-Space Antialiasing

Algorithms discussed in this section are screen based, i.e., that they operate only on the output samples of the pipeline.

> There is no one best antialiasing technique, as each has different advantages in terms of quality, ability to capture sharp details or other phenomena, appearance during movement, memory cost, GPU requirements, and speed.
> {: .prompt-tip }

The general strategy of screen-based antialiasing schemes is to use a sampling pattern for the screen and then weight and sum the samples to produce a pixel color.

### Supersampling

- Antialiasing algorithms that compute more than one *full sample per pixel* are called supersampling (or oversampling) methods. (**FSAA/SSAA**)
- A sampling method related to supersampling is based on the idea of the **accumulation buffer**. Instead of one large offscreen buffer, this method uses a buffer that has the same resolution as the desired image, but with more bits of color per channel. *The additional costs of having to re-render the scene a few times per frame and copy the result to the screen makes this algorithm costly for real-time rendering systems.*

  > The accumulation buffer used to be a separate piece of hardware. It was supported directly in the OpenGL API, but was deprecated in version 3.0. On modern GPUs the accumulation buffer concept can be implemented in a pixel shader by using a higher-precision color format for the output buffer.
  > {: .prompt-tip }
  >
- Additional samples are needed when phenomena such as object edges, specular highlights, and sharp shadows cause abrupt color changes.

  > Aliasing of object edges still remains as a major sampling problem. It is possible to use analytical methods, where object edges are detected during rendering and their influence is factored in, but these are often *more expensive and less robust* than simply taking more samples. However, GPU features such as **conservative rasterization** and **rasterizer order views** have opened up new possibilities.
  > {: .prompt-warning }
  >

> Techniques such as supersampling and accumulation buffering work by generating samples that are fully specified with **individually computed** shades and depths. *The overall gains are relatively low and the cost is high*, as each sample has to run through a pixel shader.
> {: .prompt-tip }

### MSAA

- **Multisampling antialiasing (MSAA)** lessens the high computational costs by computing the surface’s shade once per pixel and sharing this result among the samples.
  ![picture 1](/images/2024-02-14_23.35.47.png)

  Pixels may have, say, four (x,y) sample locations per fragment, each with their own color and z-depth, but the pixel shader is evaluated **only once for each object fragment** applied to the pixel.
  If all MSAA positional samples are covered by the fragment, the shading sample is evaluated at the center of the pixel. If instead the fragment covers fewer positional samples, the shading sample’s position can be **shifted** to better represent the positions covered. Doing so avoids shade sampling off the edge of a texture, for example. This position adjustment is called **centroid sampling** or centroid interpolation and is done `<u>`automatically by the GPU`</u>`, if enabled.
  Centroid sampling avoids off-triangle problems but can cause **derivative computations** to return incorrect values.

MSAA is faster than a pure supersampling scheme because the fragment is shaded only once. It focuses effort on sampling the fragment’s pixel **coverage** at a higher rate and **sharing** the computed shade. It is possible to save more memory by further de- coupling sampling and coverage, which in turn can make antialiasing faster still—the less memory touched, the quicker the render.

- EQAA (Enhanced Quality Antialiasing) is a technique that uses a combination of MSAA and a programmable resolve filter to improve the quality of the final image. It is a feature of some AMD GPUs.

  > EQAA’s “2f4x” mode stores two color and depth values, shared among four sample locations. The colors and depths are no longer stored for particular locations but rather saved in a table. Each of the four samples then needs just one bit to specify which of the two stored values is associated with its location. The coverage samples specify the contribution of each frag- ment to the final pixel color.
  > {: .prompt-tip }
  > If the number of colors stored is exceeded, a stored color is evicted and its samples are marked as unknown. These samples do not contribute to the final color. For most scenes there are relatively few pixels containing three or more visible opaque fragments that are radically different in shade, so this scheme performs well in practice.
  > {: .prompt-tip }
  >
- Once all geometry has been rendered to a multiple-sample buffer, a resolve operation is then performed. This procedure *averages the sample colors* together to determine the color for the pixel. It is worth noting that a problem can arise when using multisampling with **high dynamic range** color values. In such cases, to avoid artifacts you normally need to **tone-map** the values before the resolve.
- By default, MSAA is resolved with a **box** filter. On modern GPUs pixel or compute shaders can access the MSAA samples and use whatever reconstruction filter is desired, including one that samples from the surrounding pixels’ samples. A wider filter can reduce aliasing, though at the loss of sharp details.

  > the cubic smoothstep and B-spline filters with a filter width of **2 or 3 pixels** gave the best results overall. There is also a performance cost, as even emulating the default box filter resolve will take longer with a **custom shader**, and a wider filter kernel means increased sample access costs.
  > {: .prompt-tip }
  >

### Temporal Antialiasing

A single image is generated, possibly with MSAA or another method, and the previous images are blended in. Usually just two to four frames are used.

Older images may be given exponentially less weight, though this can have the effect of the frame **shimmering** if the viewer and scene do not move, so often equal weighting of just the last and current frame is done.

With each frame’s samples in a different subpixel location, the weighted sum of these samples gives a better coverage estimate of the **edge** than a single frame does.

**No** additional samples are needed for each frame, which is what makes this type of approach so appealing. It is even possible to use temporal sampling to allow generation of a **lower-resolution** image that is upscaled to the display’s resolution. In addition, illumination methods or other techniques that require many samples for a good result can instead use **fewer samples** each frame, since the results will be blended over several frames.

- Rapidly moving objects or quick camera moves can cause **ghosting**.
  - One solution to ghosting is to perform such antialiasing on only slow-moving objects.
  - Another important approach is to use reprojection (Section 12.2) to better correlate the previous and current frames’ objects.

  > In such schemes, objects generate motion vectors that are stored in a separate “velocity buffer” (Section 12.5). These vectors are used to **correlate the previous frame with the current one**, i.e., the vector is subtracted from the current pixel location to find the previous frame’s color pixel for that object’s surface location. Samples unlikely to be part of the surface in the current frame are discarded.
  > {: .prompt-tip }
  >

Because *no extra samples*, and so relatively *little extra work*, are needed for temporal antialiasing, there has been a strong interest and wider adoption of this type of algorithm in recent years. Some of this attention has been because **deferred shading techniques** (Section 20.1) are not compatible with MSAA and other multisampling support.

## Sample Patterns

Principle:
humans are most disturbed by aliasing on **near- horizontal and near-vertical edges**. Edges with near *45 degrees* slope are next most disturbing.

### Rotated grid supersampling

Rotated grid supersampling (RGSS) uses a rotated square pattern to give more vertical and horizontal resolution within the pixel.

The RGSS pattern is a form of **Latin hypercube** or **N-rooks sampling**, in which n samples are placed in an n×n grid, with one sample per row and column.

N-rooks is a start at creating a good sampling pattern, but it is not sufficient. For example, the samples could all be places along the diagonal of a subpixel grid and so give a poor result for edges that are nearly parallel to this diagonal.

> For better sampling we want to avoid putting two samples near each other. We also want a **uniform** distribution, spreading samples evenly over the area. To form such patterns, stratified sampling techniques such as **Latin hypercube sampling** are combined with other methods such as *jittering, Halton sequences, and Poisson disk sampling*.
> {: .prompt-tip }

In practice GPU manufacturers usually hard-wire such sampling patterns into their hardware for multisampling antialiasing.
![picture 1](/images/2024-02-15_13.47.27.png)

a basic Halton sequence works better than any MSAA pattern provided by the GPU. **A Halton sequence** generates samples in space that appear random but have low discrepancy, that is, they are well distributed over the space and none are clustered.

### Moire Fringes

A scene can be made of objects that are arbitrarily small on the screen, meaning that no sampling rate can ever perfectly capture them. If these tiny objects or features form a pattern, sampling at constant intervals can result in **Moir ́e fringes** and other interference patterns.One solution is to use **stochastic sampling**, which gives a more randomized pattern.

A pattern with less structure helps, but it can still exhibit aliasing when repeated pixel to pixel. One solution is use a different sampling pattern at each pixel, or to change each sampling location over time.

A few other GPU-supported algorithms are worth noting.

- One real-time antialiasing scheme that lets samples affect more than one pixel is NVIDIA’s older **Quincunx** method. Quincunx multisampling antialiasing uses this pattern, putting the four outer samples at the corners of the pixel. Each corner sample value is distributed to its four neighboring pixels.
- Instead of weighting each sample equally (as most other real-time schemes do), the center sample is given a weight of 1/2 , and each corner sample has a weight of 1/8 . Because of this sharing, an average of only two samples are needed per pixel, and the results are considerably better than two-sample FSAA methods. This pattern approximates a two-dimensional tent filter.
- Quincunx sampling can also be applied to **temporal antialiasing** by using a single sample per pixel. Each frame is offset half a pixel in each axis from the frame before, with the offset direction alternating between frames. *The previous frame provides the pixel corner samples*, and bilinear interpolation is used to rapidly compute the contribution per pixel. The result is averaged with the current frame. Equal weighting of each frame means there are **no shimmer artifacts for a static view**. The issue of aligning moving objects is still present, but the scheme itself is simple to code and gives a much better look while using only one sample per pixel per frame.

---

When used in a single frame, Quincunx has a low cost of only two samples by sharing samples at the pixel boundaries. The RGSS pattern is better at capturing more gradations of nearly horizontal and vertical edges. First developed for mobile graphics, the **FLIPQUAD** pattern combines both of these desirable features. Its advantages are that the cost is only two samples per pixel, and the quality is similar to RGSS (which costs four samples per pixel).

![picture 1](/images/2024-02-15_14.40.28.png){: .normal }

> \#TODO *(Further Reading)* Like Quincunx, the two-sample FLIPQUAD pattern can also be used with temporal antialiasing and spread over two frames. Drobot tackles the question of which two-sample pattern is best in his hybrid reconstruction antialiasing (HRAA) work. He explores different sampling patterns for temporal antialiasing, finding the FLIPQUAD pattern to be the best of the five tested. A checkerboard pattern has also seen use with temporal antialiasing. El Mansouri  discusses using twosample MSAA to create a checkerboard render to reduce shader costs while addressing aliasing issues. Jimenez uses SMAA, temporal antialiasing, and a variety of other techniques to provide a solution where antialiasing quality can be changed in response to rendering engine load. Carpentier and Ishiyama sample on edges, rotating the sampling grid by 45◦. They combine this temporal antialiasing scheme with FXAA (discussed later) to efficiently render on higher-resolution displays.

## Morphological Methods

Aliasing often results from edges, such as those formed by geometry, sharp shadows, or bright highlights. The knowledge that aliasing has a structure associated with it can be exploited to give a better antialiased result.

This form of antialiasing is performed as a post-process. That is, rendering is done in the usual fashion, then the results are fed to a process that generates the antialiased result.

Those that rely on additional buffers such as depths and normals can provide better results, such as subpixel reconstruction antialiasing (SRAA), but are then applicable for antialiasing only geometric edges. Analytical approaches, such as *geometry buffer antialiasing* (GBAA) and *distance-to-edge antialiasing* (DEAA), have the renderer compute additional information about where triangle edges are located, e.g., how far the edge is from the center of the pixel.

- The most general schemes need only the color buffer, meaning they can also im- prove edges from shadows, highlights, or various previously applied post-processing techniques, such as silhouette edge rendering (Section 15.2.3).
- More elaborate forms of edge detection attempt to *find pixels likely to contain an edge* at any angle and determine its coverage. The neighborhoods around potential edges are examined, with the goal of **reconstructing** as possible where the original edge was located. The edge’s effect on the pixel can then be used to **blend** in neighboring pixels’ colors.

> edge prediction and blending can give a higher- precision result than sample-based algorithms. For example, a technique that uses four samples per pixel can give only five levels of blending for an object’s edge: no samples covered, one covered, two, three, and four. The estimated edge location can have more locations and so provide better results.
> {: .prompt-tip }

---

> There are several ways image-based algorithms can go astray.
> {: .prompt-warning}

- First, the edge may not be detected if the color difference between two objects is lower than the algorithm’s threshold.
- Pixels where there are three or more distinct surfaces overlapping are difficult to interpret.
- Surfaces with high-contrast or high-frequency elements, where the color is changing rapidly from pixel to pixel, can cause algorithms to miss edges.
- In particular, text quality usually suffers when morphological antialiasing is applied to it.
- Object corners can be a challenge, with some algorithms giving them a rounded appearance.
- Curved lines can also be adversely affected by the assumption that edges are straight.
- A single pixel change can cause a large shift in how the edge is reconstructed, which can create noticeable artifacts frame to frame.
- One approach to ameliorate this problem is to use MSAA coverage masks to improve edge determination.

Morphological antialiasing schemes use only the information that is provided. For example, an object thinner than a pixel in width, such as an electrical wire or rope, will have gaps on the screen wherever it does not happen to cover the center location of a pixel.

Taking more samples can improve the quality in such situations; image-based antialiasing alone cannot. In addition, execution time can be variable depending on what content is viewed.

---

image-based methods can provide antialiasing support for modest memory and processing costs, so they are used in many applications. The color-only versions are also **decoupled** from the rendering pipeline, making them easy to modify or disable, and can even be exposed as *GPU driver options*.

- The two most popular algorithms are **fast approximate antialiasing** (FXAA), and **subpixel morphological antialiasing** (SMAA), in part because both provide solid (and free) source code implementations for a variety of machines.
- Both algorithms use color-only input, with SMAA having the advantage of being able to access MSAA samples. Each has its own variety of settings available, trading off between speed and quality.
- Costs are generally in the range of 1 to 2 milliseconds per frame, mainly because that is what video games are willing to spend.
- Finally, both algorithms can also take advantage of temporal antialiasing.

\#TODO Further Reading: Reshetov, Alexander, and Jorge Jimenez, “MLAA from 2009 to 2017,” *High-Performance Graphics research impact retrospective*, July 2017.
