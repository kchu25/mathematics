@def title = "Deep Dive: Mathematical Machinery in \"Sparse Prediction with the k-Support Norm\""
@def published = "31 October 2025"
@def tags = ["convex-optimization"]


# Deep Dive: Mathematical Machinery in "Sparse Prediction with the k-Support Norm"

## The Authors' Core Intuition

### The Problem with Standard Relaxations

The authors start with a critical observation: **scale matters in convex relaxations**. 

**Naive claim**: "$\|\cdot\|_1$ is the convex envelope of $\|\cdot\|_0$"

**Reality**: This is scale-dependent! We have:
$$\|w\|_1 \leq \|w\|_\infty \cdot \|w\|_0$$

So $\ell_1$ only lower-bounds $\ell_0$ when entries are bounded. This leads to different relaxations:

1. **With $\ell_\infty$ constraint**: $S_k^{(\infty)} = \{w : \|w\|_0 \leq k, \|w\|_\infty \leq 1\}$
   - Relaxation: $\{w : \|w\|_1 \leq k\}$
   - Sample complexity: $O(k^2 \log d)$ (quadratic in $k$!)

2. **With $\ell_2$ constraint**: $S_k^{(2)} = \{w : \|w\|_0 \leq k, \|w\|_2 \leq 1\}$
   - Relaxation: $\{w : \|w\|_1 \leq \sqrt{k}\}$
   - Sample complexity: $O(k \log d)$ (linear in $k$!)

**Key insight**: The $\ell_2$ constraint is more natural because:
- In regression, $\mathbb{E}[(x^\top w)^2]$ being bounded is reasonable
- It provides better sample complexity
- But we can do even better!

---

## Part I: The Convex Hull Construction

### The Geometric Picture

The authors ask: *What is the exact convex hull of $S_k^{(2)}$?*

$$C_k = \text{conv}\{w : \|w\|_0 \leq k, \|w\|_2 \leq 1\}$$

**Intuition**: Every point in $C_k$ can be written as:
$$w = \sum_{i=1}^m \lambda_i z_i, \quad \sum_{i=1}^m \lambda_i = 1, \quad \lambda_i \geq 0, \quad z_i \in S_k^{(2)}$$

This is a **convex combination** of k-sparse unit vectors.

### The Variational Characterization

The authors prove $C_k$ is the unit ball of:
$$\|w\|_k^{\text{sp}} = \min\left\{\sum_{I \in \mathcal{G}_k} \|v_I\|_2 : \text{supp}(v_I) \subseteq I, \sum_{I \in \mathcal{G}_k} v_I = w\right\}$$

**Why this works**: 
- Decompose $w$ into pieces $v_I$, each supported on at most $k$ coordinates
- Minimize the sum of $\ell_2$ norms of these pieces
- This is the **group lasso with overlapping groups** where groups = all k-subsets

**The puzzle**: There are $\binom{d}{k}$ groups! How to compute this?

---

## Part II: The Dual Norm - Hardy-Littlewood-Pólya Machinery

### The Duality Approach

The dual norm is:
$$\|u\|_k^{\text{sp}*} = \sup\{\langle w, u\rangle : \|w\|_k^{\text{sp}} \leq 1\}$$

**Authors' strategy**: Use the geometric definition.
$$\|u\|_k^{\text{sp}*} = \sup\{\langle w, u\rangle : w \in C_k\}$$

Since $C_k$ is convex, the supremum is attained at an **extreme point** of $C_k$. The extreme points are exactly the k-sparse unit vectors!

$$\|u\|_k^{\text{sp}*} = \max\left\{\sqrt{\sum_{i \in I} u_i^2} : I \subseteq \{1,\ldots,d\}, |I| \leq k\right\}$$

**Key inequality** (Hardy-Littlewood-Pólya):
$$\langle w, u \rangle \leq \langle w^\downarrow, u^\downarrow \rangle$$

This says: the inner product is maximized when both vectors are sorted in the same order.

**Application**: 
$$\max_{|I|=k} \sqrt{\sum_{i \in I} u_i^2} = \sqrt{\sum_{i=1}^k (|u|_i^\downarrow)^2}$$

**Result**: The dual is the **$\ell_2$ norm of the top-k entries**!

$$\|u\|_k^{\text{sp}*} = \|u\|_{(2)}^{(k)} := \left(\sum_{i=1}^k (|u|_i^\downarrow)^2\right)^{1/2}$$

**Beautiful structure**: 
- Primal interpolates between $\ell_1$ (k=1) and $\ell_2$ (k=d)
- Dual interpolates between $\ell_\infty$ (k=1) and $\ell_2$ (k=d)

