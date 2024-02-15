---
title: "Simulation Basics | Physically Based Modeling and Animation Chapter 2"
date: 2024-02-14 00:00:00 +0200
categories: [Simulation, PBM]
math: true
mermaid: true
toc: true
tip: true
tags: [graphics]     # TAG names should always be lowercase
---
## Introduction
> Physically based modeling/animation (PBM) is mainly used to generate *secondary* animation, such as cloth, hair, and environmental effects, which are driven by the motion of the character.
{: .prompt-tip }

> In this book, *dynamic simulation* refers to the continuous-time simulation, in contrast to *discrete event simulation*.
{: .prompt-tip }

## Mathematical Convention
- **Scalar**: lowercase italic, e.g., *t*.
- **Vector**: lowercase bold, e.g., **x**.
- **Unit/Directional Vector**: lowercase bold with a hat, e.g., $\hat{v}$.
- **Matrix**: uppercase, e.g., **A**.
- **Jacobians**: uppercase bold, e.g., **J**.

## Book Structure
- Basics of simulation
- Particle systems
- Rigid body simulation
- Constrained dynamics
- Fluid dynamics

# Basics of Simulation
## Model and Simulation
For dynamic simulation, we use the technique of *numerical integration* to solve the equations of motion. The basic idea is to approximate the continuous-time motion by a sequence of discrete **timesteps**.

The most common numerical integration methods are: Euler Intergration, whose fundamental hypothesis is that during the timestep, all variables are constant.

We can decrease the error by decreasing the timestep, but it increases the computational cost.

# Collision
Collision seems to be instantaneous, compared to the timestep.

When dealing with collision, we need to consider the following three stages. We may solve them simultaneously, yet we need to consider them separately. In different situations, we may use different methods to solve them.
- **Detection**: find out if there is a collision.
- **determination**: find out the collision point, time and normal.
- **response**: find out the effect of the collision.

## Detection
Considering a object colliding with a plane, we can use the normal method to detect the collision. Given object position $\mathbf{x}$ , plane normal $\mathbf{n}$, and a point on the plane $\mathbf{p}$, we can use the following equation to determine if there is a collision.
$$ d = (\mathbf{x} - \mathbf{p}) \cdot \hat{\mathbf{n}} $$

Calculate the distance at any timestep, if the distance changes sign, then there is a collision.

## Determination

If there is a collision in a timestep, we need to determine the **exact time** during the timestep, and the exact collision point. If the simulation uses a Euler integration, we can get the position at the beginning and the end of the timestep, and use **linear interpolation** to get the collision point.

