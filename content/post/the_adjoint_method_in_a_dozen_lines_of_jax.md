+++
title = "The Adjoint Method in a Dozen Lines of JAX"
author = ["Chris Cundy"]
draft = false
date = "2021-09-10T13:00:00+00:00"
+++

The Adjoint Method is a powerful method for computing derivatives of functions involving constrained optimization. It's been around for a long time, but recently has been popping up in machine learning, used in papers such as the [Neural ODE](https://arxiv.org/abs/1806.07366) and many others. I found it a bit hard to grasp until I implemented some toy examples in JAX. This post outlines the adjoint method, and steps through some concrete examples using JAX. I also discuss how this method interacts with modern autodiff.


## The Adjoint Method {#the-adjoint-method}


### Setting {#setting}

Our setting involves two vectors, \\(x \in \mathbb{R}^{d\_x}\\) and \\(\theta \in \mathbb{R}^{d\_\theta}\\).
We want to find the gradient (with respect to \\(\theta\\)) of a function
\\(f: \mathbb{R}^{d\_x} \times \mathbb{R}^{d\_\theta} \to \mathbb{R}\\). Here, \\(x\\) is implicitly
defined as a solution to \\(g(x, \theta) = \boldsymbol{0}\\) for
\\(g: \mathbb{R}^{d\_x} \times \mathbb{R}^{d\_\theta} \to \mathbb{R}^{d\_x}\\). This is a set of \\(d\_x\\) equations.

As a completely trivial example, we could consider \\(f(x, \theta) = x + \theta\\), subject to
\\(x = \theta\\), which we can write as \\(g(x, \theta) = 0\\) for \\(g(x, \theta) = x - \theta\\).

As a more complicated example, consider \\(\theta\\) parameterising the design of a building, and \\(x\\) certain stresses on the foundations. The cost \\(f\\) is the monetary cost of the design, with penalties for high stresses. The function \\(g\\) is some complicated simulation computing the stresses given the design.


### The Adjoint Equation {#the-adjoint-equation}

The core of the adjoint method is in the following equations.
I'm going to use \\(\partial\_a b\\) to denote the [Jacobian](https://en.wikipedia.org/wiki/Jacobian%5Fmatrix%5Fand%5Fdeterminant) of the function \\(b\\) with respect to \\(a\\). This is a matrix in \\(\mathbb{R}^{d\_{\text{out}}\times d\_{\text{in}}}\\), where \\(d\_\text{out}\\) is the dimension of the output of \\(b\\) and \\(d\_{\text{in}}\\) is the dimension of the input of \\(b\\), and \\((\partial \_a b)\_{ij}\\) is the derivative of the \\(i\\)th coordinate of the output of \\(b\\) with respect to the \\(j\\)th coordinate of the input.
It's worth belabouring this point since the adjoint method involves a lot of algebra with Jacobians.

We first define an **adjoint vector** \\(\lambda \in \mathbb{R}^{d\_x}\\) as the solution to the equation

\begin{align\*}
(\partial\_x g)^\top \lambda = -(\partial\_x f)^\top.
\end{align\*}

Then, the adjoint method says we can write the gradient of \\(f\\) with respect to \\(\theta\\) as

\begin{align\*}
\nabla\_\theta f = \lambda^\top \partial\_\theta g + (\partial\_\theta f)^\top.
\end{align\*}

The intuition behind this is that \\(g(x, \theta) = 0\\) implicitly defines the \\(x\\) which solves it, \\(x^\*\\), as a function of \\(\theta\\). The implicit function theorem then gives us this result.
This is useful if we have to do some complicated procedure to find the \\(x\\) satisfying \\(g(x, \theta)\\). If we want to compute \\(\nabla\_\theta f\\), We don't need to differentiate through that complicated procedure, and can just use the equation \\(g\\) directly. This allows us to even use non-differentiable methods to find \\(x\\), as long as \\(g\\) itself is available.


## Examples {#examples}


### Pen-and-paper {#pen-and-paper}

As a sanity check, let's solve this for a simple example, \\(f(x, \theta) = x + \theta\\), with \\(g(x, \theta) = \theta^2 + \theta - x\\). If we directly substitute \\(x\\) for \\(\theta\\) in the expression for \\(f\\), we can see that \\(\nabla\_\theta f = 2\theta + 2\\).