---

## Part III: Computing the Primal Norm - The Water-Filling Algorithm

### The Fenchel-Young Machinery

For any norm and its dual:
$$\|w\|_k^{\text{sp}} = \max_u \{\langle w, u \rangle - \tfrac{1}{2}\|u\|_{(2)}^{(k)2}\} + \tfrac{1}{2}\|w\|_k^{\text{sp}2}$$

**Authors' trick**: Use the **dual characterization** to compute the primal!

$$\tfrac{1}{2}\|w\|_k^{\text{sp}2} = \max_u \left\{\langle w, u \rangle - \tfrac{1}{2}\|u\|_{(2)}^{(k)2}\right\}$$

### The Lagrangian Analysis

Rewrite using $\alpha_i = |u_i^\downarrow|$ with $\alpha_1 \geq \cdots \geq \alpha_d \geq 0$:

$$\tfrac{1}{2}\|w\|_k^{\text{sp}2} = \max_{\alpha_1 \geq \cdots \geq \alpha_d \geq 0} \left\{\sum_{i=1}^d \alpha_i |w|_i^\downarrow - \tfrac{1}{2}\sum_{i=1}^k \alpha_i^2\right\}$$

**Key observation**: Only the top $k$ dual variables $\alpha_1, \ldots, \alpha_k$ appear in the penalty term!

### The Water-Filling Intuition

Think of $\alpha_i$ as "water levels" at positions $i = 1, \ldots, d$:
- Water poured from position 1 to position $d$
- Must satisfy $\alpha_1 \geq \alpha_2 \geq \cdots \geq \alpha_d \geq 0$
- Positions 1 to $k$ have "cost" $\frac{1}{2}\alpha_i^2$ (quadratic penalty)
- Positions $k+1$ to $d$ are "free" (no penalty)

**Optimization principle**: 
- For $i \leq k-1$: set $\alpha_i = |w|_i^\downarrow$ (benefit $|w|_i^\downarrow$ outweighs cost $\frac{1}{2}\alpha_i^2$ since these are large)
- At some position $k-r$ to $\ell$: the $\alpha$ values "plateau" at a constant level
- For $i > \ell$: set $\alpha_i = 0$ (too costly)

### The Plateau Condition

The optimal $\alpha$ satisfies:
$$\alpha_i = \begin{cases}
|w|_i^\downarrow & i = 1, \ldots, k-r-1 \text{ (steep part)}\\
\frac{1}{r+1}\sum_{j=k-r}^\ell |w|_j^\downarrow & i = k-r, \ldots, \ell \text{ (plateau)}\\
0 & i = \ell+1, \ldots, d \text{ (flat)}
\end{cases}$$

**Boundary conditions**: The plateau level $\bar{\alpha} = \frac{1}{r+1}\sum_{j=k-r}^\ell |w|_j^\downarrow$ must satisfy:
$$|w|_{k-r-1}^\downarrow > \bar{\alpha} \geq |w|_{k-r}^\downarrow$$
$$|w|_\ell^\downarrow > \bar{\alpha} \geq |w|_{\ell+1}^\downarrow$$

**Computational algorithm**: 
1. Sort $|w|$ in decreasing order: $O(d \log d)$
2. Search for $r, \ell$ satisfying boundary conditions: $O(k)$ or $O(d)$
3. Compute the norm using the closed form

**Final formula**:
$$\|w\|_k^{\text{sp}} = \sqrt{\sum_{i=1}^{k-r-1} (|w|_i^\downarrow)^2 + \frac{1}{r+1}\left(\sum_{i=k-r}^\ell |w|_i^\downarrow\right)^2}$$

---

## Part IV: Comparison with Elastic Net - The $\sqrt{2}$ Factor

### The Elastic Net as a Max-Norm

The elastic net with constraints $\|w\|_1 \leq \sqrt{k}, \|w\|_2 \leq 1$ has unit ball:
$$B_k^{\text{el}} = \{w : \|w\|_1 \leq \sqrt{k}\} \cap \{w : \|w\|_2 \leq 1\}$$

The corresponding norm is:
$$\|w\|_k^{\text{el}} = \max\left\{\|w\|_2, \frac{\|w\|_1}{\sqrt{k}}\right\}$$

**Geometric intuition**: The unit ball is the **intersection** of two balls (not a convex hull). This is larger than $C_k$!

### The Dual of Elastic Net

For max-norms, the dual is an **infimal convolution**:
$$\|u\|_k^{\text{el}*} = \inf_{a \in \mathbb{R}^d} \{\|a\|_2 + \sqrt{k}\|u - a\|_\infty\}$$

