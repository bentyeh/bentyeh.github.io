---
title: Feature Attribution in Deep Neural Networks
layout: post
use_toc: true
use_math: true
excerpt: Saliency maps, Integrated Gradients, and DeepLIFT explained.
hidden: true
tags: [deep-learning, model-interpretation]
category: deep-learning
---

Let $$f: \mathbb{R}^n \rightarrow \mathbb{R}$$ be the function represented by a neural network.

Let $$x = (x_1, ..., x_n) \in \mathbb{R}^n$$ be the input vector of features for which we want to assign attributions $$\phi_i$$. Let $$x' \in \mathbb{R}^n$$ be a reference feature vector for methods that require a reference.

# Evaluation of methods

As [Sundararajan et al.](#references) note, attribution methods "are hard to evaluate empirically." 

> For instance, in the context of an object recognition task, (Samek et al., 2015) suggests that we select the top k pixels by attribution and randomly vary their intensities and then measure the drop in score. If the attribution method is good, then the drop in score should be large. However, the images resulting from pixel perturbation could be unnatural, and it could be that the scores drop simply because the network has never seen anything like it in training.

Consequently, many papers have proposed sets of desirable properties for attribution methods.

- **Sensitivity**: If $$x$$ and $$x'$$ differ in one feature $$i$$ but have different predictions, then feature $$i$$ is assigned a non-zero attribution.
  - Also known as: Sensitivity(a) [[Sundararajan et al.]](#references)
  - Sensitivity implies that the attribution method does not suffer from the **gradient saturation problem**, where the gradients of input features may be zero (or close to zero) "even if the network depends heavily on those features." [[Sturmfels et al.]](#references)
  - However, the converse is not necessarily true. An attribution method that does not suffer from the gradient saturation problem is not necessarily sensitive. Consider an attribution method that assigns a feature an attribution using DeepLIFT or integrated gradients whenever the gradient is (near) zero, or an attribution of 0 otherwise.
- **Specificity**: If $$f$$ does not depend (mathematically) on feature $$i$$, then feature $$i$$ is assigned zero attribution.
  - Also known as: Sensitivity(b) [[Sundararajan et al.]](#references)
- **Continuity**: Attribution scores are continuous, even where gradients are discontinuous. [[Shrikumar et al.](#references)]
- **Completeness**: Attributions add up to the difference between the output of $$f$$ at the input $$x$$ and the reference $$x'$$.
  - Also known as: Summation-to-delta [[Shrikumar et al.]](#references), Local accuracy [[Lundberg & Lee]](#references)
- **Implementation invariance**: Attributions are always identical for two functionally equivalent networks.
- **Linearity**: Let $$g$$ and $$h$$ be functions each modeled by a deep network. The attributions for $$a \cdot g + b \cdot h$$ should be the weighted sum of the attributions for $$g$$ and $$h$$ with weights $$a$$ and $$b$$, respectively. The attributions should preserve any linearity within the network. [[Sundararajan et al.]](#references)
- **Symmetry-Preserving**: Symmetric features with identical values receive identical attributions. [[Sundararajan et al.]](#references)
  - Symmetric features are features that, if swapped, do not change the value of the function. For example, $$x_1$$ and $$x_2$$ are symmetric with respect to $$f$$ if and only if $$f(x_1, x_2) = f(x_2, x_1)$$ for all values of $$x_1, x_2$$.

# Summary

| Property                    | Gradient | Guided backprop | DeepLIFT | Integrated gradients |
| --------------------------- | -------- | --------------- | -------- | -------------------- |
| Reference-based             | No       | No              | Yes      | Yes                  |
| Attribution $$\phi_i(x)$$ | $$\nabla f_i(x)$$ | Discard negative gradients at ReLUs | Backpropagation with discrete gradients | $$(x_i - x_i') \int_0^1 \frac{\partial f(x' + \alpha(x - x'))}{\partial x_i} d\alpha$$ |
| Sensitivity                 | No       | No              | Yes      | Yes                  |
| Specificity                 | Yes      | Yes             |          | Yes                  |
| Continuity                  | No       | No              | Yes      | Yes                  | 
| Completeness                | No       | No              | Yes      | Yes                  |
| Implementation-invariant    | Yes      |                 | No       | Yes                  |
| Symmetry-preserving         | Yes      | Yes             | Yes      | Yes                  |
| Linearity                   | Yes      | Yes             |          | Yes                  |

# Gradients

$$\phi(x) = \nabla f(x)$$

Gradients roughly generalize the idea of using weights in a linear model for assigning feature importance. Compare a simple linear model

$$f_\text{linear}(x) = b + w \cdot x$$

with the linearization of $$f$$

$$\begin{aligned}
f(x) &\approx f(x') + \nabla f(x') \cdot (x - x') \\
&= \underbrace{\left(f(x') - \nabla f(x') \cdot x' \right)}_b + \underbrace{\nabla f(x')}_w \cdot x
\end{aligned}$$

The weights in the linearized model are $$\nabla f(x')$$. Note that unlike the simple linear model, the weights in the linearized model depends on the reference $$x'$$. For model interpretation, the reference $$x'$$ is usually chosen as the input $$x$$ itself.

Properties
- Saturation: No
  - Let $$f(x) = 1 - \max(0, 1 - x)$$ with gradient

    $$\nabla f(x) = \begin{cases}
      1 & x < 1 \\
      0 & x > 1 \\
      \text{undefined} & x = 1
    \end{cases}$$
    
    Consider the value of $$f$$ at two inputs: $$f(0) = 0$$, $$f(2) = 1$$. The gradient at both inputs is 0.
- Sensitivity: ??
- Reference-based: No
  - No *additional* reference is usually needed because the gradient is usually taken at the input.
  - However, testing a method for saturation at an input, or otherwise interpreting the score returned by the gradient, inherently requires a reference to compare against.
- Implementation-invariant: Yes (by Chain Rule)

## Variants

Guided backpropagation: any gradients that become negative during the backward pass are discarded at ReLUs.

Consider intermediate layers $$R^{l+1} = \mathrm{ReLU}(R^l)$$.
- Gradients: $$\frac{\partial f}{\partial R^l_i} = \frac{\partial f}{\partial R^{l+1}_i} \cdot 1(R^l_i > 1)$$
- DeconvNet: $$\frac{\partial f}{\partial R^l_i} = \frac{\partial f}{\partial R^{l+1}_i} \cdot 1(R^{l+1}_i > 1)$$ [[Zeiler & Fergus]](#references)
- Guided backpropagation: $$\frac{\partial f}{\partial R^l_i} = \frac{\partial f}{\partial R^{l+1}_i} \cdot 1(R^l_i > 1) \cdot 1(R^{l+1}_i > 1)$$

<span style="color: rgb(255, 155, 155)">Why only at ReLUs? How are these methods different at other nonlinearities?</span>
> The ’deconvolution’ is equivalent to a backward pass through the network, except that when propagating through a nonlinearity, its gradient is solely computed based on the top gradient signal, ignoring the bottom input. In case of the ReLU nonlinearity this amounts to setting to zero certain entries based on the top gradient.

# Integrated gradients

Idea: To overcome saturation and discontinuity problems, accumulate local gradients from a neutral reference to the input.

Let $$C$$ be a curve from a reference $$x'$$ to input $$x$$. Let $$\gamma(\alpha) \in \mathbb{R}^n$$ parameterize $$C$$ for $$\alpha \in [0, 1]$$

$$\gamma(\alpha) = x' + \alpha(x - x')$$

so $$\gamma(0) = x'$$ and $$\gamma(1) = x$$. For a given input feature $$x_i$$, we accumulate gradients along the curve $$C$$ as the integral

$$
\frac{\delta f(\gamma(\alpha))}{\delta \alpha}
= \sum_i \frac{\delta f(\gamma(\alpha))}{\delta \gamma_i} \times  
  \frac{\delta \gamma_i(\alpha)}{\delta \alpha}
= \sum_i \frac{\delta f(x' + \alpha' (x - x'))}{\delta x_i} \times (x_i - x'_i)
$$

$$\begin{equation} \label{eq:integrated_gradients} \begin{aligned}
\phi_i(x)
&= \int_{0}^{1} \frac{\partial f(\gamma(\alpha))}{\partial \gamma_i} \frac{d\gamma_i(\alpha)}{d\alpha} d\alpha \\
&= \int_{0}^{1} \frac{\partial f(x' + \alpha(x - x'))}{\partial x_i} (x_i - x_i') d\alpha \\
&= (x_i - x_i') \int_{0}^{1} \frac{\partial f(x' + \alpha(x - x'))}{\partial x_i} d\alpha
\end{aligned} \end{equation}$$

Each attribution $$\phi_i(x)$$ is one term in the path integral

$$\begin{equation} \label{eq:ig_path} \begin{aligned}
\int_C \nabla f(\gamma) \cdot d\gamma
&= \int_{\gamma(0) = x'}^{\gamma(1) = x} \nabla f(\gamma) \cdot d\gamma \\
&= \int_0^1 \nabla f (\gamma(\alpha)) \cdot \frac{d \gamma(\alpha)}{d\alpha} d\alpha \\
&= \int_0^1 \sum_{i=1}^n \frac{\partial f(\gamma(\alpha))}{\partial \gamma_i} \frac{d \gamma_i(\alpha)}{d\alpha} d\alpha \\
&= \sum_{i=1}^n \int_0^1 \frac{\partial f(\gamma(\alpha))}{\partial \gamma_i} \frac{d \gamma_i(\alpha)}{d\alpha} d\alpha \\
&= \sum_{i=1}^n \phi_i(x)
\end{aligned} \end{equation}$$

By the [Fundamental Theorem of Line Integrals](https://en.wikipedia.org/wiki/Gradient_theorem),

$$\int_{x'}^{x} \nabla f(\gamma) \cdot d\gamma = f(x) - f(x')$$

This yields 2 important observations:
1. The value of the path integral does not depend on the specific path $$C$$ taken from $$x'$$ to $$x$$.
2. The attributions assigned by integrated gradients satisfies the completeness property that the sum of attributions over all features equals the difference between the prediction on the input and the prediction on the reference. If the reference $$x'$$ is chosen such that the corresponding prediction is $$f(x') = 0$$, then the sum of attributions over all features simply equals the prediction on the input.
   - Furthermore, if the reference and input differ in only one feature $$k$$ (i.e., $$x_i = x_i'$$ for $$i \neq k$$), then integrated gradients attributes the difference in output values solely to variable $$x_k$$:
   
     $$\phi_i(x) = \begin{cases} 0, & i \neq k \\ f(x) - f(x'), & i = k \end{cases}$$
   
     This can be readily shown by observing that $$\phi_i(x)$$ is directly proportional to $$(x_i - x_i')$$; if $$x_i = x_i'$$, then $$\phi_i(x) = 0$$.
   - If the reference and input differ in more than one feature, then integrated gradients assigns attributions in a symmetry-preserving way. [[Sundararajan et al.]](#references)

Integrated gradients can be approximated by [Riemann sums](https://en.wikipedia.org/wiki/Riemann_sum). The paper describes a right Riemann sum of $$m$$ steps

$$\begin{aligned}
\phi_i(x)
&= (x_i - x_i') \int_{0}^{1} \frac{\partial f(x' + \alpha(x - x'))}{\partial x_i} d\alpha \\
&= (x_i - x_i') \sum_{k=1}^m \frac{\partial f(x' + \frac{k}{m}(x - x'))}{\partial x_i} \frac{1}{m} \\
&= \frac{x_i - x'_i}{m} \sum_{k=1}^m \frac{\partial f(x' + \frac{k}{m}(x - x'))}{\partial x_i} \\
&= \Delta \text{input}_i \cdot \frac{1}{m} \sum_{k=1}^m \frac{\partial f(x' + \frac{k}{m}(x - x'))}{\partial x_i} \\
&= \Delta \text{input}_i \cdot \text{average gradient}
\end{aligned}$$

where $$\Delta \text{input}_i = x_i - x'_i$$. The final equality demonstrates that integrated gradients is equivalent to the average gradient times the difference between the input and the reference, as mentioned in the [DeepLIFT video tutorials.](https://www.youtube.com/watch?v=1mrbsS7inU4&feature=youtu.be&list=PLJLjQOkqSRTP3cLB2cOOi_bQFw6KPGKML&t=439) Note that other sums (e.g., left, midpoint, trapezoidal, etc.) can be used instead.

<details markdown="block"><summary>Additional details.</summary>

1. How did the denominator $$\partial \gamma_i$$ change to $$\partial x_i$$ in Equation $$\ref{eq:integrated_gradients}$$?
   - Short answer: The denominator in the differential operator specifies the direction along which to take the derivative. $$\gamma_i$$ and $$x_i$$ refer to the same direction (dimension).
   - Long answer: Consider a different notation for the parameterized function of the curve $$C$$. Let $$x'$$ be the reference as before, but let the symbol $$\tilde{x}$$ denote the input. Then the position vector $$x$$ paramterized by $$\alpha$$ is given by
   
     $$x(\alpha) = x' + \alpha(\tilde{x} - x')$$
   
     To clarify, $$x'$$ and $$\tilde{x}$$ are constant vectors, while $$x$$ is now used as a variable denoting an arbitrary argument to the function $$f$$. Equation $$\ref{eq:integrated_gradients}$$ can then be rewritten as

     $$\begin{aligned}
     \phi_i(x)
     &= \int_{0}^{1} \frac{\partial f(x(\alpha))}{\partial x_i} \frac{dx_i(\alpha)}{d\alpha} d\alpha \\
     &= \int_{0}^{1} \frac{\partial f(x' + \alpha(\tilde{x} - x'))}{\partial x_i} (\tilde{x}_i - x_i') d\alpha \\
     &= (\tilde{x}_i - x_i') \int_{0}^{1} \frac{\partial f(x' + \alpha(\tilde{x} - x'))}{\partial x_i} d\alpha
     \end{aligned}$$

     - All we did was rename the symbol $$x$$ to $$\tilde{x}$$, and the symbol $$\gamma$$ to $$x$$. However, this makes it clearer that the reference and the input vectors are two *constant* vectors specifying the integration path, while the symbol used in the denominator ($$\partial \gamma_i$$ or $$\partial x_i$$) is a *variable* argument to $$f$$.

2. By the Chain Rule,

   $$\frac{df}{d\alpha}(\alpha) = \sum_{i=1}^n \frac{\partial f}{\partial \gamma_i}(\gamma(\alpha)) \cdot \frac{d \gamma_i}{d \alpha}(\alpha)$$

   Therefore, the path integral (Equation $$\ref{eq:ig_path}$$) is equivalent to

   $$\int_C \nabla f(\gamma) \cdot d\gamma = \int_0^1 \frac{df}{d\alpha} d\alpha$$

   See page 776 in Thomas' Calculus, 12th Edition for an explanation of the Chain Rule notation used here that explicitly shows where the derivatives are evaluated.

</details>

Code references
- [GitHub respository](https://github.com/ankurtaly/Integrated-Gradients)
  - [Relevant code file](https://github.com/ankurtaly/Integrated-Gradients/blob/master/IntegratedGradients/integrated_gradients.py)
- [TensorFlow tutorial](https://www.tensorflow.org/tutorials/interpretability/integrated_gradients)
  - [More in-depth tutorial](https://github.com/GoogleCloudPlatform/training-data-analyst/blob/master/blogs/integrated_gradients/integrated_gradients.ipynb)

# DeepLIFT

Completeness: $$\sum_i C_{\Delta x_i \Delta y} = \Delta y$$

Chain rule for multipliers: $$m_{\Delta x_i \Delta t} = \sum_j m_{\Delta x_i \Delta y_j} m_{\Delta y_j \Delta t}$$
- Separated into positive and negative contributions: $$m_{\Delta x_i \Delta t} = \sum_j \sum_{* \in \{+, -\}} m_{\Delta x_i \Delta y_j^*} m_{\Delta y_j^* \Delta t}$$

## Linear layers

Note: This applies to any layer (e.g., convolution) that can be computed using a linear layer.

Consider a neuron in a linear layer $$y = b + \sum_i w_i x_i$$. The difference from reference is computed linearly as $$\Delta y = \sum_i w_i \Delta x_i$$ and can be separated into positive and negative contributions $$\Delta y = \Delta y^+ + \Delta y^-$$ where

$$\begin{aligned}
\Delta y^+
  &= \sum_i 1\{w_i \Delta x_i > 0\} w_i \Delta x_i \\
  &= \sum_i 1\{w_i \Delta x_i > 0\} w_i (\Delta x_i^+ + \Delta x_i^-) \\
\Delta y^-
  &= \sum_i 1\{w_i \Delta x_i < 0\} w_i \Delta x_i \\
  &= \sum_i 1\{w_i \Delta x_i < 0\} w_i (\Delta x_i^+ + \Delta x_i^-) \\
\end{aligned}$$ 

DeepLIFT's Linear rule proposes distributing contributions as follows:
- If $$\Delta x_i \neq 0$$:
  - []() $$C_{\Delta x_i^+ \Delta y^+} = 1\{w_i \Delta x_i > 0\} w_i \Delta x_i^+$$
  - []() $$C_{\Delta x_i^- \Delta y^+} = 1\{w_i \Delta x_i > 0\} w_i \Delta x_i^-$$
  - []() $$C_{\Delta x_i^+ \Delta y^-} = 1\{w_i \Delta x_i < 0\} w_i \Delta x_i^+$$
  - []() $$C_{\Delta x_i^- \Delta y^-} = 1\{w_i \Delta x_i < 0\} w_i \Delta x_i^-$$
- If $$\Delta x_i = 0$$:
  - []() $$C_{\Delta x_i^+ \Delta y^+} = 0.5 w_i \Delta x_i^+$$
  - []() $$C_{\Delta x_i^- \Delta y^+} = 0.5 w_i \Delta x_i^-$$
  - []() $$C_{\Delta x_i^+ \Delta y^-} = 0.5 w_i \Delta x_i^+$$
  - []() $$C_{\Delta x_i^- \Delta y^-} = 0.5 w_i \Delta x_i^-$$

If we only separate positive and negative contributions for either the input neurons $$x_i$$ or the target neuron $$y$$, we get (equations below hold regardless of whether $$\Delta x_i = 0$$)
- []() $$C_{\Delta x_i^+ \Delta y} = C_{\Delta x_i^+ \Delta y^+} + C_{\Delta x_i^+ \Delta y^-} = w_i \Delta x_i^+$$
- []() $$C_{\Delta x_i^- \Delta y} = C_{\Delta x_i^- \Delta y^+} + C_{\Delta x_i^- \Delta y^-} = w_i \Delta x_i^-$$
- []() $$C_{\Delta x_i \Delta y^+} = C_{\Delta x_i^+ \Delta y^+} + C_{\Delta x_i^- \Delta y^+} = 1\{w_i \Delta x_i > 0\} w_i \Delta x_i = \Delta y^+$$
- []() $$C_{\Delta x_i \Delta y^-} = C_{\Delta x_i^+ \Delta y^-} + C_{\Delta x_i^- \Delta y^-} = 1\{w_i \Delta x_i < 0\} w_i \Delta x_i  = \Delta y^-$$

Thus, the unseparated contributions can be expressed as

$$\begin{aligned}
C_{\Delta x_i \Delta y}
&= w_i \Delta x_i \\
&= C_{\Delta x_i \Delta y^+} + C_{\Delta x_i \Delta y^-} \\
&= C_{\Delta x_i^+ \Delta y} + C_{\Delta x_i^- \Delta y} \\
&= C_{\Delta x_i^+ \Delta y^+} + C_{\Delta x_i^- \Delta y^+} + C_{\Delta x_i^+ \Delta y^-} + C_{\Delta x_i^- \Delta y^-}
\end{aligned}$$

The multipliers are therefore
- []() $$m_{\Delta x_i \Delta y} = m_{\Delta x_i^+ \Delta y} = m_{\Delta x_i^- \Delta y} = w_i$$
- If $$\Delta x_i \neq 0$$:
  - []() $$m_{\Delta x_i \Delta y^+} = m_{\Delta x_i^+ \Delta y^+} = m_{\Delta x_i^- \Delta y^+} = 1\{w_i \Delta x_i > 0\} w_i$$
  - []() $$m_{\Delta x_i \Delta y^-} = m_{\Delta x_i^+ \Delta y^-} = m_{\Delta x_i^- \Delta y^-} = 1\{w_i \Delta x_i < 0\} w_i$$
- If $$\Delta x_i = 0$$:
  - []() $$m_{\Delta x_i \Delta y^+} = m_{\Delta x_i \Delta y^-} = m_{\Delta x_i^+ \Delta y^+} = m_{\Delta x_i^- \Delta y^+} = m_{\Delta x_i^+ \Delta y^-} = m_{\Delta x_i^- \Delta y^-} = 0.5 w_i$$

## Element-wise operations

This applies to nonlinear transformations like the ReLU, tanh or sigmoid operations.

Consider a neuron $$y = f(x)$$ where both $$x$$ and $$y$$ are scalars. By the completeness property, $$C_{\Delta x \Delta y} = \Delta y$$ and therefore $$m_{\Delta x \Delta y} = \frac{\Delta y}{\Delta x}$$.

### Option 1: Rescale rule

DeepLIFT's Rescale rule proposes separating positive and negative components of $$\Delta y$$ as

$$\begin{aligned}
\Delta y^+
  &= 1\{\Delta y > 0 \} \Delta y \\
  &= 1\{\Delta y > 0 \} \frac{\Delta y}{\Delta x} (\Delta x^+ + \Delta x^-) \\
\Delta y^-
  &= 1\{\Delta y < 0 \} \Delta y \\
  &= 1\{\Delta y < 0 \} \frac{\Delta y}{\Delta x} (\Delta x^+ + \Delta x^-) \\
\end{aligned}$$

which suggests splitting contributions as follows:
- []() $$C_{\Delta x^+ \Delta y^+} = 1\{\Delta y > 0 \} \frac{\Delta y}{\Delta x} \Delta x^+$$
- []() $$C_{\Delta x^- \Delta y^+} = 1\{\Delta y > 0 \} \frac{\Delta y}{\Delta x} \Delta x^-$$
- []() $$C_{\Delta x^+ \Delta y^-} = 1\{\Delta y < 0 \} \frac{\Delta y}{\Delta x} \Delta x^+$$
- []() $$C_{\Delta x^- \Delta y^-} = 1\{\Delta y < 0 \} \frac{\Delta y}{\Delta x} \Delta x^-$$

If we only separate positive and negative contributions for either the input neurons $$x$$ or the target neuron $$y$$, we get
- []() $$C_{\Delta x^+ \Delta y} = C_{\Delta x^+ \Delta y^+} + C_{\Delta x^+ \Delta y^-} = \frac{\Delta y}{\Delta x} \Delta x^+$$
- []() $$C_{\Delta x^- \Delta y} = C_{\Delta x^- \Delta y^+} + C_{\Delta x^- \Delta y^-} = \frac{\Delta y}{\Delta x} \Delta x^-$$
- []() $$C_{\Delta x \Delta y^+} = C_{\Delta x^+ \Delta y^+} + C_{\Delta x^- \Delta y^+} = 1\{\Delta y > 0 \} \frac{\Delta y}{\Delta x} (\Delta x^+ + \Delta x^-) = \Delta y^+$$
- []() $$C_{\Delta x \Delta y^-} = C_{\Delta x^+ \Delta y^-} + C_{\Delta x^- \Delta y^-} = 1\{\Delta y < 0 \} \frac{\Delta y}{\Delta x} (\Delta x^+ + \Delta x^-) = \Delta y^-$$

Thus, the unseparated contributions can be expressed as

$$\begin{aligned}
C_{\Delta x \Delta y}
&= \Delta y \\
&= C_{\Delta x \Delta y^+} + C_{\Delta x \Delta y^-} \\
&= C_{\Delta x^+ \Delta y} + C_{\Delta x^- \Delta y} \\
&= C_{\Delta x^+ \Delta y^+} + C_{\Delta x^- \Delta y^+} + C_{\Delta x^+ \Delta y^-} + C_{\Delta x^- \Delta y^-}
\end{aligned}$$

The multipliers are therefore
- []() $$m_{\Delta x \Delta y} = m_{\Delta x^+ \Delta y} = m_{\Delta x^- \Delta y} = \frac{\Delta y}{\Delta x}$$
- []() $$m_{\Delta x \Delta y^+} = m_{\Delta x^+ \Delta y^+} = m_{\Delta x^- \Delta y^+} = 1\{\Delta y > 0 \} \frac{\Delta y}{\Delta x}$$
- []() $$m_{\Delta x \Delta y^-} = m_{\Delta x^+ \Delta y^-} = m_{\Delta x^- \Delta y^-} = 1\{\Delta y < 0 \} \frac{\Delta y}{\Delta x}$$

Notes
- If $$\Delta x \approx 0$$, replace $$\frac{\Delta y}{\Delta x}$$ everywhere with $$\frac{dy}{dx}$$ evaluated at $$x = x'$$.
- The Rescale rule described in the DeepLIFT paper ignores the contributions $$C_{\Delta x^- \Delta y^+}$$ and $$C_{\Delta x^- \Delta y^-}$$ (and the corresponding multipliers) as it assumes that the element-wise function $$g$$ is monotonic, which is true for many widely-used activation functions: ReLU, tanh, and sigmoid. However, there exist non-monotonic element-wise activation functions, such as Swish, for which $$C_{\Delta x^- \Delta y^+}$$ and $$C_{\Delta x^- \Delta y^-}$$ may be nonzero.

### Option 2: RevealCancel rule

The RevealCancel rule splits contributions into positive and negative components as

$$\Delta y^+ = \frac{1}{2} \left(f(x' + \Delta x^+) - f(x')\right) + \frac{1}{2} \left(f(x' + \Delta x^- + \Delta x^+) - f(x' + \Delta x^-) \right)$$

$$\Delta y^- = \frac{1}{2} \left(f(x' + \Delta x^-) - f(x')\right) + \frac{1}{2} \left(f(x' + \Delta x^- + \Delta x^+) - f(x' + \Delta x^+) \right)$$

$$m_{\Delta x^+ \Delta y^+} = \frac{C_{\Delta x^+ \Delta y^+}}{\Delta x^+} = \frac{\Delta y^+}{\Delta x^+}$$

$$m_{\Delta x^- \Delta y^-} = \frac{C_{\Delta x^- \Delta y^-}}{\Delta x^-} = \frac{\Delta y^-}{\Delta x^-}$$

<span style="color: red">How would the RevealCancel rule assign contributions $$C_{\Delta x^- \Delta y^+}$$ and $$C_{\Delta x^- \Delta y^-}$$ for non-monotonic element-wise functions?</span>

## Other layers

DeepLIFT does not support non-linear multiple-input layers.

## Example

Consider applying the Linear and Rescale rules to the following network. We show that the equations for splitting contributions and multipliers into positive and negative components is consistent with the non-split equations.

<img style="width: 100%" src="{{ site.baseurl }}/images/posts_attribution_network.svg">

Without splitting into positive and negative components

$$\begin{aligned}
m_{\Delta x_1 \Delta y}
&= m_{\Delta x_1 \Delta h_1} m_{\Delta h_1 \Delta a_1} m_{\Delta a_1 \Delta y} + m_{\Delta x_1 \Delta h_2} m_{\Delta h_2 \Delta a_2} m_{\Delta a_2 \Delta y} \\
&= w_{11} \frac{\Delta a_1}{\Delta h_1} \beta_1 + w_{12} \frac{\Delta a_2}{\Delta h_2} \beta_2
\end{aligned}$$

Using split components

$$\begin{alignat*}{2}
m_{\Delta x_1 \Delta y}
& = && m_{\Delta x_1 \Delta h_1^+} (m_{\Delta h_1^+ \Delta a_1^+} m_{\Delta a_1^+ \Delta y} + m_{\Delta h_1^+ \Delta a_1^-} m_{\Delta a_1^- \Delta y}) + {} \\
&&& m_{\Delta x_1 \Delta h_1^-} (m_{\Delta h_1^- \Delta a_1^+} m_{\Delta a_1^+ \Delta y} + m_{\Delta h_1^- \Delta a_1^-} m_{\Delta a_1^- \Delta y}) + {} \\
&&& m_{\Delta x_1 \Delta h_2^+} (m_{\Delta h_2^+ \Delta a_2^+} m_{\Delta a_2^+ \Delta y} + m_{\Delta h_2^+ \Delta a_2^-} m_{\Delta a_2^- \Delta y}) + {} \\
&&& m_{\Delta x_1 \Delta h_2^+} (m_{\Delta h_2^+ \Delta a_2^+} m_{\Delta a_2^+ \Delta y} + m_{\Delta h_2^+ \Delta a_2^-} m_{\Delta a_2^- \Delta y}) \\
& = && 1\{w_{11} \Delta x_1 > 0\} w_{11} \left(1\{\Delta a_1 > 0\} \frac{\Delta a_1}{\Delta h_1} \beta_1 + 1\{\Delta a_1 < 0\} \frac{\Delta a_1}{\Delta h_1} \beta_1 \right) + {} \\
&&& 1\{w_{11} \Delta x_1 < 0\} w_{11} \left(1\{\Delta a_1 > 0\} \frac{\Delta a_1}{\Delta h_1} \beta_1 + 1\{\Delta a_1 < 0\} \frac{\Delta a_1}{\Delta h_1} \beta_1 \right) + {} \\
&&& 1\{w_{12} \Delta x_1 > 0\} w_{12} \left(1\{\Delta a_2 > 0\}\frac{\Delta a_2}{\Delta h_2} \beta_2 + 1\{\Delta a_2 < 0\} \frac{\Delta a_2}{\Delta h_2} \beta_2 \right) + {} \\
&&& 1\{w_{12} \Delta x_1 < 0\} w_{12} \left(1\{\Delta a_2 > 0\}\frac{\Delta a_2}{\Delta h_2} \beta_2 + 1\{\Delta a_2 < 0\} \frac{\Delta a_2}{\Delta h_2} \beta_2 \right) \\
& = && 1\{w_{11} \Delta x_1 > 0\} w_{11} \left(\frac{\Delta a_1}{\Delta h_1} \beta_1 \right) + {} \\
&&& 1\{w_{11} \Delta x_1 < 0\} w_{11} \left(\frac{\Delta a_1}{\Delta h_1} \beta_1 \right) + {} \\
&&& 1\{w_{12} \Delta x_1 > 0\} w_{12} \left(\frac{\Delta a_2}{\Delta h_2} \beta_2 \right) + {} \\
&&& 1\{w_{12} \Delta x_1 < 0\} w_{12} \left(\frac{\Delta a_2}{\Delta h_2} \beta_2 \right) \\
& = && w_{11} \frac{\Delta a_1}{\Delta h_1} \beta_1 + w_{12} \frac{\Delta a_2}{\Delta h_2} \beta_2
\end{alignat*}$$

Thus

$$C_{\Delta x_1 \Delta y} = m_{\Delta x_1 \Delta y} \Delta x_1 = \left(w_{11} \frac{\Delta a_1}{\Delta h_1} \beta_1 + w_{12} \frac{\Delta a_2}{\Delta h_2} \beta_2 \right) \Delta x_1$$

# Notes

- Assigning attributions using gradients is often referred to as creating a "saliency map" due to the paper [[Simonyan et al.]](#references) that proposed the method. However, the term "saliency map" can more generally refer to a tensor of the same shape as the input that identifies "parts of an image [that] are most important to a network's decisions." [[Mundhenk et al.]](#references) In the general sense, any assignment of attributions to input features is a saliency map; gradients is only one type of saliency map.
- For multichannel (e.g., RGB) images, it may be desirable to obtain a pixel attribution by aggregating attributions along the channels.
  - When using gradients as attributions, Simonyan et al. use the maximum magnitude along all channels, while Sundararajan et al. use the sum of magnitudes along all channels.
- For visualizing attributions, Sundararajan et al. scale the intensities of pixels RGB values in the original image in proportion to their attributions.

# References

Cited works
1. Zeiler, M. D. & Fergus, R. Visualizing and Understanding Convolutional Networks. in *EECV 2014* (2014). [arXiv:1311.2901](https://arxiv.org/abs/1311.2901).
   - Deconvolutional Network
2. Simonyan, K., Vedaldi, A. & Zisserman, A. Deep Inside Convolutional Networks: Visualising Image Classification Models and Saliency Maps. in *ICLR 2014 - Workshop Track Proceedings* (2014). [arXiv:1312.6034](https://arxiv.org/abs/1312.6034).
   - Saliency maps, input that optimizes class score.
3. Springenberg, J. T., Dosovitskiy, A., Brox, T. & Riedmiller, M. Striving for Simplicity: The All Convolutional Net. in *3rd International Conference on Learning Representations, ICLR 2015 - Workshop Track Proceedings*. [arXiv:1412.6806](https://arxiv.org/abs/1412.6806).
   - Guided backpropagation.
4. Sundararajan, M., Taly, A. & Yan, Q. Axiomatic Attribution for Deep Networks. in *Proceedings of the 34th International Conference on Machine Learning* (eds. Precup, D. & Teh, Y. W.) 7, 3319–3328 (2017). [arXiv:1703.01365](https://arxiv.org/abs/1703.01365).
   - Integrated gradients and desired properties of attribution methods.
   - First submitted to arXiv as a preprint titled ["Gradients of Counterfactuals" (2016)](https://arxiv.org/abs/1611.02639) but was [rejected by ICLR 2017](https://openreview.net/forum?id=rJzaDdYxx).
5. Lundberg, S. M. & Lee, S.-I. A Unified Approach to Interpreting Model Predictions. in *Proceedings of the 31st International Conference on Neural Information Processing Systems* 4768–4777 (2017). [arXiv:1705.07874](https://arxiv.org/abs/1705.07874).
   - SHAP.
6. Shrikumar, A., Greenside, P. & Kundaje, A. Learning Important Features Through Propagating Activation Differences. in *Proceedings of the 34th International Conference on Machine Learning, ICML 2017* 70, 3145–3153 (Journal of Machine Learning Research, 2017). [arXiv:1704.02685](https://arxiv.org/abs/1704.02685).
   - DeepLIFT.
   - The citation above refers to the updated preprint. The original prepring submitted to arXiv in 2016 is titled ["Not Just a Black Box: Learning Important Features Through Propagating Activation Differences."](https://arxiv.org/abs/1605.01713)

Other helpful resources
- Mundhenk, T. N., Chen, B. Y. & Friedland, G. Efficient Saliency Maps for Explainable AI. [arXiv:1911.11293](https://arxiv.org/abs/1911.11293) (2020).
- Adebayo, J. et al. Sanity Checks for Saliency Maps. in *Proceedings of the 32nd International Conference on Neural Information Processing Systems* 9525–9536 (2018). [arXiv:1810.03292](https://arxiv.org/abs/1810.03292).
- Sixt, L., Granz, M. & Landgraf, T. When Explanations Lie: Why Many Modified BP Attributions Fail. in *Proceedings of Machine Learning and Systems 2020* 1755–1765 (2020). [arXiv:1912.09818](https://arxiv.org/abs/1912.09818).
- Sturmfels, et al. Visualizing the Impact of Feature Attribution Baselines. *Distill*, 2020. [doi:10.23915/distill.00022](https://distill.pub/2020/attribution-baselines/).
- Molnar, Christoph. *Interpretable Machine Learning: A Guide for Making Black Box Models Explainable.* [https://christophm.github.io/interpretable-ml-book/](https://christophm.github.io/interpretable-ml-book/).