---
title: Cell-mixture deconvolution and applications
layout: post
use_toc: true
use_math: true
excerpt: An introduction to cell mixture deconvolution and applications based on material taught in Stanford IMMUNOL 207, Spring 2020.
hidden: true
---

# Deconvolution

<img style="width: 100%" src="{{ site.baseurl }}/images/posts_deconvolution_matrices.svg">

We can model gene expression profiles by a system of linear equations: each sample (a bulk mixture expression profile) is a linear combination of the expression profiles of different cell types. Past studies have shown that such linear models are biologically plausible. [See references cited by the [CIBERSORT](#references) paper.] The symbols and notation used in this article follow those in the [CIBERSORTx](#references) paper. For notational convenience, we refer to the $$j$$th columns of the $$M$$ and $$F$$ matrices by $$\mathbf{m_j}$$ and $$\mathbf{f_j}$$, respectively.

Given any two of the matrices, we can estimate the third matrix. Each of the three estimation problems corresponds to a different biological question. The **Requires** column in the table below gives the necessary conditions for the system to be overdetermined.

| Problem                                                                     | Given    | Estimate | Requires  |
| --------------------------------------------------------------------------- | -------- | -------- | --------- |
| Estimate cell type proportions from bulk profile and signature matrix       | $$M, H$$ | $$F$$    | $$n > c$$ |
| Generate signature matrix from bulk profile and known cell type proportions | $$M, F$$ | $$H$$    | $$k > c$$ |
| Estimate bulk profile from signature matrix and cell type proportions       | $$H, F$$ | $$M$$    | none      |

1. In the first problem, each column (sample) of the sample proportions matrix is estimated independently, so the number of unknowns is the number of cell types $$c$$. Assuming the signature matrix is full rank, the number of linearly independent equations is $$n$$. Therefore, for the system to be overdetermined, we need $$n > c$$.
2. In the second problem, each row (gene) of the signature matrix is estimated independently, so the number of unknowns is the number of cell types $$c$$. Assuming the sample proportions matrix is full rank, the number of linearly independent equations is $$k$$. Therefore, for the system to be overdetermined, we need $$k > c$$.
   - <span style="color: red">Are there ways to estimate rows <i>dependently</i> and take advantage of existing gene-gene correlation patterns?</span>
3. In the third problem, the bulk profile estimate is a simple matrix multiplication of the signature matrix and the sample proportions matrix.

In general, there are two ways of evaluating deconvolution accuracy:
1. Accuracy of prediction: compare predicted and experimentally-determined values of a matrix.
   - Example: Suppose we have a blood sample and want to determine the relative proportions $$\mathbf{f}$$ of blood cell types (i.e., an instance of problem 1 in the table). An aliquot of the blood sample could be profiled (e.g., via RNA-seq) in bulk to generate $$\mathbf{m}$$; then, given a signature matrix $$H$$ (e.g., LM22), we can [estimate](#problem-1-estimating-cell-type-proportions) $$\mathbf{\hat{f}}$$. A separate aliquot could be sorted via flow-cytometry to generate experimentally-determined cell type proportions $$\mathbf{f}_\text{exp}$$. We can then compare $$\mathbf{\hat{f}}$$ with $$\mathbf{f}_\text{exp}$$ using Pearson correlation coefficient, RMSE, etc.
2. Accuracy of matrix factorization: compute the reconstruction error using the estimated matrix.
   - Example: Continuing the previous example, we could estimate (reconstruct) the bulk profile $$\mathbf{\hat{m}}$$ using the signature matrix $$H$$ (i.e., $$\mathbf{\hat{m}} = H \times \mathbf{\hat{f}}$$) and compare $$\mathbf{m}$$ with $$\mathbf{\hat{m}}$$ using Pearson correlation coefficient, RMSE, etc.

# Methods

## Problem type 1: Estimating cell type proportions

### Simple Linear Regression

Any linear regression technique (ordinary least squares, lasso regression, ridge regression, etc.) could be used in this context. The formula for ordinary least squares is given here for reference.

$$\mathbf{\hat{f}} = H^\dagger \mathbf{m} = \left(H^\top H\right)^{-1} H^\top \mathbf{m}$$

Some potentially useful constraints that these methods ignore include the following:
1. Cell-type proportions $$f$$ should be non-negative.
2. It may be desirable that the elements of $$f$$ sum to 1.

### CIBERSORT

Acronym: cell-type identification by estimating relative subsets of RNA transcripts

CIBERSORT uses linear $$\nu$$-support vector regression ($$\nu$$-SVR), which is a more robust estimation method. Since estimation only depends on a set of support vectors chosen from the set of $$n$$ genes in the signature matrix (which itself is usually a curated subset of all measurable genes), the algorithm
> incorporate[s] an additional round of feature selection to adaptively select genes from an existing signature matrix [[CIBERSORT paper](#references)]

CIBERSORT evaluates the significance of the estimation against a null hypothesis that no cell types in the signature matrix are present in the sample. The null distribution of reconstruction accuracies is generated as follows:
1. Randomly sample (with replacement) $$\mathbf{m}$$ to generate $$\mathbf{m^*}$$.
   - <span style="color: red">Why does this simulate a null hypotehsis that no cell types in the signature matrix are present in the sample?</span>
2. Estimate the corresponding cellular fractions $$\mathbf{f^*}$$ using CIBERSORT.
3. Measure the accuracy (Pearson correlation) of the reconstruction between $$\mathbf{m^*}$$ and $$H \mathbf{f^*}$$.

Here's an example of generating the null distribution, where random numbers were used to create $$H$$ and $$m$$.

```{python}
import numpy as np
import matplotlib.pyplot as plt
n = 10
c = 3
H = np.random.rand(n, c)
assert np.linalg.matrix_rank(H) == c
H_pinv = np.linalg.pinv(H)
f = np.random.rand(c)
f /= np.sum(f)
m = H @ f
m_stars = np.array([np.random.choice(m, len(m)) for _ in range(500)])
f_stars = np.array([H_pinv @ m_star for m_star in m_stars])
corrs = np.array([np.corrcoef(H @ f_stars[i], m_stars[i])[0, 1] for i in range(len(m_stars))])
plt.hist(corrs, bins=25)
```

## Problem type 2: Estimating sample-specific signature matrices

Signature matrices can be generated independently of a deconvolution method by combining expression profiles and selecting cell-type specific genes. Established basis matrices for immune cell types include IRIS (2005), LM22 (2015), and immunoStates (2018). This article does not attempt to explain how to generate such basis matrices, and we refer the reader to the [immunoStates paper](#references) for a more in-depth discussion.

Instead, we consider the problem of estimating sample-specific signature matrices $$H^{(j)} \in \mathbb{R}^{n \times c}$$ given a bulk matrix $$M$$ and known cell type proportions $$F$$. We motivate this problem by considering a short-coming of using a single signature matrix $$H$$ for all samples, which may come from different biological conditions (e.g., diseased versus healthy). Since the gene expression profile of the same cell type may change under different conditions, we may want to (re)construct sample-specific (or at least, condition-specific) signature matrices.

Then, by comparing these estimated signature matrices, we may be able to better understand how cell-type-specific gene expression changes across biological conditions.

In both the naive solution and the CIBERSORTx methods, we treat each gene independently.

### Naive solution

A naive solution independently fits a signature matrix $$\hat{H}^{(j)}$$ for each sample $$j = 1, ..., k$$. However, simple linear methods, e.g., $$\hat{H}^{(j)} \approx \mathbf{m}_j \times \mathbf{f}_j^\dagger$$, result in an underdetermined system of linear equations: since each sample is modeled independently of other samples, each subproblem has $$k = 1$$, thus failing the requirement that $$k > c$$.

### CIBERSORTx

Consider the linear equations for gene $$i$$ across all samples

$$\begin{aligned}
M_{i, \bullet}
&= \begin{bmatrix}
    H^{(1)}_{i,\bullet} \cdot \mathbf{f}_1 &
    \cdots & 
    H^{(k)}_{i,\bullet} \cdot \mathbf{f}_k
    \end{bmatrix} \\
&= \text{diag}\left(\begin{bmatrix}
    H^{(1)}_{i,\bullet} \cdot \mathbf{f}_1 & H^{(1)}_{i,\bullet} \cdot \mathbf{f}_2 & \cdots & H^{(1)}_{i,\bullet} \cdot \mathbf{f}_k \\
    H^{(2)}_{i,\bullet} \cdot \mathbf{f}_1 & \ddots & & \vdots \\
    \vdots & & \ddots & \\
    H^{(k)}_{i,\bullet} \cdot \mathbf{f}_1 & \cdots & & H^{(k)}_{i,\bullet} \cdot \mathbf{f}_k
    \end{bmatrix}\right) \\
&= \text{diag}\left(
    \begin{bmatrix}
        H^{(1)}_{i,\bullet} \\
        \vdots \\
        H^{(k)}_{i,\bullet}
    \end{bmatrix} \times 
    \begin{bmatrix}
        \mathbf{f}_1 & \cdots & \mathbf{f}_k
    \end{bmatrix}\right) \\
&= \text{diag}\left(G_{i,\bullet,\bullet} \times F\right)
\end{aligned}$$

This gives equation (3) in the CIBERSORTx paper and shows how the $$n \times k \times c$$ tensor $$G$$ can be thought of as a stack of $$k$$ sample-specific signature matrices. Then, $$\mathbf{m}_j = G_{\bullet, j, \bullet} \times \mathbf{f}_j$$.

Alternatively (and equivalently), $$G$$ can be thought of as a stack of $$c$$ cell-type-specific expression matrices of shape $$n \times k$$; this is the perspective presented in Figure 4.

In either case, the goal is to estimate $$G$$ given $$M$$ and $$F$$.

<span style="color: red">I'm confused by the CIBERSORTx paper. By following the "Overview of CIBERSORTx analytical framework" section, I came to understand the problem statement as estimating sample-specific signature matrices. However, Figures 4abd seem to suggest the alternative problem statement of estimating cell-type specific signature matrices. Which is correct? What are the biological interpretations of each of these problem statements? When would one be preferred over the other? What are the problem statements behind the estimates shown in Figures 5f, 6b, and 6c?</span>

However, we still have not addressed the problem of the naive formulation. To do so, CIBERSORTx uses an involved heuristic approach on top of non-negative least squares (NNLS) regression, which they validate on multiple synthetic and real datasets.

In conclusion, the authors assert that 

> variation in bulk gene expression data can be leveraged to infer latent phenotypic class structure in the underlying cell subpopulations. [[CIBERSORTx paper](#references)]

# References

1. Newman, A. M. et al. Robust enumeration of cell subsets from tissue expression profiles. *Nature Methods* 12, 453–457 (2015). [doi:10.1038/nmeth.3337](https://doi.org/10.1038/nmeth.3337).
   - CIBERSORT paper.
2. Newman, A. M. et al. Determining cell type abundance and expression from bulk tissues with digital cytometry. *Nature Biotechnology* 37, 773–782 (2019). [doi:10.1038/s41587-019-0114-2](https://doi.org/10.1038/s41587-019-0114-2).
   - CIBERSORTx paper.
3. Vallania, F. et al. Leveraging heterogeneity across multiple datasets increases cell-mixture deconvolution accuracy and reduces biological and technical biases. *Nature Communications* 9, 4735 (2018). [doi:10.1038/s41467-018-07242-6](https://doi.org/10.1038/s41467-018-07242-6).
   - immunoStates paper.