**Interpretation**: Split $u = a + (u - a)$ optimally between $\ell_2$ and $\ell_\infty$ parts.

### Lower Bound: $\|\cdot\|_k^{\text{el}} \leq \|\cdot\|_k^{\text{sp}}$

**Strategy**: Show $C_k \subseteq B_k^{\text{el}}$.

For any k-sparse unit vector $w$:
- $\|w\|_2 = 1$ ✓
- $\|w\|_1 \leq \sqrt{k} \cdot \|w\|_2 = \sqrt{k}$ (by Cauchy-Schwarz) ✓

So $S_k^{(2)} \subseteq B_k^{\text{el}}$, hence $C_k = \text{conv}(S_k^{(2)}) \subseteq B_k^{\text{el}}$.

### Upper Bound: $\|\cdot\|_k^{\text{sp}} < \sqrt{2} \|\cdot\|_k^{\text{el}}$

**Strategy**: Work in the dual space!

Need to show: $\|u\|_{(2)}^{(k)} < \sqrt{2} \cdot \|u\|_k^{\text{el}*}$

**Construction**: Choose $a = (u_1 - u_{k+1}, \ldots, u_k - u_{k+1}, 0, \ldots, 0)$.

Then:
$$\|a\|_2 = \sqrt{\sum_{i=1}^k (u_i - u_{k+1})^2}$$
$$\|u - a\|_\infty = |u_{k+1}|$$

So:
$$\|u\|_k^{\text{el}*} \leq \sqrt{\sum_{i=1}^k (u_i - u_{k+1})^2} + \sqrt{k} |u_{k+1}|$$

**Algebraic manipulation**:
\begin{align}
\text{RHS}^2 &\leq \left(\sqrt{\sum_{i=1}^k (u_i - u_{k+1})^2} + \sqrt{k} |u_{k+1}|\right)^2\\
&= \sum_{i=1}^k (u_i - u_{k+1})^2 + k u_{k+1}^2 + 2\sqrt{k \sum_{i=1}^k (u_i - u_{k+1})^2} \cdot |u_{k+1}|\\
&\leq \sum_{i=1}^k u_i^2 - k u_{k+1}^2 + k u_{k+1}^2 + 2\sqrt{k \sum_{i=1}^k u_i^2} \cdot |u_{k+1}|\\
&= \sum_{i=1}^k u_i^2 + 2\sqrt{k u_{k+1}^2 \sum_{i=1}^k u_i^2}
\end{align}

**Key inequality** (AM-GM or Cauchy-Schwarz):
$$2\sqrt{k u_{k+1}^2 \sum_{i=1}^k u_i^2} \leq k u_{k+1}^2 + \sum_{i=1}^k u_i^2$$

Therefore:
$$\|u\|_k^{\text{el}*2} \leq 2\sum_{i=1}^k u_i^2 = 2\|u\|_{(2)}^{(k)2}$$

**Result**: $\|w\|_k^{\text{sp}} < \sqrt{2} \|w\|_k^{\text{el}}$ with strict inequality (unless special cases).

---

## Part V: Proximal Operator - The Soft-Thresholding Generalization

### The Proximity Problem

For optimization, we need:
$$\text{prox}_{\frac{\lambda}{2L}\|\cdot\|_k^{\text{sp}2}}(v) = \arg\min_w \left\{\frac{1}{2}\|w - v\|^2 + \frac{\lambda}{2L}\|w\|_k^{\text{sp}2}\right\}$$

### The Fixed-Point Characterization

**Optimality condition**: $q = \text{prox}(v)$ if and only if:
$$Lv - Lq \in \partial\left(\tfrac{1}{2}\|q\|_k^{\text{sp}2}\right)$$

**Authors' insight**: Use the variational formula for $\|\cdot\|_k^{\text{sp}}$!

The subdifferential is:
$$\partial\left(\tfrac{1}{2}\|q\|_k^{\text{sp}2}\right) = \{u : \|u\|_{(2)}^{(k)} \leq 1, \langle u, q \rangle = \|q\|_k^{\text{sp}}\}$$

### The Shrinkage Structure

**Key observation**: By symmetry, $q$ has the same sign pattern and ordering as $v$.

The optimal $q$ has the form:
$$q_i = \begin{cases}
\frac{L}{L+1}v_i & i = 1, \ldots, k-r-1 \text{ (light shrinkage)}\\
v_i - \frac{T_{r,\ell}}{\ell - k + (L+1)r + L + 1} & i = k-r, \ldots, \ell \text{ (moderate shrinkage)}\\
0 & i = \ell+1, \ldots, d \text{ (hard thresholding)}
\end{cases}$$

