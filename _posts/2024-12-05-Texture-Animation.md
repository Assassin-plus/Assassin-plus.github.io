---
title: "Texture Animation | Real-time Rendering Chapter 6.4"
date: 2024-12-05 00:00:00 +0200
categories: [RTR4,Texture]
math: true
mermaid: true
toc: true
tip: true
tags: [graphics]     # TAG names should always be lowercase
---
# Texture Animation

The image applied to a surface does not have to be static. For example, a video source can be used as a texture that changes from frame to frame.

The texture coordinates need not be static, either. The application designer can explicitly change the texture coordinates from frame to frame, either in the mesh’s data itself or via functions applied in the vertex or pixel shader.