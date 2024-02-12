---
title: "Light Sources | Real-time Rendering Chapter 5.2"
date: 2024-02-12 00:00:00 +0200
categories: [RTR4,Shading]
math: true
mermaid: true
toc: true
tags: [graphics]     # TAG names should always be lowercase
---
# Light Sources
- **Punctual Light**: A light source that is infinitely small and emits light in all directions.
## Point Light / Omni Light
- A point light is a light source that emits light in all directions evenly from a single point in space.
- The intensity of the light decreases with the square of the distance from the light source.

  $$c_{light} = c_{light0}(\frac{r_0}{r})^2$$

- There're two problems when r approaches 0 and infinity.
- $c_{light}$ goes to infinity when $r \rightarrow0$ 

  $$c_{light} = c_{light0}\frac{r_0^2}{r^2+\epsilon},\epsilon = 1cm^2 \ in\ Unreal\ Engine$$

$$c_{light} = c_{light0}(\frac{r_0}{max(r,r_{min})})^2,\ in\ CryEngine$$

- $c_{light}\rightarrow 0 $ when r is large enough yet needs calculation. So we need this to become 0 at $r_{max}$ and its derivative at the same time.

$$f_{win}(r)=(1-(\frac{r}{r_{max}})^4)^{+2}$$

![picture 0](/images/dd7f0ee9ea6bd1406f4019a06c3a5b37567f251b9558ac38ccf9ed04c2f4781b.png)  

> Application requirements will affect the choice of method used. For example, having the derivative equal to 0 at $r_{max}$ is particularly important when the distance attenuation function is sampled at a relatively low spatial frequency (e.g., in light maps or per-vertex). CryEngine does not use light maps or vertex lighting, so it employs a simpler adjustment, switching to linear falloff in the range between 0.8$r_{max}$ and $r_{max}$ 

For some applications, matching the inverse-square curve is not a priority, so some other function entirely is used.

$$c_{light}(r)=c_{light0}f_{dist}(r)$$

> In some cases, the use of non-inverse-square falloff functions is driven by performance constraints. For example, the game Just Cause 2 needed lights that were extremely inexpensive to compute. This dictated a falloff function that was *simple to compute, while also being smooth enough to avoid per-vertex lighting artifacts*
$$f_{dist}(r)=(1-(\frac{r}{r_{max}})^2)^{+2}$$

> For example, the Unreal Engine, used for both realistic and stylized games, has two modes for light falloff: an **inverse-square mode**, and an **exponential falloff mode** that can be tweaked to create a variety of attenuation curves. The developers of the game Tomb Raider (2013) used **spline-editing tools** to author falloff curves, allowing for even greater control over the curve shape.

## Spotlights

![picture 1](/images/theta.png){: width="972" height="589" }

For example, the function $f_{dirF} (l)$ is used in the Frostbite game engine , and the function $f_{dirT}(l)$ is used in the three.js browser graphics library :

$$t = (\frac{\cos\theta_s-\cos\theta_u}{\cos\theta_p-cos\theta_u})^\plusmn$$

$$f_{dirF} (l) = t^2$$

$$f_{dirT}(l)=smoothstep(t)=t^2(3-2t)$$

## Other Light Types
Different types of lights can be defined by using other methods to compute the light direction. For example, in addition to the light types mentioned earlier, Tomb Raider also has **capsule lights** that use a line segment as the source instead of a point . For each shaded pixel, the *direction to the closest point on the line segment* is used as the light direction l.

In reality, light sources have size and shape, and they illuminate surface points from **multiple directions**. In rendering, such lights are called *area lights*, and their use in real-time appli- cations is steadily increasing. Area-light rendering techniques fall into two cate- gories: 
- those that simulate the softening of shadow edges that results from the area light being partially occluded (Section 7.1.2)
-  those that simulate the effect of the area light on surface shading (Section 10.1). 
> This second category of lighting is most noticeable for smooth, mirror-like surfaces, where the light’s shape and size can be clearly discerned in its reflection.

