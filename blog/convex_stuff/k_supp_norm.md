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

$\text{prox}_{\lambda \|\cdot\|_{k\text{-sup}}}(x) = \arg\min_z \frac{1}{2}\|x - z\|_2^2 + \lambda \|z\|_{k\text{-sup}}$

**Theorem**: For sufficiently large $\lambda$, this proximal operator performs **top-k hard thresholding**: it keeps the $k$ largest elements (in absolute value) from $x$ and sets all others to zero.

### Proof Sketch

**Step 1: Optimality Conditions**

Let $z^*$ be the optimal solution. The subdifferential optimality condition is:
$0 \in z^* - x + \lambda \partial \|z^*\|_{k\text{-sup}}$

where $\partial$ denotes the subdifferential.

**Step 2: Structure of the Subdifferential**

Recall that:
$\|z\|_{k\text{-sup}} = \min_{|I| \leq k} \left(\|z_I\|_2 + \|z_{I^c}\|_1\right)$

Let $I^*$ be the optimal index set achieving this minimum for $z^*$. The subdifferential has the form:

$\partial \|z^*\|_{k\text{-sup}} \ni \begin{cases}
\frac{z_i^*}{\|z_{I^*}\|_2} & \text{if } i \in I^* \text{ and } \|z_{I^*}\|_2 > 0\\
\text{sign}(z_i^*) & \text{if } i \in I^{*c} \text{ and } z_i^* \neq 0\\
[-1, 1] & \text{if } z_i^* = 0
\end{cases}$

**Step 3: Top-k Structure Emerges**

From the optimality condition: $z^* = x - \lambda g$ where $g \in \partial \|z^*\|_{k\text{-sup}}$.

Consider what happens if $z^*$ has the top-k structure: exactly $k$ non-zero entries corresponding to the largest $|x_i|$ values.

- For indices $i$ in the support with $z_i^* \neq 0$: 
  $z_i^* = x_i - \lambda g_i$
  where $g_i$ depends on whether $i$ is in the $\ell_2$ part or $\ell_1$ part.

- For indices $i$ outside the support with $z_i^* = 0$:
  $0 = x_i - \lambda g_i \implies x_i = \lambda g_i \text{ where } g_i \in [-1, 1]$
  This is satisfied when $|x_i| \leq \lambda$.

**Step 4: Large $\lambda$ Forces Top-k**

When $\lambda$ is sufficiently large:
1. The optimal support $I^*$ will have exactly $k$ elements (the penalty strongly favors sparsity)
2. These $k$ elements must be those with largest $|x_i|$ to minimize $\|x - z\|_2^2$
3. All other elements are set to zero because the penalty overwhelms keeping small values

**More precisely**: If we sort $|x|$ as $|x|_{(1)} \geq |x|_{(2)} \geq \cdots \geq |x|_{(n)}$, then for:
$\lambda > \frac{|x|_{(k)} - |x|_{(k+1)}}{2}$

the solution will select exactly the top-k elements.

### Intuition

The k-support norm penalizes vectors that spread their "mass" across many coordinates. The optimal trade-off between fitting $x$ (via $\|x-z\|_2^2$) and having small k-support norm is to:
- Keep the $k$ largest values from $x$ (to minimize reconstruction error)
- Zero out everything else (to minimize the norm penalty)

This is exactly what top-k hard thresholding does!

### Algorithmic Note

In practice, computing the proximal operator involves:
1. Sort $|x|$ to find the top-k elements
2. Apply soft thresholding or shrinkage to these elements
3. Zero out the remaining $n-k$ elements

The exact threshold depends on $\lambda$ and the specific values, but the **support** (which elements are non-zero) is determined purely by the top-k selection.

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

## Exotic Properties and Extensions

### 1. **Relationship to Other Norms**

The k-support norm can be expressed as:
$\|x\|_{k\text{-sup}} = \max_{\substack{S_1, \ldots, S_m \\ \text{partition of } [n]}} \sum_{j=1}^m \sqrt{|S_j|} \|x_{S_j}\|_2$
subject to each $|S_j| \leq k$.