where $T_{r,\ell} = \sum_{i=k-r}^\ell v_i^\downarrow$.

**Interpretation**: This is a **generalized soft-thresholding**:
- Large coordinates (top $k-r-1$): shrunk by factor $\frac{L}{L+1}$ (Ridge-like)
- Medium coordinates ($k-r$ to $\ell$): shifted by a constant (Lasso-like)
- Small coordinates ($> \ell$): set to zero (hard thresholding)

**Algorithm**: 
1. Find $r, \ell$ satisfying conditions (7)-(8)
2. Apply piecewise shrinkage formula
3. Restore original sign and ordering

**Complexity**: $O(d(k + \log d))$

---

## Part VI: The Sample Complexity Argument

### The Authors' Statistical Viewpoint

**Question**: Why is $\ell_2 + \text{sparsity}$ better than $\ell_\infty + \text{sparsity}$?

**Answer**: Sample complexity!

Let $\mathcal{F}_k = \{x \mapsto w^\top x : w \in S_k^{(2)}\}$ be the function class.

**Rademacher complexity** (roughly measures "richness" of function class):
$$\mathcal{R}_n(\mathcal{F}_k) \approx \sqrt{\frac{k \log d}{n}}$$

The sample complexity for learning is:
$$n \approx \frac{k \log d}{\epsilon^2}$$

to achieve excess risk $\epsilon$.

### The $k^2$ vs $k$ Dichotomy

With $\ell_\infty$ constraint and $\ell_1$ relaxation:
- True class: $\{w : \|w\|_0 \leq k, \|w\|_\infty \leq 1\}$
- Relaxed class: $\{w : \|w\|_1 \leq k\}$
- The $\ell_1$ ball has Rademacher complexity $\approx \sqrt{\frac{k^2 \log d}{n}}$ (quadratic!)

With $\ell_2$ constraint and $\ell_1$ relaxation:
- True class: $\{w : \|w\|_0 \leq k, \|w\|_2 \leq 1\}$
- Relaxed class: $\{w : \|w\|_1 \leq \sqrt{k}\}$
- The $\ell_1$ ball (rescaled) has complexity $\approx \sqrt{\frac{k \log d}{n}}$ (linear!)

**The gap**: Elastic net has complexity $\approx \sqrt{\frac{k \log d}{n}}$.

k-support norm is tighter, so same or better!

---

## The Big Picture: Why This All Works

### The Philosophical View

The authors are doing **systematic convex relaxation**:

1. **Start with intractable constraint**: $\|w\|_0 \leq k$ (combinatorial)
2. **Add scale constraint**: $\|w\|_2 \leq 1$ (makes problem well-posed)
3. **Take convex hull**: $C_k = \text{conv}(S_k^{(2)})$ (makes problem tractable)
4. **Extract the norm**: gauge function of $C_k$
5. **Compute the dual**: find structure (top-k $\ell_2$ norm)
6. **Exploit duality**: compute primal via dual
7. **Build algorithms**: use proximal methods

### The Technical Innovation

**Not just theory**: Every step is computational!
- Dual norm: $O(k)$ (partial sort)
- Primal norm: $O(d \log d)$ (full sort + search)
- Proximity operator: $O(d(k + \log d))$ (same structure)
- Full optimization: $O(T \cdot d(k + \log d))$ for $T$ iterations

### The Geometric Insight

The k-support norm has a **differentiability structure** that reflects the underlying sparsity:
- At k-sparse points with $\|w\|_2 = 1$: **non-differentiable** (vertices of $C_k$)
- At points with $\|w\|_0 < k$: **differentiable** (interior of faces)

**Contrast with $\ell_1$**: Always non-differentiable at sparse points.

**Effect on optimization**: Less "bias toward sparsity" than Lasso, but more than Ridge.

---

## Summary: The Conceptual Flow

1. **Identify the right scale**: $\ell_2$ not $\ell_\infty$ for statistical reasons
2. **Form convex hull**: geometric relaxation of discrete constraint
3. **Extract gauge**: functional representation of geometry
4. **Compute dual via extremal principle**: max over k-subsets → top-k norm
5. **Use duality for primal**: Fenchel conjugate gives water-filling
6. **Design proximal operator**: generalized soft-thresholding
7. **Prove approximation guarantees**: elastic net within $\sqrt{2}$
8. **Implement efficiently**: $O(d \log d)$ per iteration

This is **convex analysis done right**: geometry → analysis → algorithms → guarantees.