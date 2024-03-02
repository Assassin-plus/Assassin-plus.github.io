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
- [Basics of Simulation](#basics-of-simulation)
  - [Model and Simulation](#model-and-simulation)
- [Collision](#collision)
  - [Detection](#detection)
  - [Determination](#determination)
  - [Response](#response)
- [Implementation](#implementation)
  - [Numerical Error](#numerical-error)
    - [Tolerance](#tolerance)
    - [Limitation of Tolerance](#limitation-of-tolerance)
  - [Motionless condition](#motionless-condition)
- [Collision between Polygon and Point](#collision-between-polygon-and-point)


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

If there is a collision in a timestep, we need to determine the **exact time** during the timestep. If the simulation uses a Euler integration, we can get the position at the beginning and the end of the timestep, and use **linear interpolation** to get the collision time.

We can get the distance d. Given the distance at the beginning of the timestep $d^{[n]}$ and the end of the timestep $d^{[n+1]}$, we can use the following equation to get the collision time.
$$ f = \frac{d^{[n]}}{d^{[n]} - d^{[n+1]}} $$
which is the proportion of the timestep.

In order to get the collision point, we can re-calculate the integration using the exact time $fh$. Then we can get the collision point on the plane. Afterwaard, we need to get the collision response and integrate the remaining timestep.

## Response

After detecting the collision, we need to calculate the response. Suppose we have already got the response, how can we alter the simulation cycle to deal with the response?

```
// h is the timestep, n is the iteration number, t is the current time
//s is the state of the object (position, velocity, etc.)
s = s0;n = 0;t = 0;
while (t < tmax) do //invirance: s is the state at t
{
    //output the n'th state
    TimestepRemaining = h;
    Timestep = TimestepRemaining;   // simulate the entire timestep
    while (TimestepRemaining > 0) do
    {
        s_dot = GetDerivative(s);  //get the acceleration
        s_new = Integrate(s, s_dot, Timestep); 
        if CollisionBetween(s, s_new) then
        {
            //calculate the first collision and re-integrate
            Calculate f;
            Timestep = f * Timestep;
            s_new = Integrate(s, s_dot, Timestep);
            s_new = CollisionResponse(s, s_new);
        }
        TimestepRemaining = TimestepRemaining - Timestep;
        s = s_new;
    }
    n = n + 1;
    t = n * h;
}
```

- If there is no collision, the whole process is the same as the Euler integration. After the if, the inside of the while loop only runs once.
- If there is a collision, we need to re-integrate the time, and the new state will be exactly at the time of the collision. Because we haven't integrated the entire timestep, we need to update $TimestepRemaining$ to the remaining time. Also we may encounter multiple collisions in a single timestep, we need to iterate until the entire original timestep is integrated.
- Sometimes, animators would rather use a fixed timestep, which is essential for real-time applications or synchronization with multiple simulation threads. In this case, animators will sacrifice the accuracy of the simulation which will be discussed in Chapter 4.4.3
> When $Timestep$ is small enough to get its floating point representation 0, we will encounter a dead loop. A common solution is to change the condition to $TimestepRemaining > \epsilon$, and we will discuss this further in Chapter 3.3.1.
{: .prompt-tip }

As for the friction and collision elasticity, just model them physically and we will skip the details here.

# Implementation
## Numerical Error

- **Round-off Error**: caused by the finite precision of floating point numbers.
- **Integration Error**: caused by the numerical integration method.
- **Discretization Error**: caused by the discretization of the geometry, such as the mesh representation, and voxelization.

### Tolerance

Tolerance is a common method to deal with the numerical error. Anything inside the range of the tolerance is considered to be equal to the true value. 

If there is no tolerance, comparing two numbers would be $a-b=0$, while with tolerance, it would be $|a-b|<\epsilon$.
Also, when we want to check $a>b$, we can use $a-b<\epsilon$, and for $a<b$, we can use $a-b<-\epsilon$.

### Limitation of Tolerance

First, we cannot set a "good" tolerance for **all situations**. We can never find a tolerance that is small enough to avoid the equal error, yet large enough to catch every numerical error.

Second is the ***incidence intransitivity***. If we have $a \approx b$ and $b \approx c$, we cannot guarantee that $a \approx c$. In order to avoid this, we need to examine the hypothesis of all algorithms, and ensure that we didn't assume the transitivity.

When the tolerance cannot solve the numerical error, we need to analyze where the error comes from, and how it propagates. Then we find out which steps of the algorithm are sensitive to numerical accurancy. Usually, the condition detection is the most important step, cause these would induce evident difference in the result.

> Substitute method includes better numerical integration methods, higher precision floating point numbers, re-construction of calculation or formula, exact calculate predicates, and different discretization methods.
{: .prompt-tip }

## Motionless condition

```
if ||v|| < eps1 then
if exist plane p such that d(p) < eps2 then
if F dot n < eps3 then
if ||F_t|| < u_s * ||F_n|| then
    Motionless = true;
else
    Motionless = false;
```

The logic above describes how to determine if an object is motionless. The first condition is to check if the velocity is small enough. The second condition is to check if the object is on a plane, to avoid the situation that the object is at its highest point and reaches 0 velocity. The third condition is to check if there is a contact force to overcome the gravity. The last condition is to check if the friction force is enough to stop the object.

> Sometimes we need to judge if the object isn't bouncing but is sliding or rolling, which is to say, motionless on the normal direction but not on the tangent direction. We can just consider $||v_n||$ instead of $||v||$ and ignore friction force condition.
{: .prompt-tip }

> In real scene simulation, the detection of motionless and whether they're balanced is NP-hard. Several approximation methods will be discussed in Chapter 9.
{: .prompt-warning }


# Collision between Polygon and Point






