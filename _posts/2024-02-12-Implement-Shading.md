---
title: "Implement Shading Model | Real-time Rendering Chapter 5.3"
date: 2024-02-12 00:00:00 +0200
categories: [RTR4,Shading]
math: true
mermaid: true
toc: true
tip: true
tags: [graphics]     # TAG names should always be lowercase
---
# Implement Shading Models
In this section we will go over some key considerations for *designing and writing* implementations of shading and lighting equations. 

## Frequency of Evaluation

- determine whether the result of a given computation is always **constant over an entire draw call**. In this case, the computation can be performed by the application, typically on the CPU, though a GPU compute shader could be used for especially costly computations.
- once ever: there is a broad range of possible frequencies of evaluation
  - a **constant subexpression** in the shading equation, but this could apply to any computation based on rarely changing factors such as the **hardware configuration** and **installation options**
  >Such shading computations might be resolved when the shader is compiled, in which case there is no need to even set a uniform shader input. Alternatively, the compu- tation might be performed in an offline precomputation pass, at installation time, or when the application is loaded.
  - result of a shading computation changes over an application run, but so **slowly** that updating it every frame is *not necessary*
  > lighting factors that depend on the time of day in a virtual game world. If the computation is costly, it may be worthwhile to amortize it over multiple frames.
  - computations that are performed once per frame, such as **concatenating the view** and **perspective matrices**; or once per model, such as **updating model lighting parameters** that depend on location; or once per draw call, e.g., updating parameters for **each material within a model**. 
  > Grouping uniform shader inputs by frequency of evaluation is useful for application efficiency, and can also help GPU performance by minimizing constant updates
- If the result of a shading computation changes within a draw call, it cannot be passed to the shader through a uniform shader input. Instead, it must be computed by one of the **programmable shader stages**, each one corresponding to a different evaluation frequency:
  - **Vertex shader**: Evaluation per pre-tessellation vertex.
  - **Huill shader**: Evaluation per surface patch.
  - **Domain shader**: Evaluation per post-tessellation vertex.
  - **Geometry shader**: Evaluation per primitive.
  - **Pixel shader**: Evaluation per pixel.
- For per-pixel and per-vertex shading on models with a wide range of **vertex densities**, these may cause different errors. It's because that parts of the shading equation, *the highlight in particular*, have values that vary nonlinearly over the mesh surface. This makes them a poor fit for the vertex shader, the results of which are **interpolated linearly** over the triangle before being fed to the pixel shader.
> **In principle**, it would be possible to compute only the specular highlight part of the
shading model in the pixel shader, and calculate the rest in the vertex shader. This would likely not result in visual artifacts and in theory would save some computation. 
> **In practice**, this kind of hybrid implementation is often not optimal. The linearly varying parts of the shading model tend to be the least computationally costly, and splitting up the shading computation in this way tends to **add enough overhead**, such as duplicated computations and additional varying inputs, to outweigh any benefit.
- Implementations often normalize interpolated vectors before and after interpolation, i.e., in **both** the vertex and pixel shaders. Unlike the surface normals, vectors that point *toward specific locations*, such as the view vector and the light vector for punctual lights, are typically **not interpolated**. Instead, the interpolated surface position is used to compute these vectors in the pixel shader. 
![picture 0](</images/截屏2024-02-13 13.19.37.png>)
- The vertex shader transforms the surface geometry into “the appropriate coordinate system.” For example, if rendered scenes are expected to include *huge numbers of lights*, **world space** might be chosen to avoid transforming the light positions. Alternately, **camera space** might be preferred, to better optimize *pixel shader operations* relating to the view vector and to possibly improve precision (Section 16.6).
> some applications choose the faceted appearance of **per-primitive shading evaluation** for stylistic reasons. This style is often referred to as flat shading. In principle, flat shading could be performed in the *geometry shader*, but recent implementations typically use the **vertex shader**. 
> This is done by associating each primitive’s properties with its first vertex and disabling vertex value interpolation. Disabling interpolation (which can be done for each vertex value separately) causes the value from the first vertex to be passed to all pixels in the primitive.
## Implementation Examples
- In most typical rendering applications, varying values for material properties such as $c_{surface}$ would be stored in **vertex data** or, more commonly, in **textures** (Chapter 6).
> This implementation will use the shader’s *dynamic branching capabilities* to loop over all light sources. While this straightforward approach can work well for reasonably simple scenes, it does not scale well to large and geometrically complex scenes with many light sources. Rendering techniques to efficiently *handle large light counts* will be covered in Chapter 20.
- Shading models are not implemented in isolation, but in the context of a larger **rendering framework**.
- Before the shader code proper, the shader source includes definitions of the shader inputs and outputs.
> Since these are point lights, the definition for each one includes a position and a color. These are defined as vec4 instead of vec3 to conform to the restrictions of the GLSL std140 data layout standard. Although, as in this case, the std140 layout can lead to some wasted space, it simplifies the task of ensuring consistent data layout between CPU and GPU, which is why we use it in this sample. 
> The array of Light structs is defined inside a named uniform block, which is a GLSL feature for binding a group of uniform variables to a buffer object for faster data transfer. The array length is defined to be equal to the maximum number of lights that the application allows in a single draw call. As we will see later, the application replaces the MAXLIGHTS string in the shader source with the correct value (10 in this case) before shader compilation. The uniform integer uLightCount is the actual number of active lights in the draw call.
{: .prompt-tip}

> An example showing the `tip` type prompt.
{: .prompt-tip }