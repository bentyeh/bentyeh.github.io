---
title: Replicates and variance
layout: post
use_code: true
use_toc: true
use_math: true
hidden: true
excerpt: We consider biological and technical variance, and how to estimate each type of variance from experimental data.
tags: experimental-design
category: statistics
---

## Problem

Consider the example introduced by the article

> Blainey, P., Krzywinski, M. & Altman, N. Points of significance: replication. *Nature Methods* 11, 879–80 (2014). [doi:10.1038/nmeth.3091](https://doi.org/10.1038/nmeth.3091).

While the article clearly explains the experimental setup from an *experimental* perspective, it lacks an *explicit* description of the statistical model, which I found confusing as a reader, and which I write out below.

### Experimental setup

Consider an experiment measuring single-cell expression level of a specific gene in liver cells in mice. The experiment is designed with three levels of replication: animals ($$n_A$$ replicates), cells ($$n_C$$ replicates), and measurements ($$n_M$$ replicates), for a total of $$n = n_A n_C n_M$$ data points.

### Statistical model

Let $$X$$ be the observed expression level (say, in units FPKM), which we model as

$$X = \mu + A + C + M$$

where
- $$\mu$$ is a constant representing the true mean value
- $$A, C, M$$ are random variables representing the contribution of a given animal, cell, and measurement to the observed expression level.

| Replicate level | Type of variation | Random variable | Mean | Variance      | Number of replicates |
| --------------- | ----------------- | --------------- | ---- | ------------- | -------------------- |
| animal          | biological        | $$A$$             | 0    | $$\sigma_A^2$$  | $$n_A$$                |
| cell            | biological        | $$C$$             | 0    | $$\sigma_C^2$$  | $$n_C$$                |
| measurement     | technical         | $$M$$             | 0    | $$\sigma_M^2$$  | $$n_M$$                |

We assume the variance introduced at each level to be independent (i.e., $$A, C, M$$ are independent), so total variance is given by $$\mathrm{Var}(X) = \sigma_X^2 = \sigma_A^2 + \sigma_C^2 + \sigma_M^2$$.

We consider the following objectives for a given dataset $$\mathcal{D} = \{X_1 = x_1, ..., X_n = x_n\}$$:
- Variance estimation.
  - Estimate the total variance $$\mathrm{Var}(X)$$.
  - Estimate each source of variance ($$\sigma_A^2$$, $$\sigma_C^2$$, and $$\sigma_M^2$$).
- Compute the effective number of independent samples.

## Variance estimation

### Total variance

The unbiased sample variance estimator

$$\hat{\sigma}_X^2 = \frac{1}{n-1} \sum_{i=1}^n (x_i - \bar{x})^2$$

may underestimate the total variance if the samples are not independent (i.e., if each sample is not from a different animal). For example, if all of our samples come from different cells in the same animal ($$n = n_C$$, $$n_A = n_M = 1$$), then $$\hat{\sigma}_X^2$$ will only reflect cell and measurement variances but not animal variance.

<span style="color: red">
Q1. In this case, would it be true that $$\mathbb{E}(\hat{\sigma}_X^2) = \sigma_C^2 + \sigma_M^2$$?
</span>

<span style="color: red">
Q2. Is there any way to unbiasedly estimate $$\hat{\sigma}_X^2$$ if $$n_A \neq n$$?
</span>

### Individual variances

<span style="color: red">
Q1. Is there a general method for this?
</span>

<span style="color: red">
Q2. Let $$n_A = 1$$, $$n_C = 2$$, $$n_M = 2$$ (see table below). Let $$\bar{x}_{12} = \frac{x_1 + x_2}{2}$$ and $$\bar{x}_{34} = \frac{x_3 + x_4}{2}$$. Then is this a valid way to estimate $$\sigma_M^2$$?
</span>

<div style="color: red">
$$\hat{\sigma}_M^2 = \frac{1}{3} \left( (x_1 - \bar{x}_{12})^2 + (x_2 - \bar{x}_{12})^2 + (x_3 - \bar{x}_{34})^2 + (x_4 - \bar{x}_{34})^2 \right)$$
</div>

| i | A | C | M |
| - | - | - | - |
| 1 | 0 | 0 | 0 |
| 2 | 0 | 0 | 1 |
| 3 | 0 | 1 | 0 |
| 4 | 0 | 1 | 1 |

<span style="color: red">
Q3. Let's say we're able to individually compute $$\hat{\sigma}_A^2, \hat{\sigma}_C^2, \hat{\sigma}_M^2$$. How good of an estimate of $$\sigma_X^2$$ would be $$\hat{\sigma}_A^2 + \hat{\sigma}_C^2 + \hat{\sigma}_M^2$$?
</span>

## Effective sample size

Let $$G$$ be the true distribution of $$X$$, i.e., $$X \sim G$$. Let $$X_i$$ ($$i = 1, ..., n$$) be samples from this distribution $$G$$, and $$\bar{X} = \frac{1}{n} \sum_{i=1}^n X_i$$ be the sampling mean. Then

$$\begin{aligned}
\mathrm{Var}(\bar{X})
&= \mathrm{Var}\left( \frac{1}{n} \sum_{i=1}^n X_i \right) \\
&= \frac{1}{n^2} \mathrm{Var}\left( \sum_{i=1}^n X_i \right) \\
&= \frac{1}{n^2} \left( \sum_{i=1}^n \mathrm{Var}(X_i) + 2 \sum_{i=1}^{n-1} \sum_{j=i+1}^n \mathrm{Cov}(X_i, X_j) \right) \\
&= \frac{1}{n} \mathrm{Var}(X) + \frac{2}{n^2} \sum_{i=1}^{n-1} \sum_{j=i+1}^n \mathrm{Cov}(X_i, X_j) \\
\end{aligned}$$

If each $$X_i$$ are independent (i.e., each sample is from a different animal; see [Notes](#notes)), then all the covariance terms are zero and $$\mathrm{Var}(X) = n \mathrm{Var}(\bar{X})$$. Blainey et al. therefore suggest using the ratio of $$\mathrm{Var}(\bar{X})$$ to $$\mathrm{Var}(X)$$ as a measure of the effective number of independent samples.

### Estimating effective sample size

$$n_\text{eff} = \frac{\mathrm{Var}(X)}{\mathrm{Var}(\bar{X})}$$

$$\mathrm{Var}(X)$$ can be estimated as [above](#total-variance).

$$\mathrm{Var}(\bar{X})$$ is harder to estimate. Given $$\sigma_A^2, \sigma_C^2, \sigma_M^2$$ (either known/guessed *a priori* or [estimated from the dataset](#individual-variances)), we can calculate 

$$\mathrm{Var}(\bar{X}) = \frac{\sigma_A^2}{n_A} + \frac{\sigma_C^2}{n_A n_C} + \frac{\sigma_M^2}{n_A n_C n_M}$$

<details markdown="block"><summary>Derivation</summary>

This equation is given in Blainey et al. without proof, so here I present a derivation. Let $$a(i) \in [1, n_A]$$, $$c(i) \in [1, n_C]$$, and $$m(i) \in [1, n_M]$$ map sample $$i \in [1, n]$$ to its corresponding animal, cell, and measurement.

$$\begin{aligned}
\bar{X} &= \frac{1}{n} \sum_{i=1}^n X_i \\
&= \frac{1}{n} \sum_{i=1}^n \mu + A_{a(i)} + C_{a(i), c(i)} + M_{a(i), c(i), m(i)} \\
&= \mu + \frac{1}{n} \sum_{a=1}^{n_A} A_a n_C n_M + \frac{1}{n} \sum_{a=1}^{n_A} \sum_{c=1}^{n_C} C_{a,c} n_M + \frac{1}{n} \sum_{a=1}^{n_A} \sum_{c=1}^{n_C} \sum_{m=1}^{n_M} M_{a,c,m} \\
&= \mu + \frac{1}{n_A} \sum_{a=1}^{n_A} A_a + \frac{1}{n_A n_C} \sum_{a=1}^{n_A} \sum_{c=1}^{n_C} C_{a,c} + \frac{1}{n_A n_C n_M} \sum_{a=1}^{n_A} \sum_{c=1}^{n_C} \sum_{m=1}^{n_M} M_{a,c,m}
\end{aligned}$$

$$\begin{aligned}
\mathrm{Var}(\bar{X}) &= \mathrm{Var} \left(\mu + \frac{1}{n_A} \sum_{a=1}^{n_A} A_a + \frac{1}{n_A n_C} \sum_{a=1}^{n_A} \sum_{c=1}^{n_C} C_{a,c} + \frac{1}{n_A n_C n_M} \sum_{a=1}^{n_A} \sum_{c=1}^{n_C} \sum_{m=1}^{n_M} M_{a,c,m} \right) \\
&= \frac{1}{n_A^2} \mathrm{Var} \left( \sum_{a=1}^{n_A} A_a \right) + \frac{1}{n_A^2 n_C^2} \mathrm{Var} \left( \sum_{a=1}^{n_A} \sum_{c=1}^{n_C} C_{a,c} \right) + \frac{1}{n_A^2 n_C^2 n_M^2} \mathrm{Var} \left( \sum_{a=1}^{n_A} \sum_{c=1}^{n_C} \sum_{m=1}^{n_M} M_{a,c,m} \right) \\
&= \frac{1}{n_A^2} n_A \mathrm{Var} \left( A_a \right) + \frac{1}{n_A^2 n_C^2} n_A n_C \mathrm{Var} \left( C_{a,c} \right) + \frac{1}{n_A^2 n_C^2 n_M^2} n_A n_C n_M \mathrm{Var} \left( M_{a,c,m} \right) \\
&= \frac{\sigma_A^2}{n_A} + \frac{\sigma_C^2}{n_A n_C} + \frac{\sigma_M^2}{n_A n_C n_M}
\end{aligned}$$

</details>


<span style="color: red">
Q1. Assuming we can estimate $$\sigma_A^2, \sigma_C^2, \sigma_M^2$$ from the data, how good of an estimate of $$\mathrm{Var}(\bar{X})$$ is
</span>
<div style="color: red">
$$\frac{\hat{\sigma}_A^2}{n_A} + \frac{\hat{\sigma}_C^2}{n_A n_C} + \frac{\hat{\sigma}_M^2}{n_A n_C n_M}$$
</div>

<span style="color: red">
Q2. How good of an estimate of $$n_\text{eff}$$ is
</span>
<div style="color: red">
$$\frac{\hat{\mathrm{Var}}(X)}{\hat{\mathrm{Var}}(\bar{X})}$$
</div>
<span style="color: red">
Is it unbiased?
</span>

## Notes
- Blainey et al. use the symbol $$\sigma_\text{TOT}^2$$ instead $$\sigma_X^2$$.
- We could alternatively (and equivalently) choose to fold the true mean value $$\mu$$ into one or more of the means of $$A, C, M$$. For example, we could instead have formulated the model as

  $$X = A + C + M$$

  where $$\mu_A = \mu_C = \mu_M = \frac{1}{3} \mu$$. However, I think the formulation where $$\mu_A = \mu_C = \mu_M = 0$$ best captures the idea that each level of replication contributes some noise on top of the underlying true mean $$\mu$$.
- The analyses presented here do not assume any particular distribution (e.g., Gaussian, etc.) for $$A, C, M$$. However, for the purpose of simulation (e.g., Figure 1b in Blainey et al.), a distribution must be assumed.
- By defining $$X_i$$ as independent samples, we are implying that each sample is from a different animal, i.e., $$n_A = n$$, $$n_C = 1$$, and $$n_M = 1$$. Consider otherwise: if two samples $$X_j, X_k$$ came from the same animal, then the contribution of the animal to the variation observed in gene expression is the same, $$A_{a(j)} = A_{a(k)}$$. Knowing the expression value of one sample $$X_j$$ would allow us to more accurately estimate the expression value of the other sample, $$X_k$$. (<span style="color: red">What would be a rigorous proof of $$X_j$$ being dependent on $$X_k$$ if we know that $$A_{a(j)} = A_{a(k)}$$?</span>)

## Reproducing figures

Here's some Python code to reproduce Figure 1B.

<script src="https://gist.github.com/bentyeh/9404b48aab739308751ebcf64150c42f.js"></script>