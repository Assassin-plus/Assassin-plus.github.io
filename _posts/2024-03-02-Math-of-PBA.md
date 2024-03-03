---
title: "Math Background | Intro to Physics-Based Animation"
date: 2024-03-02 00:00:00 +0200
categories: [Simulation, GAMES103]
math: true
mermaid: true
toc: true
tip: true
tags: [graphics]     # TAG names should always be lowercase
---

<head>
    <script src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML" type="text/javascript"></script>
    <script type="text/x-mathjax-config">
        MathJax.Hub.Config({
            tex2jax: {
            skipTags: ['script', 'noscript', 'style', 'textarea', 'pre'],
            inlineMath: [['$','$']]
            }
        });
    </script>
</head>


# Symmetric Positive Difiniteness (SPD)
**A** is s.p.d. if and only if all of its eigenvalues are positive.

In practice, eigenvalue decomposition is time-consuming. We choose diagonally dominant matrices, which are easier to check.

Finally, a s.p.d. matrix is always invertible.

# Direct Linear Solver

LU decomposition, Cholesky decomposition, and QR decomposition are the most common direct linear solvers.

When **A** is sparse, **L** and **U** are not so sparse.
> Library: Intel MKL PARDISO
{: .prompt-tip }

# Iterative Linear Solver

An iterative solver has the form:
$$
\mathbf{x}^{(k+1)} = \mathbf{x}^{(k)} + \alpha M^{-1}(\mathbf{b} - A\mathbf{x}^{(k)})
$$
where $\alpha$ is the srelaxation, and $M$ is the iterative matrix.

$M$ must be easier to solve:
- Diagonal matrix: Jacobi method
- Lower/Upper triangular matrix: Gauss-Seidel method

> Simple, Fast for inexact solution and easy to parallelize.
{: .prompt-tip }

> Convergence condition and slow for exact solution.
{: .prompt-tip }

# Tensor Calculus

$$
\frac{\partial f}{\partial \mathbf{x}} = \begin{bmatrix}
\frac{\partial f}{\partial x} 
\frac{\partial f}{\partial y} 
\frac{\partial f}{\partial z}
\end{bmatrix}
$$
or
$$
\nabla f (\mathbf{x}) = \begin{bmatrix}
\frac{\partial f}{\partial x} \\
\frac{\partial f}{\partial y} \\
\frac{\partial f}{\partial z}
\end{bmatrix}
$$


if $\mathbf{f}(\mathbf{x}) = \begin{bmatrix} f(\mathbf{x}) \\ g(\mathbf{x}) \\ h(\mathbf{x}) \end{bmatrix}\in \mathbb{R}^3$, then

$$Jacoobian:\  
\mathbf{J}(\mathbf{f}) = \begin{bmatrix}
\frac{\partial f}{\partial x} & \frac{\partial f}{\partial y} & \frac{\partial f}{\partial z} \\
\frac{\partial g}{\partial x} & \frac{\partial g}{\partial y} & \frac{\partial g}{\partial z} \\
\frac{\partial h}{\partial x} & \frac{\partial h}{\partial y} & \frac{\partial h}{\partial z}
\end{bmatrix}
$$

$$Divergence:\ 
\nabla  \mathbf{f} = \frac{\partial f}{\partial x} + \frac{\partial g}{\partial {y}} + \frac{\partial{h}}{\partial {z}}
$$

$$
Curl:\ 
\nabla \times \mathbf{f} = \begin{bmatrix}
\frac{\partial h}{\partial y} - \frac{\partial g}{\partial z} \\
\frac{\partial f}{\partial z} - \frac{\partial h}{\partial x} \\
\frac{\partial g}{\partial x} - \frac{\partial f}{\partial y}
\end{bmatrix}
$$

if $f(\mathbf{x}) \in \mathbb{R}$, then:

$$
Hessian:\ 
\mathbf{H} = \mathbf{J}(\nabla f(\mathbf{x})) = \begin{bmatrix}
\frac{\partial^2 f}{\partial x^2} & \frac{\partial^2 f}{\partial x \partial y} & \frac{\partial^2 f}{\partial x \partial z} \\
\frac{\partial^2 f}{\partial y \partial x} & \frac{\partial^2 f}{\partial y^2} & \frac{\partial^2 f}{\partial y \partial z} \\
\frac{\partial^2 f}{\partial z \partial x} & \frac{\partial^2 f}{\partial z \partial y} & \frac{\partial^2 f}{\partial z^2}
\end{bmatrix}
$$

$$
Laplacian:\ 
\nabla^2 f = \nabla \cdot \nabla f = \frac{\partial^2 f}{\partial x^2} + \frac{\partial^2 f}{\partial y^2} + \frac{\partial^2 f}{\partial z^2}
$$

## Example: A Spring
Energy: $E(\mathbf{x}) = \frac{1}{2}k(\mathbf{x} - \mathbf{x}_0)^T(\mathbf{x} - \mathbf{x}_0)$

Force: $ \mathbf{f}(\mathbf{x}) = -\nabla E(\mathbf{x}) = -k(\mathbf{x} - \mathbf{x}_0)\frac{\mathbf{x}}{\vert \vert\mathbf{x}\vert \vert} $

Tangent stiffness:$\mathbf{H(x)} = -\frac{\partial \mathbf{f}}{\partial \mathbf{x}} = k(\mathbf{I} - \frac{\mathbf{x}\mathbf{x}^T}{\vert \vert\mathbf{x}\vert \vert^2})(1-\frac{L}{\vert \vert\mathbf{x}\vert \vert}) + k\frac{\mathbf{x}\mathbf{x}^T}{\vert \vert\mathbf{x}\vert \vert}$

## Example: A Spring with Two Ends

Energy: $E(\mathbf{x}) = \frac{1}{2}k(\vert \vert\mathbf{x_{01}}\vert \vert-L)^2$

Force: $\mathbf{f}(\mathbf{x}) = -\nabla E(\mathbf{x}) = \begin{bmatrix}-\nabla_0E(\mathbf{x}) \\  -\nabla_1E(\mathbf{x})\end{bmatrix} = \begin{bmatrix}\mathbf{f}_e \\ -\mathbf{f}_e\end{bmatrix},\mathbf{f}_e = -k(\vert \vert\mathbf{x_{01}}\vert \vert-L)\frac{\mathbf{x_{01}}}{\vert \vert\mathbf{x_{01}}\vert \vert}
$

Tangent stiffness:
$
\mathbf{H(x)} = \begin{bmatrix} 
\frac{\partial^2E}{\partial \mathbf{x_0}^2} & \frac{\partial^2E}{\partial \mathbf{x_0}\partial \mathbf{x_1}} \\
\frac{\partial^2E}{\partial \mathbf{x_1}\partial \mathbf{x_0}} & \frac{\partial^2E}{\partial \mathbf{x_1}^2}
\end{bmatrix} = \begin{bmatrix}\mathbf{H}_e & -\mathbf{H}_e \\ -\mathbf{H}_e & \mathbf{H}_e\end{bmatrix}
,
\mathbf{H}_e = k(\mathbf{I} - \frac{\mathbf{x_{01}}\mathbf{x_{01}}^T}{\vert \vert\mathbf{x_{01}}\vert \vert^2})(1-\frac{L}{\vert \vert\mathbf{x_{01}}\vert \vert}) + k\frac{\mathbf{x_{01}}\mathbf{x_{01}}^T}{\vert \vert\mathbf{x_{01}}\vert \vert^2}
$
