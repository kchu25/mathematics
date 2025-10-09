@def title = "Hard Thresholding: A Mathematical Overview"
@def published = "9 October 2025"
@def tags = ["convex-optimization"]

# Hard Thresholding: A Mathematical Overview

## 1. Definition

### Hard Thresholding Operator
$H_\lambda(x) = \begin{cases} 
x & \text{if } |x| > \lambda \\
0 & \text{if } |x| \leq \lambda
\end{cases}$

### Soft Thresholding Operator (for comparison)
$S_\lambda(x) = \begin{cases} 
x - \lambda & \text{if } x > \lambda \\
0 & \text{if } |x| \leq \lambda \\
x + \lambda & \text{if } x < -\lambda
\end{cases} = \text{sign}(x) \cdot \max(|x| - \lambda, 0)$

### Visual Comparison

![Hard vs Soft Thresholding](/blog/convex_stuff/soft_hard.svg)

**Key Visual Differences:**

| Hard Thresholding | Soft Thresholding |
|-------------------|-------------------|
| Discontinuous at ±λ | Continuous everywhere |
| Identity mapping for \|x\| > λ | Shifted by λ (biased) |
| Flat (zero) for \|x\| ≤ λ | Flat (zero) for \|x\| ≤ λ |

**Key Properties:**
- **Hard**: Flat at zero for $|x| \leq \lambda$, then vertical jump to diagonal $y = x$ 
- **Soft**: Flat at zero for $|x| \leq \lambda$, then diagonal lines $y = x - \lambda$ (upper) and $y = x + \lambda$ (lower)

## 2. Optimization Formulation

### Proximal Operators

**Hard thresholding** is the proximal operator of the $\ell_0$ "norm":
$$\text{prox}_{\lambda \|\cdot\|_0}(y) = \arg\min_x \left\{ \frac{1}{2}\|x - y\|^2 + \lambda \|x\|_0 \right\} = H_{\sqrt{2\lambda}}(y)$$

where $\|x\|_0 = |\{i : x_i \neq 0\}|$ counts non-zero entries.

**Soft thresholding** is the proximal operator of the $\ell_1$ norm:
$$\text{prox}_{\lambda \|\cdot\|_1}(y) = \arg\min_x \left\{ \frac{1}{2}\|x - y\|^2 + \lambda \|x\|_1 \right\} = S_\lambda(y)$$

### Derivation of Hard Thresholding

For each coordinate $i$ independently, solve:
$$\min_{x_i} \left\{ \frac{1}{2}(x_i - y_i)^2 + \lambda \cdot \mathbb{1}_{x_i \neq 0} \right\}$$

Compare two cases:
- Set $x_i = 0$: cost $= \frac{1}{2}y_i^2 + \lambda$
- Set $x_i = y_i$: cost $= 0$

Choose $x_i = y_i$ when: $0 < \frac{1}{2}y_i^2 + \lambda$, giving $|y_i| > \sqrt{2\lambda}$.

Therefore: $x_i^* = H_{\sqrt{2\lambda}}(y_i)$

## 3. Mathematical Properties

### Comparison Table

| Property | Soft Thresholding | Hard Thresholding |
|----------|-------------------|-------------------|
| Penalty | $\lambda\|x\|_1$ (convex) | $\lambda\|x\|_0$ (non-convex) |
| Continuity | Continuous | Discontinuous at $\pm\lambda$ |
| Differentiability | Not at $\pm\lambda$ | Not at $\pm\lambda$ |
| Bias | Biased: $\mathbb{E}[S_\lambda(\theta + \varepsilon)] = \text{sign}(\theta)(\|\theta\| - \lambda)_+$ | Unbiased: $\mathbb{E}[H_\lambda(\theta + \varepsilon)] = \theta$ for $\|\theta\| > \lambda$ |
| Convexity | Convex problem | Non-convex (NP-hard in general) |

### Key Properties of Hard Thresholding

1. **Unbiasedness**: For $|\theta| > \lambda$, the estimator $H_\lambda(\theta + \varepsilon)$ is unbiased for $\theta$

2. **Oracle Property**: Under appropriate conditions, selects the true support set

3. **Idempotence**: $H_\lambda(H_\lambda(x)) = H_\lambda(x)$

4. **Scaling**: For $\alpha > 0$, $H_\lambda(\alpha x) = \alpha H_{\lambda/\alpha}(x)$

## 4. Practical Algorithms

### 4.1 Iterative Hard Thresholding (IHT)

**Problem**: Solve $\min_x \frac{1}{2}\|y - Ax\|^2$ subject to $\|x\|_0 \leq k$

**Algorithm**:
$$x^{(t+1)} = H_k\left(x^{(t)} + \mu A^\top(y - Ax^{(t)})\right)$$

where $H_k(z)$ keeps the $k$ largest magnitude entries of $z$ and zeros out the rest:
$$[H_k(z)]_i = \begin{cases} 
z_i & \text{if } i \in \text{indices of top-}k \text{ values of } |z| \\
0 & \text{otherwise}
\end{cases}$$

Step size: $\mu \in (0, 2/\|A\|^2)$

