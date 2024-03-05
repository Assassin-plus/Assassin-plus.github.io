---
title: "Cloth Simulation | Intro to Physics-Based Animation"
date: 2024-03-04 00:00:00 +0200
categories: [Simulation, GAMES103]
math: true
mermaid: true
toc: true
tip: true
tags: [graphics]     # TAG names should always be lowercase
---

# A Mass-Spring System
We could turn the mesh plane into a mass-spring system to simulate the cloth. 

But first, we need to make raw mesh data into a data structure for simulation.

## Topological Construction

- Raw mesh data: 
  - Vertex List: 3d Vectors; 
  - Triangle List: index triples.
- The key to topological construction is to *sort triangle edge triples*.
  - Each triple contains:
    - edge vertex index 0,
    - edge vertex index 1,
    - triangle index.
    - index 0 < index 1.
  - Sort the triples by the first two indices.
  - Then, we will find the repeated edges close to each other.
  - Remove the repeated edges and get the edge list. (The edge list is a list of pairs of vertex index 0 & 1.)
  - Construct Neighboring triangle list for bending using the repeated edges (triangle index).


## Explicit Integration
### Algorithm

![picture1](</images/截屏2024-03-05 18.01.37.png>)

> notice: the image isn't correct. The force array should be calculated first, and then for every vertex we calculate velocity and position.
{ : .prompt-warning }


## Implicit Integration

# Bending and Locking Issues

# A Co-Rotational Method