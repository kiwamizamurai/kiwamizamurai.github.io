---
title: "How to implement NN from scratch"
date: 2022-04-25T00:00:00+00:00
# weight: 1
# aliases: ["/first"]
tags: ["NN", "ML"]
author: "umasikate"
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
description: "Neural Network with Numpy"
# canonicalURL: "https://canonical.url/to/page"
disableHLJS: true # to disable highlightjs
disableShare: false
disableHLJS: false
hideSummary: false
searchHidden: false
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
cover:
    image: "<image path/url>" # image path/url
    alt: "<alt text>" # alt text
    caption: "<text>" # display caption under cover
    relative: false # when using page bundles set this to true
    hidden: true # only hide on current single page
editPost:
    URL: "https://github.com/umasikate/content/posts"
    Text: "Suggest Changes" # edit text
    appendFilePath: true # to append file path to Edit link
---


# Abstruct

In this post, I am going to explain how mathematics is involved with Neural Network and how to implement it with only numpy.


# Content

this post was written when I was a junior student. Therefore, there might be some mistakes. Please leave the comment below.

## Prerequisites

- rudimentary calculus knowledge
    - Chain Rule
    - Jacobi-Matrix


This sections help you learn to take derivatives of vectors, matrices, and higher order tensors, thereby making you understand the following part.


### derivative

As you know, this equation holds.

$$\begin{array}{c}\vec{y}=\vec{x} W \\\  \frac{d \vec{y}}{d \vec{x}}=W \end{array}$$

What is the derivative? What does it mean? In short, derivative is how sensitivie the specific variable with regard to a certain variable is. Jacobian matrix is a good example.


$$J_f(x) = \left[\frac{\partial f}{\partial x_{1}} \cdots \frac{\partial f}{\partial x_{n}}\right] = \left[\begin{array}{ccc}\frac{\partial f_{1}}{\partial x_{1}} & \cdots & \frac{\partial f_{1}}{\partial x_{n}} \\\ \vdots & \ddots & \vdots \\\ \frac{\partial f_{m}}{\partial x_{1}} & \cdots & \frac{\partial f_{m}}{\partial x_{n}} \end{array}\right]$$

The shape is $m \times n$ due to the fact that the number of all combination of variables is $m \times n$.




### vecotr by matrix

What happens if I differentiate a vector by a matrix. The possible number of combinations of each variable will be

$$\texttt{length of vector} \times \texttt{shape of matrix}$$

(**vector** varies along one coordinate while **matrix** varies along two coordinates)

Let's validate this step by step

$$\vec{y}\_{3} = \vec{x}\_{1} W\_{1,3} + \vec{x}\_{2} W\_{2,3} + \ldots + \vec{x}\_{D} W\_{D, 3}$$

$$\frac{\partial \vec{y}\_{3}}{\partial W\_{7,8}}=0 ~~~~ \frac{\partial \vec{y}\_{3}}{\partial W\_{2,3}}=\vec{x}\_{2}$$

In general, when the index of the $\vec{y}$ component is equal to the second index of $W$, the derivative will be non-zero, but will be zero otherwise. We can write:

$$\frac{\partial\vec{y}\_{j}}{\partial W\_{i, j}} = \vec{x}\_{i}$$

but the other elements of the 3-d array will be 0. If we let $F$ represent the 3d array representing the derivative of $\vec{y}$ with respect to $W$, where

$$F_{i, j, k}=\frac{\partial \vec{y}_{i}}{\partial W\_{j,k}}$$


then

$$F_{i, j, i}=\vec{x}_{j}$$

The thing is that all other entries of $F$ are actually zero. Finally, if we define a new two-dimensional array G as

$$G_{i, j}=F_{i, j, i}$$

we can see that all of the information we need about $F$ can be stored in $G$, and that the non-trivial portion of $F$ is really two-dimensional, not three-dimensional.


### matrix by matrix

In implementations of machine learning, we have to deal with a matrix instead of a single vector.

Let’s assume that each individual $\vec{x}$ is a row vector of length $D$, and that $X$ is a two-dimensional array with $N$ rows and $D$ columns. $W$, as in our last example, will be a matrix with $D$ rows and $C$ columns. $Y$ , given by

$$
Y=X W
$$

will also be a matrix, with $N$ rows and $C$ columns. Thus, each row of $Y$ will give a row vector associated with the corresponding row of the input $X$.

$$
\frac{\partial Y_{a, b}}{\partial X_{c, d}} =
\begin{cases}
    0  & (a \neq c) \\\
    not ~ 0 & (otherwise)
\end{cases}
$$

Furthermore, we can see that

$$\frac{\partial Y_{i, j}}{\partial X_{i, k}}=\frac{\partial }{\partial X_{i, k}} \left(\sum_{k=1}^{D} X_{i, k} W_{k, j}\right)=W_{k, j}$$

doesn’t depend at all upon which row of Y and X we are comparing. This is the same as

$$
\frac{\partial Y_{i,:}}{\partial X_{i,:}}=W^T
$$

## seutp

