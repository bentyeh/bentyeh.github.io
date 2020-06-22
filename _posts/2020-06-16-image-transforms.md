---
title: 2-D Image Transformations and Alignment
layout: post
use_toc: true
use_math: true
excerpt: Given a set of corresponding points in two images, how can you align them? What if the pixel aspect ratios in the two images are different?
last_updated: 2020-06-21
---

# Representation

[Raster images](https://en.wikipedia.org/wiki/Raster_graphics) are often represented as an array of shape $$(Y, X, C)$$ corresponding to the height, width, and number of channels (e.g., $$C = 3$$ for RGB images). The values in the array indicate intensities. By convention, a coordinate of $$(y, x) = (0, 0)$$ (0-indexed) refers to the upper-left corner of the image. [[MATLAB](https://www.mathworks.com/help/images/image-coordinate-systems.html), [Python - Matplotlib](https://matplotlib.org/3.1.1/tutorials/intermediate/imshow_extent.html), [Python - Pillow]((https://pillow.readthedocs.io/en/stable/handbook/concepts.html#coordinate-system))]

For the purpose of computing 2-D transformations, however, we are primarily concerned with pixel coordinates. A pixel coordinate $$\vec{p}$$ can be represented as a vector

$$\vec{p} = \begin{bmatrix} x \\ y \end{bmatrix}$$

Note that $$x, y \in \mathbb{R}$$ are not limited to the set of non-negative integers. While a transformed coordinate does not correspond to a discrete pixel in an image, the intensities of the transformed image can be estimated by interpolation (see [below](#interpolating-intensities-from-transformed-coordinates)).

Linear transformations (maps) such as rotation, scaling, and [shear](https://en.wikipedia.org/wiki/Shear_mapping) can be represented as matrix multiplication

$$\begin{equation} \label{eq:linear_transformation}
\vec{p}' = A \vec{p}
\end{equation}$$

$$\begin{aligned}
A_\text{rotate} &=
    \begin{bmatrix}
        \cos(\theta) & -\sin(\theta) \\
        \sin(\theta) & \cos(\theta)
    \end{bmatrix} & \text{rotate counterclockwise by $\theta$} \\
A_\text{scale} &=
    \begin{bmatrix}
        s_x & 0 \\
        0 & s_y
    \end{bmatrix} & \text{scale by $s_x$ horizontally and $s_y$ vertically} \\
A_\text{shear} &=
    \begin{bmatrix}
        1 & c_x \\
        c_y & 1
    \end{bmatrix} & \text{shear by $c_x$ horizontally and $c_y$ vertically}
\end{aligned}$$

Since translations

$$\begin{aligned}
\vec{p}' &= \vec{p} + \vec{t} = \vec{p} + \begin{bmatrix} t_x \\ t_y \end{bmatrix} & \text{translate by $t_x$ horizontally and $t_y$ vertically}
\end{aligned}$$

are not strictly linear transformations, they cannot be represented as matrix multiplication as in Equation \ref{eq:linear_transformation}. However, if pixels are represented in [homogeneous coordinates](https://en.wikipedia.org/wiki/Homogeneous_coordinates), [affine](https://en.wikipedia.org/wiki/Affine_transformation) (linear map with translations) and [projective](https://en.wikipedia.org/wiki/Homography) transformations can also be computed using matrix multiplication.

For affine transformations, it is sufficient to work with a simple [augmented representation](https://en.wikipedia.org/wiki/Affine_transformation#Augmented_matrix) for both the pixel coordinate vector and the transformation.

$$
\underbrace{\begin{bmatrix} x' \\ y' \\ 1 \end{bmatrix}}_{\vec{p}'}
= \underbrace{\begin{bmatrix}
    s_x \cdot \cos(\theta) & -c_x \cdot \sin(\theta) & t_x \\
    c_y \cdot \sin(\theta) & s_y \cdot \cos(\theta) & t_y \\
    0 & 0 & 1
    \end{bmatrix}}_A
  \underbrace{\begin{bmatrix} x \\ y \\ 1 \end{bmatrix}}_{\vec{p}}
$$

For projective transformations, however, the first two elements of the last row of the transformation matrix $$A$$ may be nonzero. Consequently, the "augmented element" of the transformed vector $$\vec{p}'$$ is not necessarily $$1$$ and is a general example of a homogeneous coordinate:

> Given a point (x, y) on the Euclidean plane, for any non-zero real number Z, the triple (xZ, yZ, Z) is called a set of homogeneous coordinates for the point. By this definition, multiplying the three homogeneous coordinates by a common, non-zero factor gives a new set of homogeneous coordinates for the same point. In particular, (x, y, 1) is such a system of homogeneous coordinates for the point (x, y). [[Wikipedia]](https://en.wikipedia.org/wiki/Homogeneous_coordinates)

The Cartesian coordinate $$(\tilde{x}, \tilde{y})$$ corresponding to a homogeneous coordinate $$\vec{p}' = \begin{bmatrix} x' & y' & z \end{bmatrix}^\top$$ is then given by

$$(\tilde{x}, \tilde{y}) = \left(\frac{x'}{z}, \frac{y'}{z} \right)$$

## Types of 2-D transformations

A general 2-D transformation can be represented as a 3x3 matrix

$$
T = \begin{bmatrix}
a_0 & a_1 & a_2 \\
b_0 & b_1 & b_2 \\
c_0 & c_1 & c_2
\end{bmatrix}
$$

Different constraints on $$T$$ give rise to special named transformations with recognizable geometries.

<p style="text-align: center"><strong><a name="table1">Table 1: Special transformations</a></strong></p>

| Transform | [Projective (homography)](https://en.wikipedia.org/wiki/Homography) | [Affine](https://en.wikipedia.org/wiki/Affine_transformation) | [Similarity](https://en.wikipedia.org/wiki/Similarity_(geometry)) | [Euclidean (rigid)](https://en.wikipedia.org/wiki/Rigid_transformation) |
|-------|-------|-------|-------|-------|
| Constraints | $$c_2 = 1$$ | Projective<br>$$c_1 = c_2 = 0$$ | Affine<br>$$a_0 = b_1$$, $$b_0 = -a_1$$ | Similarity<br>$$\begin{aligned} &\cos^{-1}(a_0) \\ &= \sin^{-1}(-a_1) \\ &= \sin^{-1}(b_0) \\ &= \cos^{-1}(b_1)\end{aligned}$$ |
| # Free Parameters | 8 | 6 | 4 | 3 |
| # Defining Points | 4 | 3 | 2 | <span style="color:red">?</span> |
| Rotation | Y | Y | Y | Y |
| Translation | Y | Y | Y | Y |
| Scaling | Y | Y | Y (isotropic: same in x- and y-directions) | N |
| Shear | Y | Y | N | N |
| Geometric interpretation | Maps lines to lines | Preserves lines and parallelism | Preserves shape | Preserves Euclidean distance between points |

- Just as how "[t]he point represented by a given set of homogeneous coordinates is unchanged if the coordinates are multiplied by a common factor" ([Wikipedia](https://en.wikipedia.org/wiki/Homogeneous_coordinates)), the transform represented by a projective transformation matrix is unchanged if the matrix is multiplied by a scalar: the homogeneous coordinate given by $$T_\text{projective} \cdot \vec{p}$$ represents the same point as the homogeneous coordinate $$k T_\text{projective} \cdot \vec{p}$$ for any scalar $$k \in \mathbb{R}$$. Consequently, a projective transformation is uniquely represented by a projective transformation matrix up to a scalar factor, namely $$c_2$$. This is why a projective transformation matrix can always be represented with $$c_2 = 1$$.
- Since transformations can be represented as matrix muliplications, multiple transformations can be composed (combined) together $$T_\text{composed} = T_1 T_2 \cdots$$. There are two important considerations when composing transformations:
  1. Non-commutativity: because [matrix multiplication does not commute in general](https://en.wikipedia.org/wiki/Matrix_multiplication#Non-commutativity), the order in which transformations are composed matters. One consequence of non-commutativity is that affine transformations cannot be uniquely decomposed into translation, rotation, scale, and shear transformations, unless the order of transformations is known (for example, see this [StackExchange Math question](https://math.stackexchange.com/questions/78137/decomposition-of-a-nonsquare-affine-matrix/)). The chart below summarizes which affine transformations commute with one another; code verifying the chart can be found in the Colab notebook linked at the end of the post.

      | Commutativity | Translation | Scale | Rotation | Shear |
      |---------------|-------------|-------|----------|-------|
      | Translation   | Yes         | No    | No       | No    |
      | Scale         | No          | Yes   | No       | No    |
      | Rotation      | No          | No    | Yes      | No    |
      | Shear         | No          | No    | No       | No    |

  2. The type ([Table 1](#table1)) of the composed transform is generally the same as the most general type among the individual transforms being composed. For example, a Euclidean transform composed with a Euclidean transform is still a Euclidean transform, but a Euclidean transform composed with an affine transform is an affine transform but not necessarily a Euclidean transform.

<!-- 
Any point in the projective plane is represented by a triple (X, Y, Z), called the homogeneous coordinates or projective coordinates of the point, where X, Y and Z are not all 0.


 an [augmented representation](https://en.wikipedia.org/wiki/Affine_transformation#Augmented_matrix) is used for both the pixel coordinate vector and the transformation matrix, 

$$
\begin{bmatrix}
x' \\ y' \\ z
\end{bmatrix} = 
\begin{bmatrix}
a_0 & a_1 & a_2 \\
b_0 & b_1 & b_2 \\
c_0 & c_1 & c_2
\end{bmatrix}
\begin{bmatrix}
x \\ y \\ 1
\end{bmatrix}
$$


 it is useful to separately represent pixel coordinates as a matrix of :

$$
\begin{bmatrix}
x_1 & x_2 & \cdots \\
y_1 & y_2 & \cdots \\
1   & 1   & \cdots
\end{bmatrix}
$$

for $$i = 1, ..., n$$ where $$n$$ is the number of pixels. Note that $$x_i, y_i \in \mathbb{R}$$ are no longer limited to the set of non-negative integers. -->

# Learning transformations

Let $$S \in \mathbb{R}^{3 \times n}$$ be a set of $$n$$ augmented coordinates in the source image and $$D \in \mathbb{R}^{3 \times n}$$ be a set of corresponding augmented coordinates in the destination image.

$$
S = \begin{bmatrix}
x_{s1} & x_{s2} & \cdots \\
y_{s1} & y_{s2} & \cdots \\
1   & 1   & \cdots
\end{bmatrix}, \quad
D = \begin{bmatrix}
x_{d1} & x_{d2} & \cdots \\
y_{d1} & y_{d2} & \cdots \\
1   & 1   & \cdots
\end{bmatrix}
$$

We want to learn the transformation matrix $$T$$ such that $$T \cdot S \approx D$$.

For each set $$i = 1, ..., n$$ of corresponding points $$s_i, d_i$$, we can explicitly write out $$T s_i = d_i$$ as

$$\begin{aligned}
x_{di} &= \frac{a_0 x_{si} + a_1 y_{si} + a_2}{c_0 x_{si} + c_1 x_{si} + c_2} \\
y_{di} &= \frac{b_0 x_{si} + b_1 y_{si} + b_2}{c_0 x_{si} + c_1 x_{si} + c_2} \\
d_{i2} &= \frac{c_0 x_{si} + c_1 x_{si} + c_2}{c_0 x_{si} + c_1 x_{si} + c_2} = 1
\end{aligned}$$

The denominator $$c_0 x_{si} + c_1 x_{si} + c_2$$ ensures that $$d_{i2} = 1$$ (i.e., the last row of $$D$$ is all ones). After rearranging terms, these equations can be re-written as

$$\begin{aligned}
0 &= a_0 x_{si} + a_1 y_{si} + a_2 - c_0 x_{si} x_{di} - c_1 x_{si} x_{di} - c_2 x_{di} \\
0 &= b_0 x_{si} + b_1 y_{si} + b_2 - c_0 x_{si} y_{di} - c_1 x_{si} y_{di} - c_2 y_{di}
\end{aligned}$$

Consequently, we have a set of $$2n$$ linear equations that can be expressed in matrix form as a [homogeneous system](https://en.wikipedia.org/wiki/System_of_linear_equations#Homogeneous_systems) $$0_{2n} = A \cdot x$$ where

$$\begin{equation} \label{eq:linear_system_2n}
A = \begin{bmatrix}
    x_{s1} & y_{s1} & 1 & 0 & 0 & 0 & -x_{s1} x_{d1} & -y_{s1} x_{d1} & -x_{d1} \\
    0 & 0 & 0 & x_{s1} & y_{s1} & 1 & -x_{s1} y_{d1} & -y_{s1} y_{d1} & -y_{d1} \\
    x_{s2} & y_{s2} & 1 & 0 & 0 & 0 & -x_{s2} x_{d2} & -y_{s2} x_{d2} & -x_{d2} \\
    0 & 0 & 0 & x_{s2} & y_{s2} & 1 & -x_{s2} y_{d2} & -y_{s2} y_{d2} & -y_{d2} \\
    & & & & & \vdots
\end{bmatrix}, \quad
x = \begin{bmatrix}
    a_0 \\ a_1 \\ a_2 \\ b_0 \\ b_1 \\ b_2 \\ c_0 \\ c_1 \\ c_2
\end{bmatrix}
\end{equation}$$

Such as a system of equations can be solved in general using methods such as [singular value decomposition](https://en.wikipedia.org/wiki/Singular_value_decomposition#Solving_homogeneous_linear_equations) or gradient descent. However, simplified systems of equations exist for the special cases give in [Table 1](#table1).

<details markdown="block"><summary>Simplified equations for each special transformation.</summary>

Projective: $$c_2 = 1$$

$$
\begin{bmatrix}
x_{d1} \\ y_{d1} \\ x_{d2} \\ y_{d2} \\ \vdots
\end{bmatrix}
= \begin{bmatrix}
    x_{s1} & y_{s1} & 1 & 0 & 0 & 0 & -x_{s1} x_{d1} & -y_{s1} x_{d1} \\
    0 & 0 & 0 & x_{s1} & y_{s1} & 1 & -x_{s1} y_{d1} & -y_{s1} y_{d1} \\
    x_{s2} & y_{s2} & 1 & 0 & 0 & 0 & -x_{s2} x_{d2} & -y_{s2} x_{d2} \\
    0 & 0 & 0 & x_{s2} & y_{s2} & 1 & -x_{s2} y_{d2} & -y_{s2} y_{d2} \\
    & & & & & \vdots
\end{bmatrix}
\begin{bmatrix}
    a_0 \\ a_1 \\ a_2 \\ b_0 \\ b_1 \\ b_2 \\ c_0 \\ c_1
\end{bmatrix}
$$

Affine: $$c_1 = c_2 = 0$$

$$
\begin{bmatrix}
x_{d1} \\ y_{d1} \\ x_{d2} \\ y_{d2} \\ \vdots
\end{bmatrix}
= \begin{bmatrix}
    x_{s1} & y_{s1} & 1 & 0 & 0 & 0 \\
    0 & 0 & 0 & x_{s1} & y_{s1} & 1 \\
    x_{s2} & y_{s2} & 1 & 0 & 0 & 0 \\
    0 & 0 & 0 & x_{s2} & y_{s2} & 1 \\
    & & \vdots
\end{bmatrix}
\begin{bmatrix}
    a_0 \\ a_1 \\ a_2 \\ b_0 \\ b_1 \\ b_2
\end{bmatrix}
$$

Similarity: $$a_0 = b_1$$, $$b_0 = -a_1$$

$$
\begin{bmatrix}
x_{d1} \\ y_{d1} \\ x_{d2} \\ y_{d2} \\ \vdots
\end{bmatrix}
= \begin{bmatrix}
    x_{s1} + 0 & y_{s1} - 0 & 1 & 0 \\
    0 + y_{s1} & 0 - x_{s1} & 0 & 1 \\
    x_{s2} + 0 & y_{s2} - 0 & 1 & 0 \\
    0 + y_{s2} & 0 - x_{s2} & 0 & 1 \\
    & \vdots
\end{bmatrix}
\begin{bmatrix}
    a_0 \\ a_1 \\ a_2 \\ b_2
\end{bmatrix}
= \begin{bmatrix}
    x_{s1} &  y_{s1} & 1 & 0 \\
    y_{s1} & -x_{s1} & 0 & 1 \\
    x_{s2} &  y_{s2} & 1 & 0 \\
    y_{s2} & -x_{s2} & 0 & 1 \\
    & \vdots
\end{bmatrix}
\begin{bmatrix}
    a_0 \\ a_1 \\ a_2 \\ b_2
\end{bmatrix}
$$

</details>

Therefore, to have a fully-determined or overdetermined system of linear equations, the number of corresponding points $$n$$ must be at least half the number of free parameters (see [Table 1](#table1)).

Compare this approach with solving for $$T = D \cdot S^{-1}$$. Since the last row of $$D$$ is always $$\begin{bmatrix} 1 & 1 & 1\end{bmatrix}$$, the last row of $$D \cdot S^{-1}$$ is always $$\begin{bmatrix} 0 & 0 & 1\end{bmatrix}$$. Consequently, solving for $$T$$ as $$D \cdot S^{-1}$$ can learn an affine transformation, but not a projective transformation.
- This appears to hold if the pseudoinverse of $$S$$ is used instead of its inverse.
  - If the number of paired coordinates $$n > 3$$, then $$S$$ is not square, so it is not invertible.
  - $$S^\top (S S^\top)^{-1}$$ gives the Moore-Penrose pseudoinverse when $$S$$ is a full-rank fat ($$n \geq 3$$) matrix.
- The affine transformation learned this way appears identical to the affine transformation learned using Equation $$\ref{eq:linear_system_2n}$$.

<details markdown="block"><summary>Details</summary>

If $$n = 3$$ (i.e., $$S$$ and $$D$$ contain 3 corresponding points), then the inverse of $$S$$ is [given by](https://www.wolframalpha.com/input/?i=Inverse[{ {x1,+x2,+x3},+{y1,+y2,+y3},+{1,+1,+1}}])

$$S^{-1} = \underbrace{\frac{1}{x_{s1} y_{s2} - x_{s1} y_{s3} - x_{s2} y_{s1} + x_{s2} y_{s3} + x_{s3} y_{s1} - x_{s3} y_{s2}}}_k \begin{bmatrix}
y_{s2} - y_{s3} & x_{s3} - x_{s2} & x_{s2} y_{s3} - x_{s3} y_{s2} \\
y_{s3} - y_{s1} & x_{s1} - x_{s3} & x_{s3} y_{s1} - x_{s1} y_{s3} \\
y_{s1} - y_{s2} & x_{s2} - x_{s1} & x_{s1} y_{s2} - x_{s2} y_{s1}
\end{bmatrix}$$

Let $$k$$ denote the constant fraction factor. Since the last row of $$D$$ is always $$\begin{bmatrix} 1 & 1 & 1\end{bmatrix}$$, we get

$$
T_{31} = \sum_{i=1}^3 {S^{-1}}_{i1}
= k ((y_{s2} - y_{s3}) + (y_{s3} - y_{s1}) + (y_{s1} - y_{s2}))
= 0
$$

$$
T_{32} = \sum_{i=1}^3 {S^{-1}}_{i2}
= k ((x_{s3} - x_{s2}) + (x_{s1} - x_{s3}) + (x_{s2} - x_{s1}))
= 0
$$

$$\begin{aligned}
T_{33}
&= \sum_{i=1}^3 {S^{-1}}_{i3} \\
&= k ((x_{s2} y_{s3} - x_{s3} y_{s2}) + (x_{s3} y_{s1} - x_{s1} y_{s3}) + (x_{s1} y_{s2} - x_{s2} y_{s1})) \\
&= \frac{x_{s2} y_{s3} - x_{s3} y_{s2} + x_{s3} y_{s1} - x_{s1} y_{s3} + x_{s1} y_{s2} - x_{s2} y_{s1}}{x_{s1} y_{s2} - x_{s1} y_{s3} - x_{s2} y_{s1} + x_{s2} y_{s3} + x_{s3} y_{s1} - x_{s3} y_{s2}} \\
&= 1
\end{aligned}$$

So

$$T = D \cdot S^{-1} = 
\begin{bmatrix}
? & ? & ? \\
? & ? & ? \\
0 & 0 & 1
\end{bmatrix}$$

This appears to hold if the pseudoinverse of $$S$$ is used instead of its inverse (<span style="color:red">analytical proof?</span>):

```python
import numpy as np
n = 4 # number of points; an integer >= 3
S = np.vstack((np.random.rand(2, n), np.ones(n)))
D = np.vstack((np.random.rand(2, n), np.ones(n)))
D @ np.linalg.pinv(S)
# check that the last row is [0, 0, 1], within numerical error
```

The affine transformation learned this way appears identical to the affine transformation learned using Equation $$\ref{eq:linear_system_2n}$$ (<span style="color:red">analytical proof?</span>):

```python
import numpy as np
n = 4 # number of points; an integer >= 3
S = np.vstack((np.random.rand(2, n), np.ones(n)))
D = np.vstack((np.random.rand(2, n), np.ones(n)))
A = np.zeros((2 * n, 6))
for i in range(n):
    A[2 * i, 0:3] = S[:, i]
    A[2 * i + 1, 3:6] = S[:, i]

T1 = D @ np.linalg.pinv(S)
T2 = np.vstack(((np.linalg.pinv(A) @ D[:2, ].T.reshape(2 * n)).reshape(2, 3), [0, 0, 1]))
np.isclose(T1, T2) # should be all True
```

</details>

## Pixel aspect ratios

Let $$r$$ be the pixel aspect ratio:

$$r = \frac{\text{width represented by a pixel}}{\text{height represented by a pixel}}$$

Usually, we work with square ($$r = 1$$) pixels. However, images (especially those generated by different modalities, such as mass spectrometry imaging, MRI, etc.) may have non-square pixels. For example, a pixel that corresponds to physical dimensions of 1 cm width x 2 cm height would have a pixel aspect ratio of 0.5.

Consider a source image $$\mathcal{I}_S$$ and a destination image $$\mathcal{I}_D$$, which may be of different dimensions. Let $$r_S, r_D$$ be the aspect ratios of pixels in the source and destination images, respectively. We want to generate an image $$\mathcal{I}_{S'}$$ of the same dimension as the destination image $$\mathcal{I}_D$$ in which the source image is aligned to the destination image.

Let $$S$$ and $$D$$ be corresponding homogeneous coordinates as defined above. Let $$\tilde{r} = \frac{r_D}{r_S}$$ be the ratio of pixel aspect ratios. Then,

$$
\tilde{D} = \begin{bmatrix}
1 & 0 & 0 \\
0 & \frac{1}{\tilde{r}} & 0 \\
0 & 0 & 1
\end{bmatrix} \cdot D
= \begin{bmatrix}
x_{d1} & x_{d2} & \cdots \\
\frac{y_{d1}}{\tilde{r}} & \frac{y_{d2}}{\tilde{r}} & \cdots \\
1   & 1   & \cdots
\end{bmatrix}
$$

transforms coordinates in the destination image to a space of the same pixel aspect ratio as in the source image.

Let $$\tilde{T} \in \mathbb{R}^{3 \times 3}$$ be a transform mapping coordinates $$S$$ to $$\tilde{D}$$:

$$T \cdot S = \tilde{D}.$$

We can solve for $$\tilde{T}$$ using the systems of equations [discussed above](#learning-transformations), giving the transform that maps $$S$$ to $$\tilde{D}$$, but not to $$D$$ as desired. To account for the pixel aspect ratios, observe that

$$
\tilde{T} \cdot S = \begin{bmatrix}
1 & 0 & 0 \\
0 & \frac{1}{\tilde{r}} & 0 \\
0 & 0 & 1
\end{bmatrix} \cdot D \rightarrow
\begin{bmatrix}
1 & 0 & 0 \\
0 & \tilde{r} & 0 \\
0 & 0 & 1
\end{bmatrix} \cdot \tilde{T} \cdot S =
T \cdot S = D
$$

For projective and affine transformation matrices, the scaling matrix $$\begin{bmatrix}
1 & 0 & 0 \\
0 & \tilde{r} & 0 \\
0 & 0 & 1
\end{bmatrix}$$ can be folded into the final transformation matrix

$$T = \begin{bmatrix}
1 & 0 & 0 \\
0 & \tilde{r} & 0 \\
0 & 0 & 1
\end{bmatrix} \tilde{T}$$

without violating the constraints of an affine or projective matrix. In fact, the idea of "constraining" a projective or affine transformation to respect pre-defined pixel aspect ratio $$\tilde{r}$$ is not possible. The final learned transformation matrix $$T$$ can just as well be fit on $$S$$ and $$D$$ directly, without first having to scale $$D$$.

For more constrained transformation matrices, the final transformation matrix cannot be learned directly from $$S$$ and $$D$$ without violating assumed constraints on $$T$$.

## Interpolating intensities from transformed coordinates

Once you have a transformation matrix, how do you generate the transformed *image*, not just the transformed *coordinates*?

One general approach used by many popular image libraries ([Python - Pillow](https://pillow.readthedocs.io/en/stable/reference/Image.html?highlight=image.transform#PIL.Image.Image.transform), [Python - scikit-image](https://scikit-image.org/docs/stable/api/skimage.transform.html#skimage.transform.warp), [MATLAB](https://www.mathworks.com/help/images/ref/imwarp.html)) is inverse mapping followed by interpolation. [MATLAB's documentation of its `imwarp()` function](https://www.mathworks.com/help/images/ref/imwarp.html#mw_c3b3bd6b-2215-480f-ad17-e8c1014f1c2e) nicely summarizes the procedure:
> `imwarp` determines the value of pixels in the output image by mapping locations in the output image to the corresponding locations in the input image (inverse mapping). `imwarp` interpolates within the input image to compute the output pixel value.

Additional notes:
1. Visualizing rotations can be tricky due to the convention of displaying image coordinates $$(0, 0)$$ (in a 0-indexed language like Python) or $$(1, 1)$$ (in a 1-indexed language like MATLAB) in the upper left corner. A statement such as "rotate an image by $$\theta$$ radians counterclockwise" usually means to produce an output image such that, when displayed using image coordinate conventions, the image appears rotated by $$\theta$$ radians counterclockwise. However, if the rotated image is displayed with the origin in the lower-left corner, then it appears as if the image was rotated *clockwise*. Consequently, the rotation matrix for "rotate an image by $$\theta$$ radians counterclockwise" is

    $$
    \begin{bmatrix}
        \cos(-\theta) & -\sin(-\theta) & 0 \\
        \sin(-\theta) & \cos(-\theta) & 0 \\
        0 & 0 & 1
    \end{bmatrix}
    $$

    See the Colab notebook for an example.
2. In `skimage.transform.warp()`, the interpolation step relies on [`scipy.ndimage.map_coordinates()`](https://docs.scipy.org/doc/scipy/reference/generated/scipy.ndimage.map_coordinates.html).
3. When dealing with non-square pixels, the inverse transformation is given by

$$
T^{-1}
= \left(\begin{bmatrix}
    1 & 0 & 0 \\
    0 & \tilde{r} & 0 \\
    0 & 0 & 1
\end{bmatrix} \tilde{T} \right)^{-1}
= \tilde{T}^{-1} \begin{bmatrix}
    1 & 0 & 0 \\
    0 & \frac{1}{\tilde{r}} & 0 \\
    0 & 0 & 1
\end{bmatrix}
$$

<script src="https://gist.github.com/bentyeh/eea647fe51f8260c6dc5766f07cbd088.js"></script>

# Additional references

- scikit-image documentation
  - [User Guide: Geometrical transformations of images](https://scikit-image.org/docs/stable/user_guide/geometrical_transform.html)
  - [Example: Using geometric transforms](https://scikit-image.org/docs/stable/auto_examples/transform/plot_geometric.html)
  - [Example: Types of homographies](https://scikit-image.org/docs/stable/auto_examples/transform/plot_transform_types.html)
- Difference between affine and projective transformations: [StackOverflow](https://math.stackexchange.com/questions/1319680/what-is-the-difference-between-affine-and-projective-transformations)
- Learning a projective transform matrix from 4 points: [StackExchange Math](https://math.stackexchange.com/questions/296794/finding-the-transform-matrix-from-4-projected-points-with-javascript)