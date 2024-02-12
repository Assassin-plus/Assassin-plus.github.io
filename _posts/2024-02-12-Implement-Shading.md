---
title: "Implement Shading Model | Real-time Rendering Chapter 5.3"
date: 2024-02-12 00:00:00 +0200
categories: [RTR4,Shading]
math: true
mermaid: true
toc: true
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