### 4.2 Hard Thresholding Pursuit (HTP)

**Algorithm**:
1. $x^{(t+1/2)} = x^{(t)} + \mu A^\top(y - Ax^{(t)})$ (gradient step)
2. $S = \text{support of top-}k \text{ entries of } x^{(t+1/2)}$
3. $x^{(t+1)}_S = \arg\min_z \|y - A_S z\|^2$ (least squares on support)
4. $x^{(t+1)}_{S^c} = 0$

### 4.3 Smoothed Hard Thresholding

To handle discontinuity, use a smooth approximation:
$$H_{\lambda,\delta}(x) = \begin{cases} 
0 & \text{if } |x| \leq \lambda - \delta \\
\frac{x}{2\delta}(|x| - \lambda + \delta) & \text{if } \lambda - \delta < |x| < \lambda + \delta \\
x & \text{if } |x| \geq \lambda + \delta
\end{cases}$$

As $\delta \to 0$, this converges to hard thresholding.

## 5. Theoretical Guarantees

### 5.1 Restricted Isometry Property (RIP)

Matrix $A \in \mathbb{R}^{m \times n}$ satisfies RIP of order $k$ with constant $\delta_k$ if:
$$(1 - \delta_k)\|x\|^2 \leq \|Ax\|^2 \leq (1 + \delta_k)\|x\|^2$$
for all $k$-sparse vectors $x$.

### 5.2 Convergence of IHT

**Theorem**: If $A$ satisfies RIP with $\delta_{3k} < 1/\sqrt{32}$, then IHT converges to the true $k$-sparse signal $x^*$:
$$\|x^{(t)} - x^*\| \leq \rho^t \|x^{(0)} - x^*\| + C\|\eta\|$$

where $\rho < 1$ and $\eta$ is measurement noise.

### 5.3 Sample Complexity

For $k$-sparse signals in $\mathbb{R}^n$:
- **$\ell_1$ minimization** (soft): requires $m = O(k \log(n/k))$ measurements
- **Hard thresholding**: requires $m = O(k \log(n/k))$ measurements (similar order, better constants)

### 5.4 MSE Comparison

For estimating $k$-sparse $\theta$ from $y = \theta + \varepsilon$ where $\varepsilon \sim \mathcal{N}(0, \sigma^2 I)$:

**Soft thresholding**: $\text{MSE} = \mathbb{E}\|S_\lambda(y) - \theta\|^2$
- Bias-variance trade-off
- Shrinks all coefficients

**Hard thresholding**: $\text{MSE} = \mathbb{E}\|H_\lambda(y) - \theta\|^2$
- Lower MSE when $\theta$ is truly sparse
- Higher variance near threshold

**Oracle property**: Under sparsity, hard thresholding achieves:
$$\text{MSE}_{\text{hard}} \approx k\sigma^2 + o(k\sigma^2)$$
which matches the oracle estimator that knows the true support.

## 6. Variants and Extensions

### 6.1 Firm Thresholding

Compromise between soft and hard:
$$F_{\lambda_1,\lambda_2}(x) = \begin{cases} 
0 & \text{if } |x| \leq \lambda_1 \\
\text{sign}(x) \cdot \frac{\lambda_2(|x| - \lambda_1)}{\lambda_2 - \lambda_1} & \text{if } \lambda_1 < |x| < \lambda_2 \\
x & \text{if } |x| \geq \lambda_2
\end{cases}$$

### 6.2 SCAD (Smoothly Clipped Absolute Deviation)

Penalty function:
$$p_\lambda(x) = \begin{cases} 
\lambda |x| & \text{if } |x| \leq \lambda \\
\frac{2a\lambda|x| - x^2 - \lambda^2}{2(a-1)} & \text{if } \lambda < |x| \leq a\lambda \\
\frac{\lambda^2(a+1)}{2} & \text{if } |x| > a\lambda
\end{cases}$$

Derivative gives a thresholding rule that transitions from soft to hard.

## 7. Computational Complexity

### Per-Iteration Cost

**Soft thresholding** (LASSO via proximal gradient):
- Gradient computation: $O(mn)$ for $A \in \mathbb{R}^{m \times n}$
- Soft threshold: $O(n)$
- Total: $O(mn)$

**Hard thresholding** (IHT):
- Gradient computation: $O(mn)$
- Find top-$k$ elements: $O(n \log k)$ using quickselect or $O(n)$ average case
- Total: $O(mn)$

Both have similar per-iteration cost, but convergence behavior differs.

## 8. When to Use Which

### Use Hard Thresholding When:
- True sparsity level $k$ is known
- Signal is truly sparse (many exact zeros)
- Unbiased estimates are critical
- Fast approximate solutions suffice
- Applications: compressed sensing with known sparsity, sparse neural network pruning

### Use Soft Thresholding When:
- Sparsity level is unknown
- Need convex optimization guarantees
- Robustness to noise is critical
- Signal is compressible but not exactly sparse
- Applications: LASSO regression, general regularization

### Use Intermediate Methods When:
- Want oracle properties with convexity
- Need smooth optimization landscape
- Applications: high-dimensional statistics (SCAD, MCP)