Alternatively, using the adjoint method we have to solve \\(-1\lambda = -1\\), giving \\(\lambda = 1\\), and then find \\(\nabla\_\theta f = (2\theta + 1) + 1 = 2\theta + 2\\). So we do indeed get the same result.


### Circles {#circles}

Let's consider a more involved example, although still a bit contrived. As we run through this on paper, we'll also use JAX to double-check everything. In this problem, \\(\theta = [r, \varphi, {\hat x}\_0, {\hat x}\_1]\\) ,
\\(f(x, \theta) = \\|x - w\\|^2\\) for some fixed \\(w\in\mathbb{R}^2\\), and \\(g(x, \theta) = \left[x\_0 - (r\cos\varphi + {\hat x}\_0), x\_1 - (r\sin\varphi + {\hat x}\_1)\right]\\).
In other words, \\(\theta\\) defines a circle with centre \\(x\_0, x\_1\\), and a point on the circle with polar coordinates \\(r, \varphi\\).

```python
import jax.numpy as np

optimal_point = np.array([1., 3.])
theta_0 = rnd.normal(rnd.PRNGKey(1), shape=(4,))

def f(x, theta):
    return np.linalg.norm(x - optimal_point) ** 2

def g(x, theta):
    r, phi, x_hat_0, x_hat_1 = theta
    return np.array((x[0] - (r * np.cos(phi) + x_hat_0), x[1] - (r * np.sin(phi) + x_hat_1)))

```

By hand, we can see that \\(\partial\_x g\\) is the identity matrix, and \\(\partial\_x f\\) is the vector \\((2(x\_0 - w\_0), 2(x\_1 - w\_1))\\).
Note that we still have to know what \\(x\\) is to evaluate \\(\partial\_x f\\). We find this by solving \\(g(x, \theta) = 0\\).
We can check this with JAX, using the `jacfwd` function to get the Jacobian of a function.

```python
from jax import jacfwd

r, phi, x_hat_0, x_hat_1 = theta_0
# Find the point solving g(x, θ) = 0
our_x = np.array([r * np.cos(phi) + x_hat_0, r * np.sin(phi) + x_hat_1])

g_x = lambda x: g(x, theta_0) # Get rid of theta-dependence
partial_x_g = jacfwd(g_x) #Return a function giving the Jacobian at a point
print(partial_x_g(our_x)) # Evaluate the Jacobian

f_x = lambda x: f(x, theta_0) # Get rid of theta-dependence
partial_x_f_fn = jacfwd(f_x)
partial_x_f = partial_x_f(our_x)
print(partial_x_f)
# Double-check this is the same as what we derived by hand:
print(np.array((2 * (our_x[0] - optimal_point[0]),
		2 * (our_x[1] - optimal_point[1]))))
```

Next we need to compute \\(\partial\_\theta g\\). A bit of algebra gives

\begin{align\*}
\partial\_\theta g = \begin{pmatrix} -\cos \varphi & r\sin \varphi & -1 & 0 \\ -\sin \varphi & -r\cos \varphi & 0 & -1 \end{pmatrix}.
\end{align\*}

Now, we can double-check this in JAX

```python
g_theta = lambda theta: g(our_x, theta) # Get rid of x-dependence
partial_theta_g_fn = jacfwd(g_theta)
partial_theta_g = partial_theta_g_fn(theta_0)
print(partial_theta_g)
# Double-check this is the same as what we derived by hand:
print(np.array([[-np.cos(phi), r * np.sin(phi)],
		[-np.sin(phi), -r * np.cos(phi)]]))

```

Now we can plug in to get the gradient derived from the adjoint method, and compare to the
gradient of a function that includes solving \\(g\\):

```python
def f_g_combined(theta):
    r, phi, x_hat_0, x_hat_1 = theta
    x = np.array((r * np.cos(phi) + x_hat_0, r * np.sin(phi) + x_hat_1))
    return np.linalg.norm(x - optimal_point) ** 2

print(grad(f_g_combined)(theta_0))
print(-partial_x_f @ partial_theta_g)
```
