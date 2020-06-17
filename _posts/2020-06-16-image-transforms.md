---
title: 2-D Image Transformations and Alignment
layout: post
use_toc: true
use_math: true
excerpt: Given a set of corresponding points in two images, how can you align them? What if the pixel aspect ratios in the two images are different?
---

## Background

Consider a 2-D transformation matrix of the form

$$
T = \begin{bmatrix}
a_0 & a_1 & a_2 \\
b_0 & b_1 & b_2 \\
c_0 & c_1 & c_2
\end{bmatrix}
$$

Different constraints on $$T$$ give rise to special named transformations with recognizable geometries:

| Transform | [Projective (homography)](https://en.wikipedia.org/wiki/Homography) | [Affine](https://en.wikipedia.org/wiki/Affine_transformation) | [Similarity](https://en.wikipedia.org/wiki/Similarity_(geometry)) | [Euclidean (rigid)](https://en.wikipedia.org/wiki/Rigid_transformation) |
|-------|-------|-------|-------|-------|
| Constraints | $$c_2 = 1$$ | Projective<br>$$c_1 = c_2 = 0$$ | Affine<br>$$a_0 = b_1$$, $$b_0 = -a_1$$ | Similarity<br>$$\begin{aligned} &\cos^{-1}(a_0) \\ &= \sin^{-1}(-a_1) \\ &= \sin^{-1}(b_0) \\ &= \cos^{-1}(b_1)\end{aligned}$$ |
| # Free Parameters | 8 | 6 | 4 | 3 |
| # Defining Points | 4 | 3 | 2 | <span style="color:red">?</span> |
| Rotation | Y | Y | Y | Y |
| Translation | Y | Y | Y | Y |
| Scaling | Y | Y | Y (same in x- and y-directions) | N |
| Shear | Y | Y | N | N |
| Geometric interpretation | Maps lines to lines |  Preserves lines and parallelism | Preserves shape | Preserves Euclidean distance between points |

Let $$S \in \mathbb{R}^{3 \times n}$$ be a set of $$n$$ [homogeneous coordinates](https://en.wikipedia.org/wiki/Homogeneous_coordinates) in the source image and $$D \in \mathbb{R}^{3 \times n}$$ be a set of corresponding homogeneous coordinates in the destination image.

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

For pixel-based images, $$x$$ usually refers to the column index of a pixel, and $$y$$ usually refers to the row index of a pixel, with $$(x, y) = (1, 1)$$ (or sometimes $$(0, 0)$$) denoting the location of the pixel in the top-left corner of the image.

We want to learn the tranform matrix $$T$$ such that $$T \cdot S \approx D$$.

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

Consequently, we have a set of $$2n$$ linear equations that can be expressed in matrix form as $$0_{2n} = A \cdot x$$ where

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

Therefore, to have a fully-determined or overdetermined system of linear equations, the number of corresponding points $$n$$ must be at least half the number of free parameters (see the table above).

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

### Pixel aspect ratios

Let $$r$$ be the pixel aspect ratio:

$$r = \frac{\text{width represented by a pixel}}{\text{height represented by a pixel}}$$

Usually, we work with square ($$r = 1$$) pixels. However, images (especially those generated by different modalities, such as mass spectrometry imaging, MRI, etc.) may have non-square pixels. For example, a pixel that corresponds to physical dimensions of 1 cm width x 2 cm height would have a pixel aspect ratio of 0.5.

Consider a source image $$\mathcal{I}_S$$ and a destination image $$\mathcal{I}_D$$, which may be of different dimensions. Let $$r_S, r_D$$ be the aspect ratios of pixels in the source and destination images, respectively. We want to generate an image $$\mathcal{I}_{S'}$$ of the same dimension as the destination image $$\mathcal{I}_D$$ in which the source image is aligned to the destination image.

Let $$S$$ and $$D$$ be corresponding homogeneous coordinates as defined above. Let $$\tilde{r} = \frac{r_D}{r_S}$$ be the ratio of pixel aspect ratios. Then,

$$
D' = \begin{bmatrix}
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

Let $$T \in \mathbb{R}^{3 \times 3}$$ be a transform mapping coordinates $$S$$ to $$D'$$:

$$T \cdot S = D'$$

If $$T$$ is an affine transformation, then we can solve for $$T$$ as

$$T = D' \cdot \begin{cases}
S^{-1} & S \text{ is invertible} \\
S^\dagger & \text{otherwise}
\end{cases}$$

as discussed above. If $$T$$ is desired to be a more constrained type of transformation, $$T$$ can be estimated using other established algorithms. Of course, gradient descent can also be used to solve for $$T$$, with constraints enforced by strong regularization terms.

At this point, we have a transform $$T$$ that maps $$S$$ to $$D'$$, but not to $$D$$ as desired. To account for the pixel aspect ratios, observe that

$$
T \cdot S = \begin{bmatrix}
1 & 0 & 0 \\
0 & \frac{1}{\tilde{r}} & 0 \\
0 & 0 & 1
\end{bmatrix} \cdot D \rightarrow
\begin{bmatrix}
1 & 0 & 0 \\
0 & \tilde{r} & 0 \\
0 & 0 & 1
\end{bmatrix} \cdot T \cdot S = \tilde{T} \cdot S = D
$$

For projective and affine transformation matrices, the scaling matrix can be folded into the final transformation matrix

$$\tilde{T} = \begin{bmatrix}
1 & 0 & 0 \\
0 & \tilde{r} & 0 \\
0 & 0 & 1
\end{bmatrix} T$$

without violating the constraints. In fact, the idea of "constraining" a projective or affine transformation to respect pre-defined pixel aspect ratio $$\tilde{r}$$ is not possible. The final learned transformation matrix $$\tilde{T}$$ can just as well be fit on $$S$$ and $$D$$ directly, without first having to scale $$D$$.

For more constrained transformation matrices, the final transformation matrix cannot be learned directly from $$S$$ and $$D$$ without violating assumed constraints on $$T$$.

### A note on implementation

In Python, the transform functions of popular image libraries like [Pillow](https://pillow.readthedocs.io/en/stable/reference/Image.html?highlight=image.transform#PIL.Image.Image.transform) and [scikit-image](https://scikit-image.org/docs/stable/api/skimage.transform.html#skimage.transform.warp) ask for the inverse transform matrix $$\tilde{T}^{-1}$$ such that $$\tilde{T}^{-1} \cdot D = S$$. Observe that

$$
\tilde{T}^{-1}
= \left(\begin{bmatrix}
    1 & 0 & 0 \\
    0 & \tilde{r} & 0 \\
    0 & 0 & 1
\end{bmatrix} T \right)^{-1}
= T^{-1} \begin{bmatrix}
    1 & 0 & 0 \\
    0 & \frac{1}{\tilde{r}} & 0 \\
    0 & 0 & 1
\end{bmatrix}
$$

The reason why the inverse transform matrix is desired is because of how interpolation is performed. By mapping the destination coordinates to the source coordinates, we can ask for the pixel value at, for example, location (1.5, 1.7) in the source image (see [`scipy.ndimage.map_coordinates()`](https://docs.scipy.org/doc/scipy/reference/generated/scipy.ndimage.map_coordinates.html)). Since pixel indices must be integers, so (1.5, 1.7) is interpolated (e.g., using nearest-neighbor, bilinear, bicubic, etc.) from neighboring/nearby pixels.

<script src="https://gist.github.com/bentyeh/eea647fe51f8260c6dc5766f07cbd088.js"></script>

## References

- scikit-image documentation
  - [User Guide: Geometrical transformations of images](https://scikit-image.org/docs/stable/user_guide/geometrical_transform.html)
  - [Example: Using geometric transforms](https://scikit-image.org/docs/stable/auto_examples/transform/plot_geometric.html)
  - [Example: Types of homographies](https://scikit-image.org/docs/stable/auto_examples/transform/plot_transform_types.html)
- Difference between affine and projective transformations: [StackOverflow](https://math.stackexchange.com/questions/1319680/what-is-the-difference-between-affine-and-projective-transformations)
- Learning a projective transform matrix from 4 points: [StackExchange Math](https://math.stackexchange.com/questions/296794/finding-the-transform-matrix-from-4-projected-points-with-javascript)