This reveals it as a **cluster norm** that groups coordinates optimally.

### 2. **Dual Norm**

The dual norm (k-support)* has a beautiful form:
$\|y\|_{k\text{-sup}}^* = \max_{|I| = k} \left(\|y_I\|_\infty + \frac{1}{\sqrt{k}}\|y_{I^c}\|_2\right)$

This is useful for deriving optimality conditions and understanding the geometry of the unit ball.

### 3. **Variational Characterization**

The k-support norm admits a variational form:
$\|x\|_{k\text{-sup}}^2 = \min_{\substack{u, v \\ u + v = x}} \frac{1}{k}\|u\|_1^2 + \|v\|_2^2$

This decomposition into $\ell_1$ and $\ell_2$ components provides insight into how it balances sparsity and density.

### 4. **Non-Smooth Optimization Landscape**

Unlike $\ell_1$, the k-support norm is **not smooth** and its unit ball has **flat faces** at exactly the points with sparsity level $k$. This creates interesting optimization challenges:
- The gradient is undefined at points where the support changes
- Proximal gradient methods need careful implementation
- The optimal support can "jump" discontinuously as parameters change

### 5. **Connection to OMP and Matching Pursuit**

The k-support norm provides a **convex relaxation** of Orthogonal Matching Pursuit (OMP). While OMP greedily selects k atoms, minimizing the k-support norm finds the optimal k-sparse representation via convex optimization.

### 6. **Group Sparsity Extension**

The norm extends naturally to **overlapping group structures**:
$\|x\|_{k,G} = \min_{\substack{G' \subseteq G \\ |G'| \leq k}} \left(\sum_{g \in G'} \|x_g\|_2 + \sum_{g \notin G'} \|x_g\|_1\right)$

This allows selecting $k$ groups while keeping within-group structure.

### 7. **Statistical Properties**

For sparse linear regression with k-sparse true signal:
- K-support norm achieves **minimax optimal** estimation rates
- It has better sample complexity than $\ell_1$ when the sparsity level is known
- Recovery guarantees hold under weaker restricted isometry properties (RIP) than $\ell_1$

### 8. **Non-Isotropy**

Unlike $\ell_2$ (rotationally invariant) or $\ell_1$ (coordinate-aligned), the k-support norm ball has a **hybrid geometry**:
- It's "round" in the k-dimensional subspace of largest coefficients
- It's "diamond-shaped" (like $\ell_1$) in the orthogonal complement
- This geometry adapts to the signal structure

### 9. **Atomic Norm Perspective**

The k-support norm is an **atomic norm** where the atoms are:
$\mathcal{A}_k = \left\{x : \|x\|_2 = 1, \|x\|_0 \leq k\right\}$

This connects it to the broader theory of atomic norms and structured sparsity.

### 10. **Computational Trick: QuickSelect**

Computing the k-support norm can be done in **expected linear time** $O(n)$ using QuickSelect instead of full sorting:
1. Use QuickSelect to partition around the k-th largest element
2. Compute $\ell_1$ norm of top-k and $\ell_2$ norm of the rest
3. No need to fully sort!

### 11. **Failure Mode: Unknown Sparsity**

The k-support norm has an interesting failure mode: if the true sparsity is $k' < k$, using $\|x\|_{k\text{-sup}}$ will **not** recover the sparser solution. It commits to using exactly $k$ coordinates. This is unlike $\ell_1$, which gracefully adapts to the actual sparsity level.

### 12. **Matrix Extension: (k,k)-Support Norm**

For matrices, there's a natural extension:
$\|X\|_{(k,k)} = \min_{\substack{|I| \leq k \\ |J| \leq k}} \left(\|X_{I,J}\|_F + \|X_{I,J^c}\|_{2,1} + \|X_{I^c,J}\|_{1,2} + \|X_{I^c,J^c}\|_{1,1}\right)$

This promotes bi-sparse structure in rows and columns simultaneously, useful for bi-clustering.