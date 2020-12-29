---
title: Statistical Models of Gene Expression Analysis
layout: post
use_toc: true
use_math: true
excerpt: Why is the negative binomial distribution used to model sequencing read counts? What do FPKM and TPM mean? What are common models of differential gene expression and pathway analysis?
last_updated: 2020-06-08
tags: [gene-expression]
---

$$
\newcommand{\abs}[1]{\left\lvert#1\right\rvert}
$$

# Probability distribution of counts

## Negative binomial distribution

$$K \sim \text{NB}(r, p)$$: number of failures until $$r$$ successes have occurred; $$p$$ is the probability of success

$$P(K = k) = {k + r - 1 \choose r - 1} p^r (1 - p)^k = {k + r - 1 \choose k} p^r (1 - p)^k$$

<details markdown="block"><summary>Derivation</summary>

Assumptions
1. All trials are independent.
2. The probability $$p$$ of sucesss stays the same from trial to trial.

Let $$A$$ be the event that in the first $$k + r - 1$$ trials, we observe $$r - 1$$ successes (or equivalently, $$k$$ failures). The Binomial distribution tells us that

$$P(A) = {k + r - 1 \choose r - 1} p^{r-1} (1 - p)^k = {k + r - 1 \choose k} p^{r-1} (1 - p)^k$$

Let $$B$$ be the event that the $$(k + r)$$ trial is a success.

$$P(B) = p$$

Since all trials are independent, $$A$$ and $$B$$ are independent events, so

$$P(K = k) = P(A \cap B) = P(A) P(B)$$

</details>

Notes and References
1. The negative binomial may alternatively be formulated as $$K + r = Y \sim \text{NB}(r, p)$$ = the number of the trial on which the $$r$$th success occurs. See Wackerly's *Mathematical Statistics with Applications*, 7th Edition, or Rice's *Mathematical Statistics and Data Analysis*, 3rd Edition.
2. [Wikipedia](https://en.wikipedia.org/wiki/Negative_binomial_distribution) uses "success" and "failure" oppositely of the formulation presented here. Swap $$p$$ and $$1 - p$$ for the equations to match.
3. For a reference that matches the formulation used here, see [https://www.johndcook.com/negative_binomial.pdf](https://www.johndcook.com/negative_binomial.pdf).

### Properties

$$\begin{aligned}
\mathbb{E}(K) &= \mu = \frac{r(1 - p)}{p} \\
\text{Var}(K) &= \sigma^2 = \frac{r(1 - p)}{p^2} = \mu + \frac{1}{r} \mu^2 = \mu + \alpha \mu^2
\end{aligned}$$

where $$\alpha = \frac{1}{r}$$ is called the **dispersion** parameter.

Then, we can re-express $$r$$ and $$p$$ in terms of $$\mu$$ and $$\sigma$$, or $$\mu$$ and $$\alpha$$:

$$\begin{aligned}
r &= \frac{\mu^2}{\sigma^2 - \mu} = \frac{1}{\alpha} \\
p &= \frac{\mu}{\sigma^2} = \frac{1}{1 + \alpha \mu}
\end{aligned}$$

Now, we can equivalently parameterize the negative binomial distribution as follows:
- Mean and variance: $$K \sim \text{NB}(\mu, \sigma^2)$$

  $$P(K = k) = {k + \frac{\mu^2}{\sigma^2 - \mu} - 1 \choose k} \left(\frac{\mu}{\sigma^2} \right)^{\frac{\mu^2}{\sigma^2 - \mu}} \left(\frac{\sigma^2 - \mu}{\sigma^2} \right)^k$$

- Mean and dispersion: $$K \sim \text{NB}(\mu, \alpha)$$

  $$P(K = k) = {k + \frac{1}{\alpha} - 1 \choose k} \left(\frac{1}{1 + \alpha \mu} \right)^{\frac{1}{\alpha}} \left(\frac{\alpha \mu}{1 + \alpha \mu} \right)^k$$

### Formulation as Gamma-Poisson

Idea: $$K$$ is a Poisson distribution where the mean of the Poisson distribution is proportional to a Gamma-distributed random variable.

$$K \sim \text{NB}(r, p)$$ is equivalent to $$K \mid \Lambda = \lambda \sim \text{Poi}(s\lambda)$$ where $$\Lambda \sim \text{Gamma}(a = r, \theta = \frac{1-p}{sp})$$
- []()$$p = \frac{1}{s\theta + 1}$$
- The $$a$$ parameter in the Gamma distribution is *not* the same as the $$\alpha$$ dispersion parameter.

$$
P(K = k \mid \Lambda = \lambda) = \frac{(s\lambda)^k e^{-s\lambda}}{k!}, \quad
P(\Lambda = \lambda) = \frac{1}{\theta^a \Gamma(a)} \lambda^{a-1} e^{-\frac{\lambda}{\theta}}
$$

<details markdown="block"><summary>Derivation</summary>

$$\begin{aligned}
P(K = k)
&= \int_{0}^{\infty} P(K = k, \Lambda = \lambda) d\lambda \\
&= \int_{0}^{\infty} P(K = k \mid \Lambda = \lambda) P(\Lambda = \lambda) d\lambda \\
&= \int_{0}^{\infty} \frac{(s\lambda)^k e^{-s\lambda}}{k!} \frac{1}{\theta^a \Gamma(a)} \lambda^{a-1} e^{-\frac{\lambda}{\theta}} d\lambda \\
&= \frac{s^k}{k! \Gamma(a)} \theta^{-a} \int_{0}^{\infty} \lambda^{k + a - 1} e^{-\lambda (\frac{s\theta + 1}{\theta})} d\lambda \\
&= \frac{s^k}{k! \Gamma(a)} \theta^{-a} \Gamma(k + a) \left(\frac{\theta}{s\theta + 1} \right)^{k + a} \underbrace{\int_{0}^{\infty} \frac{\lambda^{k + a - 1} e^{-\lambda (\frac{s\theta + 1}{\theta})} \left(\frac{s\theta + 1}{\theta} \right)^{k + a}}{\Gamma(k + a)} d\lambda}_{\text{integral over support of a Gamma distribution = 1}} \\
&= \frac{s^k}{k! \Gamma(a)} \theta^{-a} \Gamma(k + a) \left(\frac{\theta}{s\theta + 1} \right)^{k + a} \\
&= \frac{\Gamma(k + a)}{k! \Gamma(a)} \left(\frac{1}{s\theta + 1} \right)^a \left(\frac{s\theta}{s\theta + 1} \right)^k \\
&= \frac{(k + a - 1)!}{k! (a - 1)!} \left(\frac{1}{s\theta + 1} \right)^a \left(\frac{s\theta}{s\theta + 1} \right)^k \\
&= {k + a - 1 \choose k} \left(\frac{1}{s\theta + 1} \right)^a \left(\frac{s\theta}{s\theta + 1} \right)^k
\end{aligned}$$

</details>

## Count data

Let $$N$$ be the total number of RNA transcripts in a sample and $$p_i$$ be the real proportion of those transcripts belonging to gene $$i$$.

Consider a sequencing run that samples $$n$$ of those transcripts (i.e., $$n$$ is the total number of reads). Note that $$n$$ is usually very large ($$n > 10^5$$) and $$p_i$$ is usually very small ($$p_i < 0.01$$). (For example, a TPM value of $$10000$$ corresponds to a $$p_i$$ of $$0.01$$.)

Let $$k_i$$ be the observed read counts of gene $$i$$.

| Model             | $$\text{Binom}(n, p_i)$$      | $$\text{Poi}(\lambda_i = np_i)$$ | $$\text{NB}(r_i, \phi_i) = \text{Poi}(n P_i), P_i \sim \text{Gamma}(a = r_i, \theta = \frac{1-\phi_i}{n\phi_i})$$ |
| ----------------- | --------------------------- | ------------------------------ | -------------------------------------------------------------------------------------------------------- |
| $$\mathbb{E}(k_i)$$ | $$np_i$$                      | $$\lambda_i = np_i$$             | $$\frac{r_i(1-\phi_i)}{\phi_i} = n a \theta = n \mathbb{E}(P_i)$$
| $$\text{Var}(k_i)$$ | $$np_i (1-p_i) \approx np_i$$ | $$\lambda_i = np_i$$             | $$\frac{r_i(1-\phi_i)}{\phi_i^2} = a n \theta + a n^2 \theta^2 = \frac{n \mathbb{E}(P_i)}{\phi_i}$$

Notes
1. For the variance under the Binomial model, the approximation $$n p_i (1 - p_i) \approx n p_i$$ holds because $$p_i$$ is small.
2. Since $$n$$ is large and $$p_i$$ is small, the Poisson distribution accurately approximates the Binomial distribution, and we see that the means and variance under both models are the same.
3. The symbol $$p_i$$ used here is not the same as the symbol $$p$$ used in the previous section describing the negative binomial distribution, hence the use of $$\phi_i$$ in the table.

Negative binomial interpretation
- Gamma-Poisson interpretation: In the Binomial / Poisson models, we assume that $$p_i$$ is fixed (constant) for gene $$i$$ across all samples in the same condition. In a Gamma-Poisson model, we assume that $$p_i$$ is the value of a random variable $$P_i$$ whose distribution across samples in the given condition follows a Gamma distribution. [[DESeq paper]](#references)
  - We can think of a fixed $$p_i$$ as corresponding to technical replicates: samples are measurements of the same underlying population of $$N$$ transcripts, of which a proportion $$p_i$$ belong to gene $$i$$. Then, there should be no overdispersion, and the counts should follow a Poisson distribution.
  - We can think of a varying $$p_i$$ as corresponding to biological replicates: samples are measurements of different mice, etc., where the proportion of transcripts belonging to gene $$i$$ varies over the population of mice, etc., according to a Gamma distribution. [[Robinson & Oshlack]](#references) [[Bioramble blog]](#references)
- Direct interpretation: Each read is a trial, and the read (trial) is "successful" if it does *not* come from gene $$i$$. The negative binomial models the number of "failed" reads (reads from gene $$i$$) until $$r$$ "successful" reads (reads not from gene $$i$$) have been observed, given that $$\phi_i = 1 - p_i$$ is the probability of observing a "successful" read. Finally,

  $$
  \frac{r_i(1 - \phi_i)}{\phi_i}
  = \frac{r_i}{\phi_i} p_i
  = \frac{\text{# of reads not from gene $i$}}{\text{proportion of reads not from gene $i$}} p_i
  = (\text{# total reads}) p_i
  = np_i
  $$

- Empirical evidence: Empirically, the variability of read counts is larger than the Binomial and Poisson distributions allows and is better approximated by a Negative Binomial distribution. [[DESeq paper]](#references) [[Bioramble blog]](#references)

# Differential expression analysis

## Dataset

Dataset size
- $$n$$: number of genes
- $$m$$: number of samples
- $$c$$: number of conditions (including intercept)

Count matrix: $$K \in \mathbb{N}^{n \times m}$$
- Rows: genes
- Columns: samples
- $$K_{ij}$$: number of sequencing reads mapped to gene $$i$$ in sample $$j$$
  - When referring to the count matrix as a random variable (e.g., see the [DESeq2 model](#deseq2)), a capital letter $$K$$ is used. When referring to the observed count matrix (e.g., a real dataset), a lower-case $$k$$ is used.
  - Let $$N_j = \sum_{i=1}^n k_{ij}$$ be the total number of mapped reads in sample $$j$$.

### Model (aka design) matrix

$$X \in \{0,1\}^{m \times c}$$
- Rows: samples
- Columns: conditions
- Example: Each sample is a patient
  - Intercept: The intercept parameter is used to model gene expression for a healthy, non-drugged patient.
  - Drug: The patient was administered the drug (1) or not (0).
  - Diseased: The patient is diseased (1) or healthy (0).

| sample $$j$$ | intercept | $$x_{1j}$$: drug | $$x_{2j}$$: diseased |
| ------------ | --------- | ---------------- | -------------------- |
| 1            | 1         | 0                | 0                    |
| 2            | 1         | 1                | 0                    |
| 3            | 1         | 0                | 1                    |
| 4            | 1         | 1                | 1                    |

[]()

If the normalized count $$q_{ij}$$ of gene $$i$$ is modeled as

$$\log_2 q_{ij} = \beta_{i0} + x_{1j} \beta_{i1} + x_{2j} \beta_{i2} + x_1 x_2 \beta_{i12}$$

then the $$\beta_{il}$$ parameters give the $$\log_2$$ fold change due to factor $$l$$ above control, where the control is modeled with just the intercept term. In the example above, sample 1 ($$x_{11} = x_{21} = 0$$) would be considered the control with $$\log_2 q_{i1} = \beta_{i0}$$.

Then, if we consider sample 2 ($$x_{12} = 1, x_{22}= 0$$), for example, we see that $$\beta_{i1}$$ gives the $$\log_2$$ fold change in expression of gene $$i$$ for a patient treated with the drug versus a control patient.

$$\begin{gathered}
\log_2 q_{i2} = \beta_{i0} + \beta_{i1} \\
\rightarrow \beta_{i1} = \log_2 q_{i2} - \beta_{i0}
= \log_2 q_{i2} - \log_2 q_{i1}
= \log_2 \frac{q_{12}}{q_{i1}}
\end{gathered}$$

Similarly, if we consider sample 4 ($$x_{14} = 1, x_{24}= 1$$), for example, we see that $$\beta_{i12}$$ is the $$\log_2$$ fold change due to the interaction of the drug with disease.

$$\begin{aligned}
\beta_{i12} &= \log_2 q_{i4} - (\beta_{i0} + \beta_{i1} + \beta_{i2}) \\
&= \log_2 q_{i4} - \left(\log_2 q_{i1} + \log_2 \frac{q_{12}}{q_{i1}} + \log_2 \frac{q_{13}}{q_{i1}} \right) \\
&= \log_2 q_{i4} - \left(\log_2 \frac{q_{i2} q_{i3}}{q_{i1}} \right) \\
&= \log_2 \frac{q_{i1} q_{i4}}{q_{i2} q_{i3}}
\end{aligned}$$

## Normalization

Raw alignment counts are often not directly comparable across samples or across genes due to the following biases:
- library size (sequencing depth): larger library sizes result in higher counts for the entire sample.
- uniquely high counts of a few genes can take up a large proportion of the sequencing "real estate," leading to relatively lower counts for the remaining genes. [[Robinson & Oshlack]](#references)
- gene length: longer genes are more likely to be "sampled" (sequenced) by RNA-seq.
  - This is an important concern when comparing the expression of different genes *within the same sample* (e.g., whether the expression of GLUT2 (glucose transporter 2) or GLUT4 (glucose transporter 4) higher in a particular sample).
  - This is not a significant concern for differential gene expression analysis, since gene length should exert the same relative bias across samples.

This article presents a few popular methods. The first three methods (naive normalization, median-of-ratios, and TMM) estimate a sample-specific factor $$s_j$$ yielding normalized counts $$q_{ij} = \frac{\mathbb{E}(K_{ij})}{s_j}$$ that can be compared *across* samples. There also exist methods for estimating sample- and *gene*-specific factors $$s_{ij}$$ such that $$q_{ij} = \frac{\mathbb{E}(K_{ij})}{s_{ij}}$$, but they are not discussed here. Additionally, spike-in controls (known quantities of artificial fragments) can be used to specifically address technical variability in library preparation and sequencing and are especially important when the assumption that the majority of the genes are not differentially expressed is invalid. However, spike-in controls may suffer from their own reliability issues and rely on the strict assumption that technical biases affect spike-ins in the same way as they do genes. [[Risso et al.]](#references) The gene-length dependent metrics (RPKM, FPKM, and TPM) estimate normalized proportions best used for comparisons *within* samples.

For further readings, consider the following review articles:
- Dillies, M.-A. et al. A comprehensive evaluation of normalization methods for Illumina high-throughput RNA sequencing data analysis. *Briefings in Bioinformatics* 14, 671-83 (2013). [doi:10.1093/bib/bbs046](https://doi.org/10.1093/bib/bbs046).
- Liu X. et al. Normalization Methods for the Analysis of Unbalanced Transcriptome Data: A Review. *Frontiers in Bioengineering and Biotechnology* 7 (2019). [doi:10.3389/fbioe.2019.00358](https://doi.org/fbioe.2019.00358).

### Naive normalization

Assumptions / limitations
- Only corrects for library size
- Each gene in a sample is normalized by the same factor: $$s_{ij} = s_j$$

Method: Divide by the total count of (mapped) sequencing reads of a sample, then multiply by the mean total count across all samples. The multiplication step ensures that the normalized values are counts (as assumed by [models](#model) like DESeq2) rather than proportions.

$$\hat{s}_j = \frac{N_j}{\bar{N}}, \quad \bar{N} = \frac{1}{m} \sum_{j=1}^m N_j = \frac{1}{m} \sum_{i=1}^n \sum_{j=1}^m k_{ij}$$

Problem: This potentially reduces the power of the method to pick out differentially expressed genes, and in extreme cases, might lead to false positives. Consider the following dataset where gene C is knocked-out in sample 3. Using the normalized counts $$q_{ij}$$, one might erroneously conclude that genes A, B, D, and E are upregulated in sample 3 relative to samples 1 and 2.

| gene          | sample 1 | sample 2 | sample 3 | $$q_{i1}$$ | $$q_{i2}$$ | $$q_{i3}$$ | 
|---------------|----------|----------|----------|------------|------------|------------| 
| A             | 100      | 90       | 125      | 96.7       | 96.7       | 120.8      | 
| B             | 200      | 180      | 250      | 193.3      | 193.3      | 241.7      | 
| C             | 300      | 270      | 0        | 290        | 290        | 0          | 
| D             | 400      | 360      | 500      | 386.7      | 386.7      | 483.3      | 
| E             | 500      | 450      | 625      | 483.3      | 483.3      | 604.2      | 
| Total         | 1500     | 1350     | 1500     |            |            |            |
| $$\hat{s}_j$$ | 1.03     | 0.93     | 1.03     |            |            |            |

More robust alternatives to normalizing to the total count include normalizing to the upper quartile (75th percentile) or median (50th percentile) of nonzero counts.

### Median-of-ratios

This is the default normalization method in DESeq and DESeq2 (`DESeq2::estimateSizeFactors()`).

Assumptions / limitations
- Corrects for library size and uniquely high counts
- Most genes are not differentially expressed

Method: Consider a pseudo-reference sample $$K^R$$ whose counts for each gene are obtained by taking the geometric mean across samples. The size factor $$\hat{s}_j$$ for sample $$j$$ is computed as the median of the ratios of the $$j$$-th sample's counts to those of the pseudo-reference:

$$
\hat{s}_j = \text{median}_{i: K_i^R > 0} \frac{K_{ij}}{K_i^R}, \quad
K_i^R = \left(\prod_{j=1}^m K_{ij} \right)^\frac{1}{m} = \exp\left(\frac{1}{m} \sum_{j=1}^m \log K_{ij} \right)
$$

Example: Using the same example as above, the normalized counts of genes A, B, D, and E are the same across samples 1, 2, and 3, as desired.

| gene        | sample 1 | sample 2 | sample 3 | $$K_i^R$$ | $$\frac{K_{i1}}{K_i^R}$$ | $$\frac{K_{i2}}{K_i^R}$$ | $$\frac{K_{i3}}{K_i^R}$$ | $$q_{i,1}$$ | $$q_{i,2}$$ | $$q_{i,3}$$ | 
|-------------|----------|----------|----------|---------|------------------------|------------------------|------------------------|-----------|-----------|-----------| 
| A           | 100      | 90       | 125      | 104.00  | 0.96                   | 0.87                   | 1.20                   | 104.00    | 104.00    | 104.00    | 
| B           | 200      | 180      | 250      | 208.01  | 0.96                   | 0.87                   | 1.20                   | 208.01    | 208.01    | 208.01    | 
| C           | 300      | 270      | 0        |         |                        |                        |                        | 312.01    | 312.01    | 0.00      | 
| D           | 400      | 360      | 500      | 416.02  | 0.96                   | 0.87                   | 1.20                   | 416.02    | 416.02    | 416.02    | 
| E           | 500      | 450      | 625      | 520.02  | 0.96                   | 0.87                   | 1.20                   | 520.02    | 520.02    | 520.02    | 
| Total       | 1500     | 1350     | 1500     |         |                        |                        |                        | 1560.06   | 1560.06   | 1248.05   | 
| $$\hat{s}_j$$ |          |          |          |         | 0.96                   | 0.87                   | 1.20                   |           |           |           | 

[]()

### Trimmed mean of M values (TMM)

This is the default normalization method in the edgeR R package (`edgeR::calcNormFactors()`).

> A trimmed mean is the average after removing the upper and lower x% of the data. The TMM procedure is doubly trimmed, by log-fold-changes $$M_{ij}^r$$ (sample $$j$$ relative to sample $$r$$ for gene $$i$$) and by absolute intensity ($$A_i$$). By default, we trim the $$M_i$$ values by 30% and the $$A_i$$ values by 5%. [[Adapted from Robinson & Oshlack]](#references)

Symbols
- $$M_{ij}^r = \log_2 \frac{K_{ij} / N_j}{K_{ir} / N_r}$$: gene-wise log-fold changes (sample $$j$$ relative to sample $$r$$ for gene $$i$$)
- $$A_{ij}^r = \log_2 \sqrt{\frac{K_{ij} \cdot K_{ir}}{N_j \cdot N_r}}$$: absolute expression (log-geometric mean of expression in samples $$j$$ and $$r$$)

Let $$M^*$$ and $$A^*$$ be the percentile thresholds for trimming genes by log-fold change and by absolute expression, respectively. For RNA-seq, [Robinson & Oshlack](#references) propose default values of $$M^* = 30$$ and $$A^* = 5$$. Let $$\mathcal{Q}$$ be the quantile function, i.e., $$\mathcal{Q}(v, p)$$ returns the $$p$$th quantile of vector $$v$$. Then, the set of untrimmed genes is 

$$\begin{aligned}
I^* = \{i \mid
&\left(\mathcal{Q}(M_{\bullet j}^r, M^*) < M_{ij}^r < \mathcal{Q}(M_{\bullet j}^r, 100 - M^*)\right) \wedge \\
&\left(\mathcal{Q}(A_{\bullet j}^r, A^*) < A_{ij}^r < \mathcal{Q}(A_{\bullet j}^r, 100 - A^*)\right) \wedge \\
&\left(k_{ij}, k_{ir} > 0 \right)
\}
\end{aligned}$$

Finally, the size factors are calculated as

$$\log_2 \hat{s}_j = \frac{\sum_{i \in I^*} w_{ij}^r M_{ij}^r}{\sum_{i \in I^*} w_{ij}^r}$$

where the weights are the inverse of estimated asymptotic variances

$$
w_{ij}^r
\approx \frac{1}{\mathrm{Var}(M_{ij}^r)}
\propto \left(\frac{N_j - K_{ij}}{N_j K_{ij}} + \frac{N_r - K_{ir}}{N_r K_{ir}} \right)^{-1}
$$

<span style="color: red">In the edgeR [source code](https://rdrr.io/bioc/edgeR/src/R/calcNormFactors.R), the size factor produced by the [median-of-ratios method](#median-of-ratios) is further divided by the library size, but the TMM size factor is not. Why?</span>

<details markdown="block"><summary>Derivation</summary>

If the mean of $$K_{ij}$$ is known, the variance of $$M_{ij}^r$$ can be estimated using the [delta method](https://en.wikipedia.org/wiki/Delta_method).

Let

$$M_{ij}^r = g(K_{ij}, K_{ir}) = \log_2 \frac{K_{ij} / N_j}{K_{ir} / N_r}$$

Linearizing $$g(K_{ij}, K_{ir})$$ around $$\mathbb{E}(K_{ij}) = \mu_{ij}$$ and $$\mathbb{E}(K_{ir}) = \mu_{ir}$$ gives

$$
M_{ij}^r
= g(K_{ij}, K_{ir})
\approx g(\mu_{ij}, \mu_{ir}) + (K_{ij} - \mu_{ij}) \frac{\partial}{\partial K_{ij}} g(\mu_{ij}, \mu_{ir}) + (K_{ir} - \mu_{ir}) \frac{\partial}{\partial K_{ir}} g(\mu_{ij}, \mu_{ir})
$$

with

$$\begin{gathered}
\frac{\partial}{\partial K_{ij}} g(\mu_{ij}, \mu_{ir}) = \frac{1}{\mu_{ij} \ln 2} \\
\frac{\partial}{\partial K_{ir}} g(\mu_{ij}, \mu_{ir}) = -\frac{1}{\mu_{ir} \ln 2}
\end{gathered}$$

so

<!-- $$\begin{aligned}
\mathbb{E}(M_{ij}^r)
&\approx \mathbb{E}\left(g(\mu_{ij}, \mu_{ir}) + (K_{ij} - \mu_{ij}) \frac{\partial}{\partial K_{ij}} g(\mu_{ij}, \mu_{ir}) + (K_{ir} - \mu_{ir}) \frac{\partial}{\partial K_{ir}} g(\mu_{ij}, \mu_{ir}) \right) \\
&= g(\mu_{ij}, \mu_{ir}) + \frac{\partial}{\partial K_{ij}} g(\mu_{ij}, \mu_{ir}) \underbrace{\mathbb{E}(K_{ij} - \mu_{ij})}_{0} + \frac{\partial}{\partial K_{ir}} g(\mu_{ij}, \mu_{ir}) \underbrace{\mathbb{E}(K_{ir} - \mu_{ir})}_{0} \\
&= g(\mu_{ij}, \mu_{ir})
\end{aligned}$$ -->

$$\begin{aligned}
\mathrm{Var}(M_{ij}^r)
&\approx \mathrm{Var}\left( g(\mu_{ij}, \mu_{ir}) + (K_{ij} - \mu_{ij}) \frac{\partial}{\partial K_{ij}} g(\mu_{ij}, \mu_{ir}) + (K_{ir} - \mu_{ir}) \frac{\partial}{\partial K_{ir}} g(\mu_{ij}, \mu_{ir}) \right) \\
&= 0 + \left[\frac{\partial}{\partial K_{ij}} g(\mu_{ij}, \mu_{ir}) \right]^2 \mathrm{Var}(K_{ij} - \mu_{ij}) + \left[\frac{\partial}{\partial K_{ir}} g(\mu_{ij}, \mu_{ir}) \right]^2 \mathrm{Var}(K_{ir} - \mu_{ir}) \\
&= \left[\frac{\partial}{\partial K_{ij}} g(\mu_{ij}, \mu_{ir}) \right]^2 \mathrm{Var}(K_{ij}) + \left[\frac{\partial}{\partial K_{ir}} g(\mu_{ij}, \mu_{ir}) \right]^2 \mathrm{Var}(K_{ir}) \\
&= \frac{1}{\mu_{ij}^2 (\ln 2)^2} \mathrm{Var}(K_{ij}) + \frac{1}{\mu_{ir}^2 (\ln 2)^2} \mathrm{Var}(K_{ir})
\end{aligned}$$

where the first equality assumes no covariance term between $$K_{ij}$$ and $$K_{ir}$$.

[Robinson & Oshlack](#references) assume $$K_{ij} \sim \text{Binom}(N_j, p_{ij})$$ (so $$\mu_{ij} = N_j p_{ij}$$), giving

$$\begin{aligned}
\mathrm{Var}(M_{ij}^r)
&\approx \frac{1}{N_j^2 p_{ij}^2 (\ln 2)^2} (N_j p_{ij} (1 - p_{ij})) + \frac{1}{N_r^2 p_{ir}^2 (\ln 2)^2} (N_r p_{ir} (1 - p_{ir})) \\
&= \frac{1}{(\ln 2)^2} \left(\frac{1 - p_{ij}}{N_j p_{ij}} + \frac{1 - p_{ir}}{N_r p_{ir}} \right) \\
&= \frac{1}{(\ln 2)^2} \left(\frac{N_j - N_j p_{ij}}{N_j N_j p_{ij}} + \frac{N_r - N_r p_{ir}}{N_r N_r p_{ir}} \right) \\
&\approx \frac{1}{(\ln 2)^2} \left(\frac{N_j - K_{ij}}{N_j K_{ij}} + \frac{N_r - K_{ir}}{N_r K_{ir}} \right)
\end{aligned}$$

The $$(\ln 2)^2$$ factors can be dropped from the weights because they would cancel between the weights in the numerator and the denominator in the size factors equation.

Note that other distributions (e.g., negative binomial, Poisson) can be assumed for $$K_{ij}$$ and would yield different values for $$\mathrm{Var}(K_{ij})$$ and consequently $$\mathrm{Var}(M_{ij}^r)$$.

<!-- If we assume $$K_{ij} \sim \text{NB}(\mu_{ij}, \alpha_i)$$, then

$$\begin{aligned}
\mathrm{Var}(M_{ij}^r)
&\approx \frac{1}{\mu_{ij}^2 (\ln 2)^2} (\mu_{ij} + \alpha_i \mu_{ij}^2) + \frac{1}{\mu_{ir}^2 (\ln 2)^2} (\mu_{ir} + \alpha_i \mu_{ir}^2) \\
&= \frac{1 + \alpha_i \mu_{ij}}{\mu_{ij} (\ln 2)^2} + \frac{1 + \alpha_i \mu_{ir}}{\mu_{ir} (\ln 2)^2}
\end{aligned}$$

If we assume $$K_{ij} \sim \text{Poi}(\mu_{ij})$$, then

$$\begin{aligned}
\mathrm{Var}(M_{ij}^r)
&\approx \frac{1}{\mu_{ij}^2 (\ln 2)^2} (\mu_{ij}) + \frac{1}{\mu_{ir}^2 (\ln 2)^2} (\mu_{ir}) \\
&= \frac{1}{(\ln 2)^2} \left(\frac{1}{\mu_{ij}} + \frac{1}{\mu_{ir}} \right) \\
&= \frac{1}{(\ln 2)^2} \frac{\mu_{ij} + \mu_{ir}}{\mu_{ij} \mu_{ir}}
\end{aligned}$$ -->

</details>

### Gene length-dependent metrics

Symbols
- $$X_t$$: number of reads or fragments mapping to transcript $$t$$
- $$N = \sum_{t \in T} X_t$$: total number of mapped reads
- $$\tilde{l}_t = l_t - m + 1$$: effective length of a transcript $$t$$, i.e., the number of positions in a transcript in which a read of length $$m$$ can start
- $$p_t = \frac{m_t}{\sum_{t \in T} m_t}$$: relative abundance of transcript $$t$$, where $$m_t$$ is the copy number of transcript $$t$$ in the sample
  - $$M = \sum_{t \in T} m_t$$ generally cannot be inferred from sequencing data <span style="color:red">but can be measured using qPCR?</span>

|                                | RPKM                                               | FPKM                                                | TPM                     | 
|--------------------------------|----------------------------------------------------|-----------------------------------------------------|-------------------------| 
| Acronym                        | reads per kilobase per million mapped reads        | fragments per kilobase per per million mapped reads | transcripts per million | 
| Formula                        | $$\frac{X_t}{(N / {10}^6)(\tilde{l}_t / {10}^3)}$$ | $$\frac{X_t}{(N / {10}^6)(\tilde{l}_t / {10}^3)}$$  | $${10}^6 \hat{p}_t = 10^6 \frac{X_t / \tilde{l}_t}{\sum_{r \in T} X_r / \tilde{l}_r}$$ | 
| Comparable across experiments* | No                                                 | No                                                  | Yes                     | 

*RPKM and FPKM values are not comparable across samples because they do not sum (over all unique transcripts in a sample) to a fixed value: $$\sum_{t \in T} \text{RPKM}_t$$ is not constant. By normalizing to total effective (with respect to gene length) number of mapped reads instead of total number of mapped reads $$N$$, TPM always sums to $$10^6$$ for a given sample. Note that an equivalent estimate of relative abundance is $$\hat{p}_t = \frac{\text{RPKM}_t}{\sum_{r \in T} \text{RPKM}_r}$$.
- Since RPKM and FPKM are not comparable across samples, there is not a defined value corresponding to 1 transcript per sample. However, as a very rough estimate, values in the range of 1-10 have been suggested. [[Pachter's blog post]](#references) By definition, a relative abundance value $$p_t = \frac{1}{M}$$ (or a TPM of $$\frac{ {10}^6 }{M}$$) corresponds to 1 transcript per sample.

However, even the relative abundance values $$p_t$$ (and consequently TPM values) are not necessarily comparable across experiments due to the potential bias introduced by high counts of a few genes, as explored in the example in the [naive normalization section](#naive-normalization). In short, consider samples 1 and 2 that are identical, except that one gene (say, gene C) is completely knocked-out in sample 2. Because the relative abundance values $$\hat{p}_t$$ sum to one (by construction), the relative abundance values in sample 2 will be higher than those in sample 1 for all genes except gene C, for which $$p_C = 0$$ in sample 2.

For a brief comparison of the three metrics, consider
1. [Harold Pimentel's blog post](https://haroldpimentel.wordpress.com/2014/05/08/what-the-fpkm-a-review-rna-seq-expression-units/)
2. [Lior Pachter's 2013 CSHL Keynote](https://youtu.be/5NiFibnbE8o?t=1832)

<details markdown="block"><summary>Derivation</summary>

Definitions
- A **transcript** $$t$$ is a unique sequence (strand) of mRNA corresponding to a gene isoform. It is characterized by a length $$l_t$$.
- The physical copies of transcripts that are sequenced are called **fragments**.
- A **read** $$f$$ is characterized by the its length, the fragment it came from (and consequently the transcript it maps to), and the position it maps to in the transcript.

Model (from [[Pachter's arXiv article]](#references))
- $$T$$: set of transcripts (mRNA isoform molecules; equivalently, the cDNA molecules reverse transcribed from such mRNA isoform molecules)
- $$F$$: set of reads from a sequencing run
  - $$F_t = \{f: f \in_\text{map} t\} \subseteq F$$: the set of reads mapping to transcript $$t \in T$$
    - We define the operator $$f \in_\text{map} t$$ to mean that read $$f$$ maps to transcript $$t$$.
    - $$X_t = \lvert F_t \rvert$$: number of reads mapping to transcript $$t$$
  - $$N = \lvert F \rvert = \sum_{t \in T} X_t$$: total number of mapped reads
  - $$m$$: fixed length of all reads
- $$\tilde{l}_t = l_t - m + 1$$: effective length of a transcript $$t \in T$$, i.e., the number of positions in a transcript in which a read of length $$m$$ can start
- $$p_t$$: relative abundance of transcript $$t$$, i.e., the proportion of all mRNA molecules corresponding to transcript $$t$$
  - By construction, $$\sum_{t \in T} p_t = 1$$.
- $$\alpha_t = P(f \in_\text{map} t) = \frac{p_t \tilde{l}_t}{\sum_{r \in T} p_r \tilde{l}_r}$$: probability of selecting a read from transcript $$t$$
  - By construction, $$\sum_{t \in T} \alpha_t = 1$$.
  - $$\alpha_t \neq p_t$$ because $$\alpha_t$$ accounts for transcript lengths.
  - Note that $$p_t$$ can be expressed in terms of $$\alpha$$

    $$\begin{aligned}
    p_t
    &= \frac{\frac{p_t}{\sum_{r \in T} p_r \tilde{l}_r}}{\frac{1}{\sum_{t' \in T} p_{t'} \tilde{l}_{t'}}}
      = \frac{\frac{p_t}{\sum_{r \in T} p_r \tilde{l}_r}}{\frac{\sum_{r \in T} p_r}{\sum_{t' \in T} p_{t'} \tilde{l}_{t'}}} \\
    &= \frac{\frac{p_t}{\sum_{r \in T} p_r \tilde{l}_r}}{\sum_{r \in T} \frac{p_r}{\sum_{t' \in T} p_{t'} \tilde{l}_{t'}}}
      = \frac{\frac{p_t \tilde{l}_t}{\sum_{r \in T} p_r \tilde{l}_r} / \tilde{l}_t}{\sum_{r \in T} \frac{p_r \tilde{l}_r}{\sum_{t' \in T} p_{t'} \tilde{l}_{t'}} / \tilde{l}_r} \\
    &= \frac{\alpha_t / \tilde{l}_t}{\sum_{r \in T} \alpha_r / \tilde{l}_r}
    \end{aligned}$$

We model the observation of a particular read $$f$$ that maps to some position $$\gamma$$ in transcript $$t$$ as a generative sequence of probabilistic events:
1. Choose the transcript $$t$$ from which to select a read $$f$$: $$P(f \in_\text{map} t) = \alpha_t$$.
2. Choose a position uniformly at random from among $$\tilde{l}_t = l_t - m + 1$$ possible positions to begin the read: $$P(f \mid f \in_\text{map} t) = \frac{1}{\tilde{l}_t}$$

The likelihood of observing the reads $$F$$ as a function of the parameters $$\alpha$$ (or equivalently, the parameters $$p$$) is

$$\begin{aligned}
L(\alpha)
&= \prod_{f \in F} P(f)
 = \prod_{t \in T} \prod_{f \in F_t} P(f)
 = \prod_{t \in T} \prod_{f \in F_t} P(f \in_\text{map} t) P(f \mid f \in_\text{map} t) \\
&= \prod_{t \in T} \prod_{f \in F_t} \frac{\alpha_t}{\tilde{l}_t}
 = \prod_{t \in T} \left(\frac{\alpha_t}{\tilde{l}_t} \right)^{X_t} \\
\rightarrow l(\alpha)
&= \log L(\alpha) = \sum_{t \in T} X_t \left(\log \alpha_t - \log \tilde{l}_t \right)
\end{aligned}$$

The maximum likelihood estimate for $$\alpha_t$$ can be found by building the Lagrangian and setting its derivative to zero.

$$\begin{aligned}
\mathcal{L}(\alpha) &= L(\alpha) + \beta \sum_{t \in T} \alpha_t \\
0 &= \frac{\partial\mathcal{L}}{\partial \alpha_t} = \frac{X_t}{\alpha_t} + \beta
  \rightarrow \alpha_t = -\frac{X_t}{\beta} \\
1 &= \sum_{t \in T} \alpha_t = \sum_{t \in T} -\frac{X_t}{\beta}
  \rightarrow \beta = - \sum_{t \in T} X_t = - N \\
\Rightarrow \hat{\alpha_t} &= \frac{X_t}{N}
\end{aligned}$$

Finally,

$$\begin{aligned}
\hat{p}_t
&= \frac{\hat{\alpha}_t / \tilde{l}_t}{\sum_{r \in T} \hat{\alpha}_r / \tilde{l}_r}
 = \frac{\frac{X_t}{N \tilde{l}_t}}{ {\sum_{r \in T} \frac{X_r}{N \tilde{l}_r}}}
 = \frac{X_t}{(N / {10}^6)(\tilde{l}_t / {10}^3)} \frac{ {10}^{-9}}{ {\sum_{r \in T} \frac{X_r}{N \tilde{l}_r}}} \\
&= \text{RPKM}_t \cdot \frac{N \cdot {10}^{-9}}{ {\sum_{r \in T} \frac{X_r}{\tilde{l}_r}}}
\end{aligned}$$

where (equivalent ways of expressing RPKM)

$$
\text{RPKM}_t
= \frac{X_t}{(N / {10}^6)(\tilde{l}_t / {10}^3)}
= \frac{\hat{\alpha}_t \cdot {10}^9}{\tilde{l}_t}
= \frac{p_t \cdot {10}^9}{\sum_{r \in T} p_r \tilde{l}_r}
$$

Intepretation
- RPKM is based on maximum likelihood estimates for $$\alpha$$, so it itself is an *estimate*.
- The denominator $$\sum_{r \in T} p_r \tilde{l}_r$$ is a weighted average of transcript lengths, where the weights are transcript abundances. While this is constant for a given experiment, it is *not* constant *across* experiments, since the transcript abundances $$p_r$$ are likely different. Hence, RPKM values are not comparable across experiments.
- FPKM is a generalization of RPKM where a single fragment might yield multiple reads, e.g., in paired-end sequencing.
  > With paired-end RNA-seq, two reads can correspond to a single fragment, or, if one read in the pair did not map, one read can correspond to a single fragment. [[RNA-Seq blog]](https://www.rna-seqblog.com/rpkm-fpkm-and-tpm-clearly-explained/)
  
  Formally, consider a set of raw reads $$F'$$. Reads from the same fragment are treated as a single processed read to generate the set of processed reads $$F$$. With this pre-processing step, the formula for FPKM is identical to that of RPKM.

</details>

## Model

### DESeq2

Generalized linear model (GLM) with logarithmic link

$$\begin{aligned}
K_{ij} &\sim \text{NB}(\mu_{ij}, \alpha_i) \\
\mu_{ij} &= s_{ij} q_{ij} \\
\log_2 q &= \beta X^\top & \left(\log_2 q_{ij} = \sum_{l} \beta_{il} X_{jl} \right)
\end{aligned}$$

Hierarchical construction of negative binomial
- $$R_{ij} \sim \text{Gamma}(a_{ij} = \frac{q_{ij}^2}{v_{ij}}, \theta_{ij} = \frac{v_{ij}}{q_{ij}})$$: normalized count of transcripts from sample $$j$$ belonging to gene $$i$$; the distribution is over biological replicates from experimental condition $$\rho(j)$$ 
  - []()$$\mathbb{E}(R_{ij}) = a_{ij} \theta_{ij} = q_{ij}$$ <!-- The empty links []() are necessary for the math to be rendered in-line, since there is no text on the line. -->
  - []()$$\text{Var}(R_{ij}) = a_{ij} \theta_{ij}^2 = v_{ij}$$
- []()$$K_{ij} \mid R_{ij} \sim \text{Poi}(s_j R_{ij}) \rightarrow K_{ij} \sim \text{NB}(\mu_{ij}, \alpha_i)$$
  - $$\mu_{ij} = \mathbb{E}(K_{ij}) = a_{ij} s_j \theta_{ij} = s_j q_{ij}$$ [[Eq. (2), DESeq paper]](#references)
  - $$\sigma^2_{ij} = \text{Var}(K_{ij}) = a_{ij} s_j \theta_{ij} + a_{ij} s_j^2 \theta_{ij}^2 = s_j q_{ij} + s_j^2 v_{ij}$$ [[Eq. (3), DESeq paper]](#references)
  - []()$$\alpha_i = \frac{1}{a_{ij}} = \frac{v_{ij}}{q_{ij}^2}$$

Parameters (in order of estimation) [[DESeq2 vignette]](#references)
- $$s_{ij}$$: gene- and sample-specific normalization factor
  - By default, DESeq2 uses only a sample-specific normalization factor by assuming $$s_{ij} = s_j$$ for all $$i = 1, ..., n$$. This will account for differences in total reads (sequencing depth). See the median-of-ratios method described under the [Normalization](#normalization) section.
  - To account for effects of GC content, gene length, and other gene-specific properties during the sample preparation (e.g., PCR amplification) and sequencing processes, consider using packages/methods like [cqn](https://www.bioconductor.org/packages/release/bioc/html/cqn.html) or [EDASeq](https://bioconductor.org/packages/release/bioc/html/EDASeq.html) to calculate gene-specific normalization factors.
- $$q \in \mathbb{R}^{n \times m}$$: $$q_{ij}$$ is a normalized count of transcripts from sample $$j$$ belonging to gene $$i$$.
  - For a given gene $$i$$, DESeq2 uses the same estimate $$\hat{q}_{i, \rho(j)}$$ for all samples from the same experimental condition. In other words, for all samples $$\tilde{j}$$ such that $$\rho(\tilde{j}) = \rho(j)$$, where $$\rho(j)$$ is the experimental condition of sample $$j$$, we have $$q_{i\tilde{j}} = q_{i, \rho(j)}$$.
  - $$\hat{q}_{i, \rho(j)} = \frac{1}{m_{\rho(j)}} \sum_{\tilde{j}: \rho(\tilde{j}) = \rho(j)} \frac{k_{ij}}{\hat{s}_j}$$. Estimate as the average of normalized counts from samples in the same experimental condition.
- $$\alpha_i$$: dispersion values for each gene
   - See next [section](#sharing-dispersion-information-across-genes) for estimation method.
   - Note that $$v_{ij}$$, $$a_{ij}$$, and $$\theta_{ij}$$ are not necessarily explicitly estimated but can be computed from $$\alpha_i$$, $$q_{ij}$$, and $$s_{ij}$$.
- $$\beta \in \mathbb{R}^{n \times c}$$: $$\beta_{il}$$ is the $$\log_2$$ fold change due to experimental condition $$l$$.
  - DESeq2 fits a zero-centered normal distribution to the observed distribution of MLE (maximum-likelihood estimate) $$\beta$$ values and uses that as a prior for final MAP (*maximum a posteriori*) estimation of $$\beta$$. The normal prior leads to "shrunken" logarithmic fold change (LFC) estimates, which are more accurate when the amount of (Fisher) information for a gene is low (e.g., when the gene has very low counts and/or high dispersion.)

#### Sharing dispersion information across genes

When sample sizes are small, dispersion estimates $$\alpha_i$$ are highly variable (noisy). DESeq2 addresses this problem by assuming that "genes of similar average expression strength have similar dispersion." The counts $$K_{ij}$$ are first fit to the negative binomial model (as a generalized linear model) using MLE. Next, DESeq2 fits a smooth curve regressing the MLE dispersion estimate against the mean of normalized counts. Finally, the MLE estimates are shrunk towards the smooth curve by treating the smooth curve as a prior and generating MAP estimates of the dispersion values. [[DESeq2 paper]](#references)

# Knowledge base-driven pathway analysis

Let $$G$$ denote the set of all genes considered, with $$\abs{G} = N$$. For example, if we consider all protein-coding human genes, $$N \approx 20438$$ (according to statistics from [Ensembl](http://www.ensembl.org/Homo_sapiens/Info/Annotation)).

Input
- Expression matrix
  - Rows: genes $$G$$
  - Columns: samples labeled by phenotype (e.g., control1, control2, cancer1, cancer2, ...)
- Pathways (gene sets)
  - In the sections below, I use the symbol $$S$$ to denote a gene set of interest.
  - Following [[Khatri et al.]](#references), the sections below focus "on methods that exploit pathway knowledge in public repositories such as GO ... rather than on methods [such as [WGCNA](https://horvath.genetics.ucla.edu/html/CoexpressionNetwork/Rpackages/WGCNA/)] that infer pathways from molecular measurements." Consequently, the analyses are only as good as the pathways annotated in existing databases; see [[Tomczak et al.]](#references) and [[Haynes et al.]](#references).

## Over-representation analysis (ORA)

Idea: "statistically evaluates the fraction of genes in a particular pathway found among the set of genes showing changes in expression" [[Khatri et al.]](#references)

Assumptions
1. Each gene is independent of other genes.
2. Each pathway is independent of other pathways.

Limitations
1. Ignores real values associated with each gene, since it only considers whether a gene is part of some gene set or not.
2. Only uses inforation from the most significantly differentially expressed genes.

Method
1. Choose set $$\delta$$ of differentially expressed genes (DEGs). Common threshold: fold-change > 2 and $$q$$-value < 0.05.
2. For each pathway gene set $$S$$ (of size $$\abs{S} = N_S$$), test for enrichment of DEGs. The null hypothesis is that the set of DEGs is not enriched for genes in the pathway, i.e., the proportion $$\frac{\abs{\delta \cap S}}{\abs{\delta}}$$ of pathway genes in the set of DEGs is the same as the proportion $$\frac{\abs{S}}{N}$$ of pathway genes in the set of all genes.
   - <span style="color: red">[Khatri et al. (Text S2.3)](#references) argues that ORA approaches test competitive null hypotheses because they compare "the proportion of differentially expressed genes in a pathway with the proportion of differentially expressed genes not in the pathway." However, the second "proportion" should be the proportion of pathway genes in the set of all genes.</span>
   - Test sample: $$x = \abs{\delta \cap S}$$. $$\abs{\delta}$$ genes are drawn from the set $$\delta$$ of all DEGs.
   - Null distribution: $$\abs{\delta}$$ genes are drawn from the set $$G$$ of all genes.
     - Binomial: assumes genes are drawn with replacement. $$X \sim \mathrm{Binom}(n = \abs{\delta}, p = \frac{N_S}{N})$$.

       $$
       p
       = P(X \geq x)
       = 1 - \sum_{k=0}^{x-1} P(X = k)
       = 1 - \sum_{k=0}^{x-1} {\abs{\delta} \choose k} \left(\frac{N_S}{N}\right)^k \left(1 - \frac{N_S}{N} \right)^{\abs{\delta} - k}
       $$

     - Hypergeometric (Fisher's exact test): assumes genes are drawn without replacement. $$X \sim \mathrm{Hyper}(N = N, n = \abs{\delta}, K = N_S)$$, where the notation follows [Wikipedia](https://en.wikipedia.org/wiki/Hypergeometric_distribution).

       $$
       p
       = P(X \geq x)
       = 1 - \sum_{k=0}^{x-1} P(X = k)
       = 1 - \sum_{k=0}^{x-1} \frac{ {N_S \choose k}{N - N_S \choose \abs{\delta} - k}}{ {N \choose \abs{\delta}}}
       $$

## Functional class scoring (FCS)

Idea: "weaker but coordinated changes in sets of functionally related genes (i.e., pathways) can also have significant effects" [[Khatri et al.]](#references)

Method
1. Compute gene-level statistics. Examples:
   - correlation with phenotype
   - t-statistic
   - fold-change
2. Aggregate gene-level statistics into a single pathway-level statistic. Examples:
   - Kolmogorov-Smirnov statistic
   - sum, mean, or median of gene-level statistics
   - Wilcoxon rank sum
3. Assess the statistical significance of the pathway-level statistic.
   - Competitive null hypothesis: "genes in a pathway are at most as often differentially expressed as the genes not in the pathway" [[Khatri et al., Text S2.3]](#references)
     - Null distribution: permute gene labels
   - Self-contained null hypothesis: "no genes in a given pathway are differentially expressed" [[Khatri et al., Text S2.3]](#references)
     - Null distribution: permute class (phenotype) labels for each sample. This null incorporates the correlation structure of genes. This is reasonable if we assume that most pathways are not affected by different phenotypic conditions.

Assumptions
1. Each pathway is independent of other pathways.
2. [Only for the self-contained null hypothesis] The correlation structure of genes is preserved across phenotypic conditions.

### GSEA

Acronym: Gene Set Enrichment Analysis

Algorithm
1. Compute gene-level statistic: fold-change of average expression between conditions, correlation between expression and phenotype, etc.
2. Rank genes from most-to-least differentially-expressed according to their gene-level statistic.
   - Let $$r_i$$ denote the gene-level statistic of the $$i$$th ranked gene $$g_i$$, where $$i \in [1, N]$$.
3. Use a "random walk" to calculate enrichment score (ES).
   - Use labels $$y_i \in \{0, 1\}$$ to denote whether ranked gene $$i$$ is in the gene set $$S$$ of interest.
   - Old method [[GSEA 2003]](#references): equal weights at every step; $$\text{ES}$$ is a standard [Kolmogorov-Smirnov](https://en.wikipedia.org/wiki/Kolmogorov%E2%80%93Smirnov_test) statistic.

     $$\begin{gathered}
     p_\text{hit} = \sqrt{\frac{N - N_S}{N_S}} \\
     p_\text{miss} = -\sqrt{\frac{N_S}{N - N_S}} \\
     \text{ES} = \max_{n \in [1, N]} \left\lvert \sum_{i=1}^n y_i p_\text{hit} + (1 - y_i) p_\text{miss} \right\rvert
     \end{gathered}$$

   - New method [[GSEA 2005]](#references): steps are weighted according to each gene's gene-level statistic; $$\text{ES}$$ is a weighted [Kolmogorov-Smirnov](https://en.wikipedia.org/wiki/Kolmogorov%E2%80%93Smirnov_test) statistic.
     - Formulation to match the notation of the old method:

       $$\begin{aligned}
       p_\text{hit}(i) &= \begin{cases}
         \frac{\abs{r_i}^p}{N_R} & i \in S \\
         0 & \text{otherwise}
       \end{cases}, & N_R = \sum_{j=1}^N y_j \abs{r_j}^p \\
       p_\text{miss} &= -\frac{1}{N - N_S} \\
       \text{ES} &= \max_{n \in [1, N]} \left\lvert \sum_{i=1}^n y_i p_\text{hit}(i) + (1 - y_i) p_\text{miss} \right\rvert
       \end{aligned}$$

     - Formulation from the 2005 paper:

       $$\begin{aligned}
       P_\text{hit}(S, i) &= \sum_{g_j \in S,\, j \leq i} \frac{\abs{r_j}^p}{N_R}, & N_R = \sum_{g_j \in S}^N \abs{r_j}^p \\
       P_\text{miss}(S, i) &= \sum_{g_j \notin S,\, j \leq i} \frac{1}{N - N_S} \\
       \text{ES}(S) &= \max_{i \in [1, N]} \left\lvert P_\text{hit}(S, i) - P_\text{miss}(S, i) \right\rvert
       \end{aligned}$$

       - $$P_\text{hit}(S, i)$$ and $$P_\text{miss}(S, i)$$ are the fraction (with respect to genes $$i = 1, ..., i$$) of genes in / not in $$S$$ weighted by their gene-level statistics.
     - $$p$$ is a hyperparameter controlling the weight of each step in the random walk.
       - The authors recommend a default of $$p = 1$$. [[GSEA 2005 supplemental]](#references)
       - When $$p=0$$, the new method becomes nearly identical to the old method with equal weights at every step: $$p_\text{hit}(i)$$ becomes $$\frac{1}{N_S}$$.
   - Note that the "random walk" always returns to 0:
     - Old method:

       $$\begin{aligned}
         \sum_{i=1}^{N} y_i p_\text{hit} + (1 - y_i) p_\text{miss}
         &= N_S p_\text{hit} + (N - N_S) p_\text{miss} \\
         &= \sqrt{N_S (N - N_S)} - \sqrt{(N - N_S) N_S} \\
         &= 0
       \end{aligned}$$

     - New method:

       $$\begin{aligned}
         \sum_{i=1}^{N} y_i p_\text{hit}(i) + (1 - y_i) p_\text{miss}
         &= \left( \sum_{i=1}^N y_i \frac{\abs{r_i}^p}{N_R} \right) + (N - N_S) \left(- \frac{1}{N - N_S}\right) \\
         &= \left( \frac{1}{N_R} \sum_{i=1}^N y_i \abs{r_i}^p \right) - 1 \\
         &= \frac{1}{N_R} N_R - 1 \\
         &= 0
       \end{aligned}$$

4. Estimate significance with permutation test.
   1. Permute column labels of expression matrix.
   2. Perform steps 1-3 of the GSEA algorithm with the permuted expression matrix (including re-computing gene-level statistics and re-ranking genes) and compute an enrichment score.
      - [New method only] Discard this enrichment score sampled from the null distribution if its sign does not match the sign of the observed $$\text{ES}(S)$$. Separately considering positively and negatively scoring gene sets is necessary because
        > the use of weighted steps could cause the distribution of observed enrichment scores to be asymmetric in cases where many more genes are correlated with one of the two phenotypes. <span style="color: red">Why?</span>
   3. Estimate nominal $$p$$-value for $$\text{ES}(S)$$ from the null distribution.
   4. [New method only] Compute a normalized enrichment score $$\text{NES}(S)$$ that adjusts for variation in gene set size.
5. Correct for multiple hypothesis testing (each gene set is a hypothesis).

## Pathway-topology (PT)-based approaches

Idea: Take advantage of knowledge (e.g., KEGG, MetaCyc, Reactome, PantherDB) of how (subcellular localization; activation/inhibition; etc.) genes in a pathway interact, not just the knowledge that they are in the same pathway.

Method: Generally follows same three steps as [functional class scoring](#functional-class-scoring-fcs) but incorporates additional pathway information in computing gene-level and pathway-level statistics.

Limitations
- > [T]rue pathway topology is dependent on the type of cell due to cell-specific gene expression profiles and [the] condition being studied. [[Khatri et al.]](#references)

### SPIA

Acronym: Signaling Pathway Impact Analysis

Idea
> The impact analysis combines two types of evidence: (i) the over-representation of DE genes in a given pathway and (ii) the abnormal perturbation of that pathway, as measured by propagating measured expression changes across the pathway topology. These two aspects are captured by two independent probability values, $$P_{NDE}$$ and $$P_{PERT}$$. [[SPIA paper]](#references)

**DEG enrichment analysis: $$P_{NDE}$$**

> Any of the existing ORA or FCS approaches can be used to calculate $$P_{NDE}$$, as long as this probability remains independent of the magnitudes of the fold-changes.

**Pathway perturbation analysis: $$P_{PERT}$$**

Define the following symbols:
- $$g_1, ..., g_{N_S}$$: genes in the pathway $$S$$
- $$\Delta E(g_i)$$: $$\log_2$$ fold-change in expression of gene $$g_i$$
- $$N_{ds}(g_i)$$: number of genes downstream of $$g_i$$ in the pathway
- $$\beta_{i,j} = \begin{cases}
   -1 & \text{gene } j \text{ inhibits gene } i \\
    1 & \text{gene } j \text{ activates gene } i \\
    0 & \text{otherwise}
  \end{cases}$$
- $$PF(g_i)$$: perturbation factor of gene $$i$$ (this is the gene-level statistic)

Linear system

$$PF(g_i) = \Delta E(g_i) + \sum_{j=1}^{N_S} \beta_{ij} \frac{PF(g_j)}{N_{ds}(g_j)}$$


$$
\underbrace{\begin{bmatrix}
  PF(g_1) \\ PF(g_2) \\ \vdots \\ PF(g_N)
\end{bmatrix}}_{PF}
=
\underbrace{\begin{bmatrix}
  \Delta E(g_1) \\ \Delta E(g_2) \\ \vdots \\ \Delta E(g_{N_S})
\end{bmatrix}}_{\Delta E}
+
\underbrace{\begin{bmatrix}
  \frac{\beta_{1, 1}}{N_{ds}(g_1)} & \frac{\beta_{1, 2}}{N_{ds}(g_2)} & \cdots & \frac{\beta_{1, N_S}}{N_{ds}(g_{N_S})} \\
  \frac{\beta_{2, 1}}{N_{ds}(g_1)} & \frac{\beta_{2, 2}}{N_{ds}(g_2)} & \cdots & \frac{\beta_{2, N_S}}{N_{ds}(g_{N_S})} \\
  \vdots & & \ddots & \vdots \\
  \frac{\beta_{N_S, 1}}{N_{ds}(g_1)} & \frac{\beta_{N_S, 2}}{N_{ds}(g_2)} & \cdots & \frac{\beta_{N_S, N_S}}{N_{ds}(g_{N_S})}
\end{bmatrix}}_B
\times
\underbrace{\begin{bmatrix}
  PF(g_1) \\ PF(g_2) \\ \vdots \\ PF(g_{N_S})
\end{bmatrix}}_{PF}
$$

Observe that the perturbation factor of gene $$j$$ is distributed among all its downstream interacting partners. (This is reminiscent of the PageRank system of equations.)

The net accumulations of the perturbations from other genes on gene $$i$$ is $$\text{Acc}(g_i) = PF(g_i) - \Delta E(g_i)$$. The total accumulated perturbation in the pathway $$S$$ is then computed as the pathway-level statistic

$$t_A = \sum_{i=1}^{N_S} \mathrm{Acc}(g_i)$$

Finally, $$P_{PERT} = P(T_A \geq t_A \mid H_0)$$ is calculated as the $$p$$-value of the pathway-level statistic $$t_A$$ following a bootstrap procedure described in the supplemental text of the [[SPIA paper]](#references).

**Combining DEG and pathway analyses: $$P_G$$**

Let the observed / computed values of $$P_{NDE}$$ and $$P_{PERT}$$ be $$p_{nde}, p_{pert}$$, respectively with $$p_{nde} \cdot p_{pert} = c$$.

Since $$P_{NDE}$$ and $$P_{PERT}$$ are $$p$$-values, under the null hypothesis they are uniformly distributed on the interval $$[0, 1]$$. The support of their joint distribution is therefore the unit square. Since $$P_{NDE}$$ and $$P_{PERT}$$ are independent, points with the same probability lie as $$(p_{nde}, p_{pert})$$ lie along the hyperbola defined by $$P_{NDE} P_{PERT} = c$$. Then, the probability of obtaining a set of $$p$$-values as extreme or more extreme than $$(p_{nde}, p_{pert})$$ is the area under and to the left of the hyperbola (see Figure S1 in the [[SPIA paper]](#references)):

$$
P_G
= \int_0^c dP_{NDE} + \int_c^1 \frac{c}{P_{NDE}} dP_{NDE}
= c + c \cdot \ln P_{NDE} \rvert_c^1
= c - c \cdot \ln c
$$

# References

1. Robinson, M. D. & Oshlack, A. A scaling normalization method for differential expression analysis of RNA-seq data. *Genome Biology* 11, R25 (2010). [doi:10.1186/gb-2010-11-3-r25](https://doi.org/10.1186/gb-2010-11-3-r25).
   - > Depending on the experimental situation, Poisson seems appropriate for technical replicates and Negative Binomial may be appropriate for the additional variation observed from biological replicates.
   - Presents the ["trimmed mean of M values" (TMM) library count normalization method](#trimmed-mean-of-m-values-tmm).
2. Risso, D. et al. Normalization of RNA-seq data using factor analysis of control genes or samples. *Nature Biotechnology* 32, 896–902 (2014). [doi:10.1038/nbt.2931](https://doi.org/10.1038/nbt.2931)
   - Explores the reliability of spike-in controls and presents a normalization strategy remove unwanted variation (RUV) (not explained here).
3. Lipp, J. Why sequencing data is modeled as negative binomial. *Bioramble* (2016). [https://bioramble.wordpress.com/2016/01/30/why-sequencing-data-is-modeled-as-negative-binomial/](https://bioramble.wordpress.com/2016/01/30/why-sequencing-data-is-modeled-as-negative-binomial/).
4. Anders, S. & Huber, W. Differential expression analysis for sequence count data. *Genome Biology* 11, R106 (2010). [doi:10.1186/gb-2010-11-10-r106](https://doi.org/10.1186/gb-2010-11-10-r106).
   - DESeq paper.
5. Love, M. I., Huber, W. & Anders, S. Moderated estimation of fold change and dispersion for RNA-seq data with DESeq2. *Genome Biol* 15, 550 (2014). [doi:10.1186/s13059-014-0550-8](https://doi.org/10.1186/s13059-014-0550-8).
   - DESeq2 paper.
6. Love, M. I., Anders, S. & Huber, W. Analyzing RNA-seq data with DESeq2. [https://bioconductor.org/packages/release/bioc/vignettes/DESeq2/inst/doc/DESeq2.html](https://bioconductor.org/packages/release/bioc/vignettes/DESeq2/inst/doc/DESeq2.html) (2020).
   - DESeq2 vignette.
7. Pachter, L. Models for transcript quantification from RNA-Seq. *arXiv*:1104.3889 [q-bio, stat] (2011). [http://arxiv.org/abs/1104.3889](http://arxiv.org/abs/1104.3889).
   - Derives general likelihood model for RNA-seq reads, likelihood estimation of $$p$$ and $$\alpha$$ values, and inference using the EM-algorithm.
8. Mortazavi, A., Williams, B. A., McCue, K., Schaeffer, L. & Wold, B. Mapping and quantifying mammalian transcriptomes by RNA-Seq. *Nature Methods* 5, 621–628 (2008). [doi:10.1038/nmeth.1226](https://doi.org/10.1038/nmeth.1226).
   - One of the original RNA-seq papers; introduces RPKM metric. See [Wikipedia](https://en.wikipedia.org/wiki/RNA-Seq#History), [Lior Pachter's \*Seq chronology](https://liorpachter.wordpress.com/seq/), and [this blog post](http://nextgenseek.com/2014/03/the-first-published-paper-on-rna-seq-setting-the-record-straight/).
9.  Trapnell, C. et al. Transcript assembly and quantification by RNA-Seq reveals unannotated transcripts and isoform switching during cell differentiation. *Nature Biotechnology* 28, 511–515 (2010). [doi:10.1038/nbt.1621](https://doi.org/10.1038/nbt.1621).
   - Cufflinks paper; introduces FPKM metric.
11. Li, B. & Dewey, C. N. RSEM: accurate transcript quantification from RNA-Seq data with or without a reference genome. *BMC Bioinformatics* 12, 323 (2011). [doi:10.1186/1471-2105-12-323](https://doi.org/10.1186/1471-2105-12-323).
    - RSEM paper; introduces TPM metric.
12. Pachter, L. Estimating number of transcripts from RNA-Seq measurements (and why I believe in paywall). *Bits of DNA* (2014). [https://liorpachter.wordpress.com/2014/04/30/estimating-number-of-transcripts-from-rna-seq-measurements-and-why-i-believe-in-paywall/](https://liorpachter.wordpress.com/2014/04/30/estimating-number-of-transcripts-from-rna-seq-measurements-and-why-i-believe-in-paywall/).
    - Explains why the FPKM value of a single gene cannot be converted to "transcripts per cell" without knowing the FPKM values of all genes.
13. Holmes, S. & Huber, W. *Modern Statistics for Modern Biology*. (Cambridge University Press, 2018). [https://web.stanford.edu/class/bios221/book/index.html](https://web.stanford.edu/class/bios221/book/index.html).
    - [Chapter 4](https://web.stanford.edu/class/bios221/book/Chap-Mixtures.html) derives the negative binomial model as a hierarchical Gamma-Poisson model.
    - [Chapter 8](https://web.stanford.edu/class/bios221/book/Chap-CountData.html) describes the DESeq2 model.
14. Khatri, P., Sirota, M. & Butte, A. J. Ten Years of Pathway Analysis: Current Approaches and Outstanding Challenges. *PLoS Computational Biology* 8, e1002375 (2012). [doi:10.1371/journal.pcbi.1002375](https://doi.org/10.1371/journal.pcbi.1002375).
    - Classifies existing pathway analysis methods (as of 2012) into one of three approaches: ORA, FCS, or PT; analyzes the assumptions and limitations of each approach.
15. Mootha, V. K. et al. PGC-1α-responsive genes involved in oxidative phosphorylation are coordinately downregulated in human diabetes. *Nature Genetics* 34, 267–273 (2003). [doi:10.1038/ng1180](https://doi.org/10.1038/ng1180).
    - Preliminary GSEA paper.
16. Subramanian, A. et al. Gene set enrichment analysis: A knowledge-based approach for interpreting genome-wide expression profiles. *Proc. Natl. Acad. Sci.* 102, 15545–15550 (2005). [doi:10.1073/pnas.0506580102](https://doi.org/10.1073/pnas.0506580102).
    - Robust GSEA paper. Describes software package `GSEA-P` and the creation of the Molecular Signature Database (MSigDB).
17. Tomczak, A. et al. Interpretation of biological experiments changes with evolution of the Gene Ontology and its annotations. *Scientific Reports* 8, 5115 (2018). [doi:10.1038/s41598-018-23395-2](https://doi.org/10.1038/s41598-018-23395-2).
    - > Our analysis suggests that GO evolution may have affected the interpretation and possibly reproducibility of experiments over time.
18. Haynes, W. A., Tomczak, A. & Khatri, P. Gene annotation bias impedes biomedical research. *Scientific Reports* 8, 1362 (2018). [doi:10.1038/s41598-018-19333-x](https://doi.org/10.1038/s41598-018-19333-x).
    - > Collectively, our results provide an evidence of a strong research bias in literature that focuses on well-annotated genes instead of those with the most significant disease relationship in terms of both expression and genetic variation.
19. Tarca, A. L. et al. A novel signaling pathway impact analysis. *Bioinformatics* 25, 75–82 (2009). [doi:10.1093/bioinformatics/btn577](https://doi.org/10.1093/bioinformatics/btn577).
    - SPIA paper.