---
title: Statistical Models of Gene Expression Analysis
layout: post
use_toc: true
use_math: true
excerpt: Why is the negative binomial distribution used to model sequencing read counts? What do FPKM and TPM mean?
---

# Probability distribution of counts

## Probability background

### Negative binomial distribution

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
3. For a reference that matches the formulation used here, see https://www.johndcook.com/negative_binomial.pdf.

#### Properties

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

#### Formulation as Gamma-Poisson

Idea: $$K$$ is a Poisson distribution where the mean of the Poisson distribution is proportional to a Gamma-distributed random variable.

$$K \sim \text{NB}(r, p)$$ is equivalent to $$K \mid \Lambda = \lambda \sim \text{Poi}(s\lambda)$$ where $$\Lambda \sim \text{Gamma}(a = r, \theta = \frac{1-p}{sp})$$
- $$p = \frac{1}{s\theta + 1}$$
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

| Model             | $$\text{Binomial}(n, p_i)$$   | $$\text{Poi}(\lambda_i = np_i)$$ | $$\text{NB}(r_i, \phi_i) = \text{Poi}(n P_i), P_i \sim \text{Gamma}(a = r_i, \theta = \frac{1-\phi_i}{n\phi_i})$$ |
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
  = \frac{\text{\# of reads not from gene $i$}}{\text{proportion of reads not from gene $i$}} p_i
  = (\text{\# total reads}) p_i
  = np_i
  $$

- Empirical evidence: Empirically, the variability of read counts is larger than the Binomial and Poisson distributions allows and is better approximated by a Negative Binomial distribution. [[DESeq paper]](#references) [[Bioramble blog]](#references)

## Gene length-dependent models

### Summary

Symbols
- $$X_t$$: number of reads or fragments mapping to transcript $$t$$
- $$N = \sum_{t \in T} X_t$$: total number of mapped reads
- $$\tilde{l}_t = l_t - m + 1$$: effective length of a transcript $$t$$, i.e., the number of positions in a transcript in which a read of length $$m$$ can start
- $$p_t = \frac{m_t}{\sum_{t \in T} m_t}$$: relative abundance of transcript $$t$$, where $$m_t$$ is the copy number of transcript $$t$$ in the sample
  - $$M = \sum_{t \in T} m_t$$ generally cannot be inferred from sequencing data but can be measured using qPCR.

|                                                                | RPKM                                               | FPKM                                                    | TPM                        | 
|----------------------------------------------------------------|----------------------------------------------------|---------------------------------------------------------|----------------------------| 
| Acronym                                                        | reads per kilobase per millions of reads mapped    | fragments per kilobase per per millions of reads mapped | transcripts per million    | 
| Formula                                                        | $$\frac{X_t}{(N / {10}^6)(\tilde{l}_t / {10}^3)}$$ | $$\frac{X_t}{(N / {10}^6)(\tilde{l}_t / {10}^3)}$$      | $$\hat{p}_t \cdot {10}^6$$ | 
| Comparable across experiments                                  | Bad                                                | Bad                                                     |      Not great                  | 
| Value for 1 transcript per sample (i.e., $$p_t = \frac{1}{M}$$) | ~10 [[Pachter's blog post]](#references)            | ~10                                                     | $$\frac{ {10}^6}{M}$$        | 

[]()

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
  - $$\sum_{t \in T} p_t = 1$$
- $$\alpha_t = P(f \in_\text{map} t)$$: probability of selecting a read from transcript $$t$$

We model the observation of a particular read $$f$$ that maps to some position $$\gamma$$ in transcript $$t$$ as a generative sequence of probabilistic events:
1. Choose the transcript $$t$$ from which to select a read $$f$$

   $$P(f \in_\text{map} t) = \alpha_t = \frac{p_t \tilde{l}_t}{\sum_{r \in T} p_r \tilde{l}_r}$$

   Observe that
   - $$\sum_{t \in T} \alpha_t = 1$$
   - $$\alpha_t \neq p_t$$ because $$\alpha_t$$ accounts for transcript lengths.
   - $$p_t$$ can be expressed in terms of $$\alpha$$ (see [[Pachter's arXiv article]](#references)): $$p_t = \frac{\alpha_t / \tilde{l}_t}{\sum_{r \in T} \alpha_r / \tilde{l}_r}$$
2. Choose a position uniformly at random from among $$\tilde{l}_t = l_t - m + 1$$ possible positions to begin the read

   $$P(f \mid f \in_\text{map} t) = \frac{1}{\tilde{l}_t}$$

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
 = \frac{X_t}{N \tilde{l}_t} \frac{1}{ {\sum_{r \in T} \frac{X_r}{N \tilde{l}_r}}}
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
- The denomiator $$\sum_{r \in T} p_r \tilde{l}_r$$ is a weighted average of transcript lengths, where the weights are transcript abundances. While this is constant for a given experiment, it is *not* necessarily constant *across* experiments, since the transcript abundances $$p_r$$ are likely different. Hence, RPKM values are not truly comparable across experiments.
  - Even the relative abundance values $$p_t$$ (and consequently TPM values) are not directly comparable across experiments, because the denominator changes across experiments

    $$
    \hat{p}_t
    = \frac{\hat{\alpha}_t / \tilde{l}_t}{\sum_{r \in T} \hat{\alpha}_r / \tilde{l}_r}
    = \frac{X_t / \tilde{l}_t}{\sum_{r \in T} X_r / \tilde{l}_r}
    = \frac{\text{RPKM}_t}{\sum_{r \in T} \text{RPKM}_r}
    $$

    To build intuition for the lack of comparability, consider two samples that are identical, except that one gene (say, gene $$a$$) is completely knocked-out in sample 2. Because the relative abundance values $$\hat{p}_t$$ must sum to one (by construction), the relative abundance values in sample two will be higher than those in sample 1 for all genes except gene $$a$$, for which $$p_a = 0$$ in sample 2. See [below](#naive-normalization) for a similar example.
  - The only rigorous solution is to spike in known quantities of artificial fragments into every sample and normalize based on counts of those transcripts.

#### FPKM

FPKM is a generalization of RPKM where a single fragment might yield multiple reads, e.g., in paired-end sequencing.
> With paired-end RNA-seq, two reads can correspond to a single fragment, or, if one read in the pair did not map, one read can correspond to a single fragment. [[RNA-Seq blog]](https://www.rna-seqblog.com/rpkm-fpkm-and-tpm-clearly-explained/)

Formally, consider a set of raw reads $$F'$$. Reads from the same fragment are treated as a single processed read to generate the set of processed reads $$F$$. With this pre-processing step, the formula for FPKM is identical to that of RPKM.

</details>

<a name="references"></a><details markdown="block"><summary>References</summary>

1. Robinson, M. D. & Oshlack, A. A scaling normalization method for differential expression analysis of RNA-seq data. *Genome Biol* 11, R25 (2010). https://doi.org/10.1186/gb-2010-11-3-r25.
2. Lipp, J. Why sequencing data is modeled as negative binomial. *Bioramble* (2016). https://bioramble.wordpress.com/2016/01/30/why-sequencing-data-is-modeled-as-negative-binomial/.
3. Anders, S. & Huber, W. Differential expression analysis for sequence count data. *Genome Biol* 11, R106 (2010). https://doi.org/10.1186/gb-2010-11-10-r106.
   - DESeq paper.
4. Pachter, L. Models for transcript quantification from RNA-Seq. *arXiv*:1104.3889 [q-bio, stat] (2011). http://arxiv.org/abs/1104.3889.
5. Mortazavi, A., Williams, B. A., McCue, K., Schaeffer, L. & Wold, B. Mapping and quantifying mammalian transcriptomes by RNA-Seq. *Nature Methods* 5, 621–628 (2008). https://doi.org/10.1038/nmeth.1226.
   - One of the original\* RNA-seq papers; introduces RPKM metric. \*See [Wikipedia](https://en.wikipedia.org/wiki/RNA-Seq#History), [Lior Pachter's \*Seq chronology](https://liorpachter.wordpress.com/seq/), and [this blog post](http://nextgenseek.com/2014/03/the-first-published-paper-on-rna-seq-setting-the-record-straight/).
7. Trapnell, C. et al. Transcript assembly and quantification by RNA-Seq reveals unannotated transcripts and isoform switching during cell differentiation. *Nature Biotechnology* 28, 511–515 (2010). https://doi.org/10.1038/nbt.1621.
   - Cufflinks paper; introduces FPKM metric.
8. Li, B. & Dewey, C. N. RSEM: accurate transcript quantification from RNA-Seq data with or without a reference genome. *BMC Bioinformatics* 12, 323 (2011). https://doi.org/10.1186/1471-2105-12-323.
   - RSEM paper; introduces TPM metric.
9. Pachter, L. Estimating number of transcripts from RNA-Seq measurements (and why I believe in paywall). *Bits of DNA* (2014). https://liorpachter.wordpress.com/2014/04/30/estimating-number-of-transcripts-from-rna-seq-measurements-and-why-i-believe-in-paywall/.

</details>

# Differential expression analysis

## DESeq / DESeq2

### Dataset

Dataset numbers
- $$n$$: number of genes
- $$m$$: number of samples
- $$c$$: number of conditions (including intercept)

Count matrix: $$K \in \mathbb{N}^{n \times m}$$
- Rows: genes
- Columns: samples
- $$K_{ij}$$: number of sequencing reads mapped to gene $$i$$ in sample $$j$$

#### Model (aka design) matrix: $$X \in \{0,1\}^{m \times c}$$
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

### Normalization

We want to estimate a sample-specific factor $$s_j$$ allowing us to compute normalized counts $$q_{ij} = \frac{\mathbb{E}(K_{ij})}{s_j}$$. Normalized counts can then be directly compared across samples.

#### Naive normalization

Divide by the total number of sequencing reads of a sample.

$$\hat{s}_j = \sum_{i=1}^n k_{ij}$$

Problem: This potentially reduces the power of the method to pick out differentially expressed genes, and in extreme cases, might lead to false positives. Consider the following dataset where gene C was knocked-out in sample 3. Using the normalized counts $$q_{ij}$$, one might erroneously conclude that genes A, B, D, and E are upregulated in sample 3 relative to samples 1 and 2.

| gene          | sample 1 | sample 2 | sample 3 | $$q_{i1}$$ | $$q_{i2}$$ | $$q_{i3}$$ | 
|---------------|----------|----------|----------|------------|------------|------------| 
| A             | 100      | 90       | 125      | 1/15       | 1/15       | 1.25/15    | 
| B             | 200      | 180      | 250      | 2/15       | 2/15       | 2.5/15     | 
| C             | 300      | 270      | 0        | 3/15       | 3/15       | 0          | 
| D             | 400      | 360      | 500      | 4/15       | 4/15       | 5/15       | 
| E             | 500      | 450      | 625      | 5/15       | 5/15       | 6.25/15    | 
| Total         | 1500     | 1350     | 1500     |            |            |            |
| $$\hat{s}_j$$ | 1500     | 1350     | 1500     |            |            |            |

[]()

#### Median-of-ratios

Consider a pseudo-reference sample $$K^R$$ whose counts for each gene are obtained by taking the geometric mean across samples. The size factor $$\hat{s}_j$$ for sample $$j$$ is computed as the median of the ratios of the $$j$$-th sample's counts to those of the pseudo-reference:

$$\hat{s}_j = \text{median}_{i: K_i^R > 0} \frac{K_{ij}}{K_i^R}, \quad K_i^R = \left(\prod_{j=1}^m K_{ij} \right)^\frac{1}{m}$$

Using the same example, the normalized counts of genes A, B, D, and E are the same across samples 1, 2, and 3, as desired.

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

### Model

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
  - $$\mu_{ij} = \mathbb{E}(K_{ij}) = a_{ij} s_j \theta_{ij} = s_j q_{ij}$$ [[Eq. (2), DESeq paper]](#references-1)
  - $$\sigma^2_{ij} = \text{Var}(K_{ij}) = a_{ij} s_j \theta_{ij} + a_{ij} s_j^2 \theta_{ij}^2 = s_j q_{ij} + s_j^2 v_{ij}$$ [[Eq. (3), DESeq paper]](#references-1)
  - []()$$\alpha_i = \frac{1}{a_{ij}} = \frac{v_{ij}}{q_{ij}^2}$$

Parameters (in order of estimation) [[DESeq2 vignette]](#references-1)
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

When sample sizes are small, dispersion estimates $$\alpha_i$$ are highly variable (noisy). DESeq2 addresses this problem by assuming that "genes of similar average expression strength have similar dispersion." The counts $$K_{ij}$$ are first fit to the negative binomial model (as a generalized linear model) using MLE. Next, DESeq2 fits a smooth curve regressing the MLE dispersion estimate against the mean of normalized counts. Finally, the MLE estimates are shrunk towards the smooth curve by treating the smooth curve as a prior and generating MAP estimates of the dispersion values. [[DESeq2 paper]](#references-1)

<a name="references-1"></a><details markdown="block"><summary>References</summary>

1. Anders, S. & Huber, W. Differential expression analysis for sequence count data. *Genome Biol* 11, R106 (2010). https://doi.org/10.1186/gb-2010-11-10-r106.
   - DESeq paper.
2. Love, M. I., Huber, W. & Anders, S. Moderated estimation of fold change and dispersion for RNA-seq data with DESeq2. *Genome Biol* 15, 550 (2014). https://doi.org/10.1186/s13059-014-0550-8.
   - DESeq2 paper.
3. Holmes, S. & Huber, W. *Modern Statistics for Modern Biology*. (Cambridge University Press, 2018). https://web.stanford.edu/class/bios221/book/index.html.
   - [Chapter 4](https://web.stanford.edu/class/bios221/book/Chap-Mixtures.html) derives the negative binomial model as a hierarchical Gamma-Poisson model.
   - [Chapter 8](https://web.stanford.edu/class/bios221/book/Chap-CountData.html) describes the DESeq2 model.
4. Love, M. I., Anders, S. & Huber, W. Analyzing RNA-seq data with DESeq2. https://bioconductor.org/packages/release/bioc/vignettes/DESeq2/inst/doc/DESeq2.html (2020).
   - DESeq2 vignette.

</details>

# Pathway analysis

## GSEA

Data
- Expression matrix
  - Rows: genes tested
  - Columns: samples (control1, control2, ..., cancer1, cancer2, ...)
- Gene sets

Algorithm
1. Rank genes according to differential expression
   - Rank according to fold-change between average (or median) expression between conditions
2. Use random walk to calculate enrichment score
   - Use index $$i \in \{1, ..., n_\text{total}\}$$ to denote ranked genes, where $$i = 1$$ corresponds to gene with largest positive fold-change
   - Use labels $$y_i \in \{0, 1\}$$ to denote whether ranked gene $$i$$ is in the gene set of interest

   $$p_\text{hit} = \sqrt{\frac{n_\text{total} - n_\text{gene set}}{n_\text{gene set}}}$$

   $$p_\text{miss} = -\sqrt{\frac{n_\text{gene set}}{n_\text{total} - n_\text{gene set}}}$$

   $$\text{ES} = \max_n \sum_{i=1}^n y_i p_\text{hit} + (1 - y_i) p_\text{miss}$$

   - For simplicity, the equation for $$\text{ES}$$ above only looks for positive enrichment, but can also look at the minimum value achieved during the random walk.
3. Estimate significance with permutation test
   - Permute column labels of expression matrix, recalculate enrichment score
4. Correct for multiple hypothesis testing (each gene set is a hypothesis)

Note that

$$\begin{aligned}
  \sum_{i=1}^{n_\text{total}} y_i p_\text{hit} + (1 - y_i) p_\text{miss}
  &= n_\text{gene set} p_\text{hit} + (n_\text{total} - n_\text{gene set}) p_\text{miss} \\
  &= \sqrt{n_\text{gene set} (n_\text{total} - n_\text{gene set})} - \sqrt{(n_\text{total} - n_\text{gene set}) n_\text{gene set}} \\
  &= 0
\end{aligned}$$

Reference: Stanford BMI 214, Lecture by Emily Flynn, 10/8/2019
