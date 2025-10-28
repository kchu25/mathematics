@def title = "K-Support Norm and Top-K Thresholding"
@def published = "28 October 2025"
@def tags = ["convex-optimization"]

# K-Support Norm and Top-K Thresholding

## The Main Result

The **top-k thresholding operator** (selecting the k largest elements in absolute value and zeroing out the rest) is the proximal operator of the **k-support norm**.

For top-30 selection specifically, this solves:

$$\min_z \frac{1}{2}\|x - z\|_2^2 + \lambda \|z\|_{k\text{-sup}}$$

where the k-support norm is defined below.

## Definition of K-Support Norm

The k-support norm of a vector $x \in \mathbb{R}^n$ is defined as:

$$\|x\|_{k\text{-sup}} = \min_{\substack{I \subseteq \{1,\ldots,n\} \\ |I| \leq k}} \left(\|x_I\|_2 + \|x_{I^c}\|_1\right)$$

where:
- $x_I$ denotes the subvector of $x$ indexed by $I$
- $I^c$ is the complement of $I$
- $|I|$ is the cardinality of $I$

**Equivalent characterization**: If we sort $|x|$ in decreasing order as $|x|_{(1)} \geq |x|_{(2)} \geq \cdots \geq |x|_{(n)}$, then:

$$\|x\|_{k\text{-sup}} = \sum_{i=1}^k |x|_{(i)} + \sqrt{k} \sqrt{\sum_{i=k+1}^n |x|_{(i)}^2}$$

This is the sum of the top-k absolute values plus $\sqrt{k}$ times the $\ell_2$ norm of the remaining elements.

## Proof of Convexity

**Theorem**: The k-support norm is convex.

**Proof**: We'll use the equivalent form:

$$\|x\|_{k\text{-sup}} = \min_{I: |I| \leq k} \left(\|x_I\|_2 + \|x_{I^c}\|_1\right)$$

### Step 1: Each function in the minimization is convex

For any fixed subset $I$ with $|I| \leq k$, the function:
$$f_I(x) = \|x_I\|_2 + \|x_{I^c}\|_1$$

is convex because:
- $\|x_I\|_2$ is a norm (convex)
- $\|x_{I^c}\|_1$ is a norm (convex)
- The sum of convex functions is convex

### Step 2: Pointwise minimum preserves convexity

The k-support norm is:
$$\|x\|_{k\text{-sup}} = \min_{I: |I| \leq k} f_I(x)$$

**Key fact**: The pointwise minimum (infimum) of a family of convex functions is convex.

**Proof of this fact**: Let $x, y \in \mathbb{R}^n$ and $\theta \in [0,1]$. Then:

$$\|θx + (1-θ)y\|_{k\text{-sup}} = \min_{I: |I| \leq k} f_I(θx + (1-θ)y)$$

Since each $f_I$ is convex:
$$f_I(θx + (1-θ)y) \leq θf_I(x) + (1-θ)f_I(y)$$

Taking minimum over all $I$ on the right:
$$\min_I f_I(θx + (1-θ)y) \leq \min_I [θf_I(x) + (1-θ)f_I(y)]$$

Note that for any $I$:
$$θf_I(x) + (1-θ)f_I(y) \geq θ\min_J f_J(x) + (1-θ)\min_J f_J(y)$$

Therefore:
$$\|θx + (1-θ)y\|_{k\text{-sup}} \leq θ\|x\|_{k\text{-sup}} + (1-θ)\|y\|_{k\text{-sup}}$$

This proves convexity. ∎

## Connection to Top-K Thresholding

The proximal operator of the k-support norm is:

$$\text{prox}_{\lambda \|\cdot\|_{k\text{-sup}}}(x) = \arg\min_z \frac{1}{2}\|x - z\|_2^2 + \lambda \|z\|_{k\text{-sup}}$$

**Result**: This proximal operator performs **top-k hard thresholding** with an appropriate threshold value that depends on $\lambda$ and the values in $x$.

For appropriate choice of $\lambda$, this exactly selects the top-k elements by magnitude and zeros out the rest.

## Why Is K-Support Norm Rarely Used?

Despite being a convex sparsity-promoting norm, the k-support norm is less popular than $\ell_1$ for several reasons:

### 1. **Computational Complexity**
- Computing the k-support norm requires sorting (finding top-k elements): $O(n \log n)$ or $O(n \log k)$
- The $\ell_1$ norm is just a sum: $O(n)$
- For large-scale problems, this difference matters

> **Note**: While $O(n \log n)$ is technically "almost linear" and in practice performs quite well, the constant factors matter in iterative optimization algorithms. When you need to compute the norm (and its proximal operator) thousands or millions of times during gradient descent, even small multiplicative factors add up. Moreover, modern hardware vectorization makes simple operations like summing all elements extremely fast, while sorting is harder to parallelize efficiently. That said, you're right that this complexity difference is often overstated as a barrier—with modern quickselect or median-of-medians algorithms, finding top-k can even be done in $O(n)$ expected time!

### 2. **Less Accessible Optimization**
- $\ell_1$ has simple proximal operators and subgradients
- $\ell_1$ fits naturally into LASSO and other well-studied frameworks
- K-support norm requires more specialized algorithms

### 3. **Hyperparameter Selection**
- With $\ell_1$, you tune one parameter ($\lambda$) and get varying sparsity levels
- With k-support norm, you must pre-specify $k$ (the exact sparsity level)
- Cross-validation for discrete $k$ is less smooth than for continuous $\lambda$

### 4. **Theoretical Development**
- $\ell_1$ has decades of theory (compressed sensing, RIP conditions, etc.)
- K-support norm is newer (introduced by Argyriou et al., 2012)
- Less established theory for recovery guarantees

### 5. **Software Availability**
- Every optimization library has $\ell_1$ penalties built-in
- K-support norm requires custom implementation

### 6. **The Sparsity Dilemma**
- If you know $k$ beforehand, you can just do best subset selection
- If you don't know $k$, then tuning the k-support norm isn't much better than tuning $\ell_1$

## When K-Support Norm Is Useful

Despite these limitations, the k-support norm excels in:

1. **Fixed sparsity constraints**: When you know exactly how many features you want
2. **Group sparsity**: Extensions to structured sparsity (overlapping groups)
3. **Theoretical analysis**: Provides convex relaxation with exact sparsity control
4. **Portfolio optimization**: Selecting exactly k assets
5. **Feature selection with budget**: Hard constraint on number of selected features

## Special Cases

- **k = 1**: $\|x\|_{1\text{-sup}} = \min_i (|x_i| + \|\{x_j : j \neq i\}\|_2)$ — not a standard norm
- **k = n**: Reduces to $\|x\|_1$ ($\ell_1$ norm)
- The norm interpolates between sparse ($\ell_1$-like) and dense ($\ell_2$-like) penalization as $k$ varies

## References

- Argyriou, A., Foygel, R., & Srebro, N. (2012). "Sparse prediction with the k-support norm." *NIPS*.
- McDonald, A. M., Pontil, M., & Stamos, D. (2016). "New perspectives on k-support and cluster norms." *JMLR*.