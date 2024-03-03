---
title: "Rigid Body Dynamics| Intro to Physics-Based Animation"
date: 2024-03-03 00:00:00 +0200
categories: [Simulation, GAMES103]
math: true
mermaid: true
toc: true
tip: true
tags: [graphics]     # TAG names should always be lowercase
---

# Translational Motion

$$
\mathbf{v}(t^{[1]})=\mathbf{v}(t^{[0]})+\mathbf{M}^{-1}\int_{t^{[0]}}^{t^{[1]}}\mathbf{f}(\mathbf{x},\mathbf{v},t)dt
$$

$$
\mathbf{x}(t^{[1]})=\mathbf{x}(t^{[0]})+\int_{t^{[0]}}^{t^{[1]}}\mathbf{v}(ßt)dt
$$

## Integration Methods

### Explicit Euler
1st-order accurate, simple, and easy to implement.
$$
\int_{t^{[0]}}^{t^{[1]}}\mathbf{v}(t)dt \approx \mathbf{v}(t^{[0]})\Delta t
$$

### Implicit Euler
omit.

### Midpoint Method
2nd-order accurate, but requires the evaluation of the force at the midpoint.
$$
\int_{t^{[0]}}^{t^{[1]}}\mathbf{v}(t)dt \approx \Delta t \mathbf{v}(t^{[0.5]})
$$

## Semi-Implicit

$$ \mathbf{v}^{[1]} = \mathbf{v}^{[0]} + \mathbf{M}^{-1}\mathbf{f}^{[0]}\Delta t $$

$$ \mathbf{x}^{[1]} = \mathbf{x}^{[0]} + \mathbf{v}^{[1]}\Delta t $$

## Leapfrog method

$$ \mathbf{v}^{[0.5]} = \mathbf{v}^{[-0.5]} + \mathbf{M}^{-1}\mathbf{f}^{[0]}\Delta t $$

$$ \mathbf{x}^{[1]} = \mathbf{x}^{[0]} + \mathbf{v}^{[0.5]}\Delta t $$

# Types of Forces

## Drag Force

$$ \mathbf{f}_{\text{drag}} = -\sigma \mathbf{v} $$

Since the drag force is proportional to the velocity, a simple alternative is to use a damping coefficient.

$$ \mathbf{v}^{[1]} = \alpha \mathbf{v}^{[0]} $$

# Rotation

## Represented by Matrix

> pros: friendly for applying rotation to each vertex by matrix-vector multiplication.
> cons: not friendly for dynamics.
> * Redundancy: 9 elements, but only 3 DoFs.
> * Non-intuitive: hard to understand the physical meaning of each element.
> * Time-derivative: hard to compute the time-derivative (rotational velocity) of the matrix.
{: .prompt-tip }

## Represented by Euler Angles

> pros: intuitive, easy to understand. Each axial rotation uses an angle.
> cons: not friendly for dynamics.
> * gimbal lock: lose DoFs.
> * Time-derivative: hard to compute the time-derivative (rotational velocity) of the matrix.
{: .prompt-tip }

## Represented by Quaternion

To represent a rotation around $\mathbf{v}$ by an angle $\theta$, we use the quaternion:

$$ \mathbf{q} = [\cos(\theta/2) + \mathbf{v}] $$

$$ ||\mathbf{q}|| = 1 \ or \ ||\mathbf{v}||^2 = \sin ^2\frac{\theta}{2}$$

# Rotational Motion

$$
\mathbf{\omega}^{[1]} = \mathbf{\omega}^{[0]} + \Delta t (\mathbf{I^{[0]}}^{-1})\mathbf{\tau}^{[0]}
$$

$$
\mathbf{q}^{[1]} = \mathbf{q}^{[0]} + [0 , \frac{\Delta t}{2}\mathbf{\omega}^{[0]}]  \times \mathbf{q}^{[0]}
$$

## Torque and Inertia

torque: $\mathbf{\tau} = \mathbf{I}\mathbf{\omega} = (\mathbf{R}\mathbf{r}_i) \times \mathbf{f}_i $

inertia: $\mathbf{I}_{ref} = m_i(\mathbf{r}_i^T \mathbf{r}_i\mathbf{1} -  \mathbf{r}_i\mathbf{r}_i^T),\ \mathbf{I} = \mathbf{RI_{ref}R^T}$

# list of variables

* $\mathbf{v}$: velocity
* $\mathbf{x}$: position (transform.position in Unity)
* $\mathbf{\omega}$: angular velocity
* $\mathbf{q}$: quaternion (transform.rotation in Unity)
* $\mathbf{M}$: mass
* $\mathbf{I}$: inertia tensor
* $\mathbf{f}$: force
* $\mathbf{\tau}$: torque


