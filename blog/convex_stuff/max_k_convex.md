@def title = "Convexity of the Sum of k Largest Elements"
@def published = "9 October 2025"
@def tags = ["convex-optimization"]

# Convexity of the Sum of k Largest Elements

## The Result

The function $f(x) = \sum_{i=1}^{k} x_{[i]}$, which sums the k largest components of a vector $x \in \mathbb{R}^n$, is **convex**.

## Why It's True

The proof relies on a clever reformulation: the sum of the k largest elements can be written as:

$$f(x) = \max_{S \subseteq \{1,\ldots,n\}, |S|=k} \sum_{i \in S} x_i$$

In other words, $f(x)$ is the maximum sum we can get by choosing any k coordinates from $x$.

## The Key Insight

- Each choice of k coordinates gives us a **linear function**: $g_S(x) = \sum_{i \in S} x_i$
- Linear functions are convex
- Taking the **pointwise maximum** of convex functions yields a convex function
- Therefore, $f(x)$ is convex

## Intuition

Think of it this way: we're computing $\binom{n}{k}$ different linear functions (one for each way to choose k indices), and then taking their maximum. Since the maximum of any collection of convex functions is convex, we're done.

## Detailed Proof

**Proof (Maximum of Linear Functions):**

We express $f(x)$ as a maximum over all k-element subsets:

$f(x) = \max_{S \subseteq \{1,\ldots,n\}, |S|=k} \sum_{i \in S} x_i$

For each fixed subset $S$ with $|S| = k$, define $g_S(x) = \sum_{i \in S} x_i$. This is a linear function (hence convex).

Since $f(x) = \max_S g_S(x)$ is the pointwise maximum of convex functions, and the pointwise maximum of any collection of convex functions is convex, we conclude that $f$ is convex. $\square$

**Verification:** Let $x, y \in \mathbb{R}^n$ and $\lambda \in [0,1]$:

\[
\begin{aligned}
f(\lambda x + (1-\lambda)y) &= \max_{S:\, |S|=k} \sum_{i \in S} (\lambda x_i + (1-\lambda) y_i) \\
&= \max_{S:\, |S|=k} \big[\lambda \sum_{i \in S} x_i + (1-\lambda) \sum_{i \in S} y_i\big] \\
&\le \lambda \max_{S:\, |S|=k} \sum_{i \in S} x_i + (1-\lambda) \max_{S:\, |S|=k} \sum_{i \in S} y_i \\
&= \lambda f(x) + (1-\lambda) f(y)
\end{aligned}
\]

## How to Optimize It

Since $f(x) = \sum_{i=1}^{k} x_{[i]}$ is convex, we can use standard convex optimization techniques:

### 1. **Subgradient Methods**

The function is not differentiable everywhere, but we can compute subgradients. A subgradient at $x$ is:

$g \in \partial f(x) \text{ where } g_i = \begin{cases} 1 & \text{if } x_i \text{ is among the k largest (and unique)} \\ \in [0,1] & \text{if } x_i \text{ is on the boundary (tied for k-th place)} \\ 0 & \text{otherwise} \end{cases}$

**Subgradient descent:** $x^{t+1} = x^t - \alpha_t g^t$ where $g^t \in \partial f(x^t)$

### 2. **Proximal Operators**

For regularization problems like $\min_x \frac{1}{2}\|Ax - b\|^2 + \lambda \sum_{i=1}^{k} x_{[i]}$, we need the proximal operator:

$\text{prox}_{\lambda f}(y) = \arg\min_x \left\{ \frac{1}{2}\|x - y\|^2 + \lambda \sum_{i=1}^{k} x_{[i]} \right\}$

This can be computed efficiently using soft-thresholding with a careful calculation of the threshold value.

### 3. **Smoothing Approaches**

Replace the non-smooth max with a smooth approximation:

$f_\mu(x) = \mu \log\left(\sum_{S: |S|=k} \exp\left(\frac{1}{\mu}\sum_{i \in S} x_i\right)\right)$

As $\mu \to 0$, $f_\mu(x) \to f(x)$. This smooth version can be optimized with gradient-based methods.

### 4. **Reformulation as Linear Program**

Minimize $\sum_{i=1}^{k} x_{[i]}$ over $x$ subject to constraints by introducing auxiliary variables:

\[
\begin{aligned}
\min_{x, t} \quad & \sum_{j=1}^{k} t_j \\
\text{s.t.} \quad & t_1 \ge t_2 \ge \cdots \ge t_k \\
& t_j \ge x_i \quad \text{for all } i, j
\end{aligned}
\]

Plus any original constraints on $x$. This can be solved with LP solvers.

### 5. **Projected Subgradient for Constrained Problems**

For $\min_{x \in C} f(x)$ where $C$ is a convex set:

$x^{t+1} = \Pi_C(x^t - \alpha_t g^t)$

where $\Pi_C$ is projection onto $C$ and $g^t \in \partial f(x^t)$.

## Why This Matters

This result is used throughout convex optimization, particularly in:
- Robust optimization
- Sparse signal processing  
- Portfolio optimization (e.g., focusing on worst-case scenarios)
- The k-support norm and other regularizers