I assume the following architecture.

$$
\begin{array}{c}
\text { True: } t \\\
\text { Loss: } \mathbb{E}[t - y] \\\
\\
\text { Input: } x_{0} \\\
\text { Layer1 output: } x_{1} = f_{1} \left( W_{1} x_{0} \right) \\\
\text { Layer2 output: } x_{2} = f_{2} \left( W_{2} x_{1} \right) \\\
\text { Output: } x_{3} = f_{3} \left( W_{3} x_{2} \right) \\\
\end{array}
$$

For the sake of implementation, I rewrite this with Matrix.

$$
\begin{array}{c}
\text { True: } T \\\
\text { Objective: } \mathbb{E}[T - Y] \\\
\\\
\text { Input: } X_{0} \\\
\text { Layer1 output: } X_{1} = f_{1} \left( X_{0} W_{1} \right) \\\
\text { Layer2 output: } X_{2} = f_{2} \left( X_{1} W_{2} \right) \\\
\text { Output: } X_{3} = f_{3} \left( X_{2} W_{3} \right) \\\
\end{array}
$$

and the each dimension is as follows

```
in      : id
layer_1 : h1
layer_2 : h2
out     : od
```


## backpropagation

As a loss fuction, I use customized-MSE:

$$ \mathbb{E}[T - Y] = \frac{1}{2} \|X_3 - T\|_2^2 $$

Then, let's derive the derivative w.r.t each Weight.

w.r.t. $W_{3}$

$$
\begin{aligned}
\frac{\partial E}{\partial W_{3}} &= \frac{\partial E}{\partial X_{3}} \frac{\partial X_{3}}{\partial W_{3}} \\\
&=\left(X_{3}-T\right) \frac{\partial X_{3}}{\partial W_{3}} \\\
&=\left[\left(X_{3}-T\right) \circ f_{3}^{\prime}\left( X_{2} W_{3} \right)\right] \frac{\partial X_{2} W_{3} }{\partial W_{3}} \\\
&= X_{2}^{T} \delta_{3} \\\
\text { where } \delta_{3} &=\left[\left(X_{3}-T\right) \circ f_{3}^{\prime}\left( X_{2} W_{3} \right)\right] \\\
\end{aligned}
$$

$\frac{\partial E}{\partial W_{3}}$ must have the same dimensions as $W_3 ~(h2 \times od)$. $\delta_3$ is $n \times od$. And $X_2$ is $n \times h2$

w.r.t. $W_{2}$

$$
\begin{aligned}
\frac{\partial E}{\partial W_{2}} &= \frac{\partial E}{\partial X_{3}} \frac{\partial X_{3}}{\partial W_{2}} \\\
&=\left(X_{3}-T\right) \frac{\partial X_{3}}{\partial W_{2}} \\\
&=\left[\left(X_{3}-t\right) \circ f_{3}^{\prime}\left( X_{2} W_{3} \right)\right] \frac{\partial\left( X_{2} W_{3} \right)}{\partial W_{2}} \\\
&=\delta_{3} \frac{\partial\left(X_{2} W_{3} \right)}{\partial W_{2}} \\\
&=\delta_{3} \frac{\partial\left( X_{2} W_{3} \right)}{\partial X_{2}} \frac{\partial X_{2}}{\partial W_{2}} \\\
&= \delta_{3} W_{3}^{T} \frac{\partial X_{2}}{\partial W_{2}} \\\
&=\left[\delta_{3} W_{3}^{T} \circ f_{2}^{\prime}\left( X_{1} W_{2} \right)\right] \frac{\partial  X_{1} W_{2} }{\partial W_{2}} \\\
&= X_{1}^{T} \delta_{2} \\\
\text { where } \delta_{2} &=\left[ \delta_{3} W_{3}^{T} \circ f_{2}^{\prime}\left(  X_{1} W_{2} )\right)\right] \\\
\end{aligned}
$$

$\frac{\partial E}{\partial W_{2}}$ must have the same dimensions as $W_2 ~(h1 \times h2)$. $\delta_2$ is $n \times h2$. And $X_1$ is $n \times h1$

w.r.t. $W_{1}$

$$
\begin{aligned}
\frac{\partial E}{\partial W_{1}} &=\left[W_{2}^{T} \delta_{2} \circ f_{1}^{\prime}\left( X_{0} W_{1} \right)\right] X_{0}^{T} \\\
&= X_{0}^{T} \delta_{1}
\end{aligned}
$$

$\frac{\partial E}{\partial W_{1}}$ must have the same dimensions as $W_1 ~(id \times h1)$. $\delta_1$ is $n \times h1$. And $X_0$ is $n \times id$

## code

It's time to implement this. As a dataset, I use iris from sklearn. For the sake of validation of our implementation, I utilized the loss and accuracy.

{{< gist kiwamizamurai 3fdd2f6a0f3c6b963e30ae1f4549384e >}}

# References

- [Vector, Matrix, and Tensor Derivatives by Erik Learned-Miller](http://cs231n.stanford.edu/vecDerivs.pdf)
