@def title = "K-Support Norm: A Complete Guide"
@def published = "31 October 2025"
@def tags = ["convex-optimization"]

# K-Support Norm: A Complete Guide

Based on "Sparse Prediction with the k-Support Norm" by Argyriou, Foygel, and Srebro (2012)

---

## The Big Picture: Why Do We Need This?

**The Problem:** When doing regression with sparse data, we want to find predictors that:
1. Have few non-zero coefficients (sparsity)
2. Don't have huge coefficient values (bounded $\ell_2$ norm)

**Traditional approaches:**
- **Lasso** ($\ell_1$ regularization): Encourages sparsity but can be too aggressive
- **Elastic Net**: Combines $\ell_1$ and $\ell_2$ penalties, but is this the best we can do?

**The k-support norm answer:** It turns out the elastic net is NOT the tightest convex relaxation! The k-support norm is.

---

## What Are We Trying to Relax?

We want sparse vectors with bounded $\ell_2$ norm. Mathematically, we care about the set:

$$S_k^{(2)} = \{w \in \mathbb{R}^d : \|w\|_0 \leq k, \|w\|_2 \leq 1\}$$

**Translation:** Vectors with at most $k$ non-zero entries and $\ell_2$ norm at most 1.

**The problem:** This set isn't convex, making optimization hard (NP-hard, actually).

**The solution:** Take the convex hull (smallest convex set containing it):

$$C_k = \text{conv}(S_k^{(2)}) = \text{conv}\{w : \|w\|_0 \leq k, \|w\|_2 \leq 1\}$$

The k-support norm is the norm whose unit ball equals $C_k$.

---

## Definition: Two Equivalent Views

### View 1: Gauge Function (Abstract)

The k-support norm $\|\cdot\|_k^{\text{sp}}$ is the gauge function of $C_k$:

$\|w\|_k^{\text{sp}} = \inf\{\lambda \geq 0 : w \in \lambda C_k\}$

**Intuition:** How much do we need to scale the unit ball $C_k$ to contain $w$?

> **What is a gauge function?**
> 
> A gauge function (also called a Minkowski functional) is a way to create a norm from any convex set $C$ that contains the origin. Given a convex set $C$, its gauge function $\gamma_C(x)$ answers: *"What's the smallest factor I need to scale $C$ by so that $x$ fits inside?"*
> 
> Formally: $\gamma_C(x) = \inf\{\lambda \geq 0 : x \in \lambda C\}$
> 
> **Example:** If $C$ is the unit ball of the $\ell_2$ norm (a sphere), then $\gamma_C(x) = \|x\|_2$.
> 
> **Key properties:**
> - $\gamma_C(\alpha x) = \alpha \gamma_C(x)$ for $\alpha \geq 0$ (positive homogeneity)
> - $\gamma_C(x + y) \leq \gamma_C(x) + \gamma_C(y)$ (triangle inequality)
> - If $C$ is symmetric around the origin, $\gamma_C$ is a norm!
> 
> **Why it matters here:** The k-support norm is defined as the gauge function of the convex hull $C_k$. This automatically makes it a norm with $C_k$ as its unit ball.

---

### View 2: Variational Formula (Computational)

$\|w\|_k^{\text{sp}} = \min \left\{\sum_{I \in \mathcal{G}_k} \|v_I\|_2 : \text{supp}(v_I) \subseteq I, \sum_{I \in \mathcal{G}_k} v_I = w\right\}$

where $\mathcal{G}_k$ is the set of all subsets of $\{1, \ldots, d\}$ with cardinality at most $k$.

**Translation:** Decompose $w$ into pieces, each supported on at most $k$ coordinates, and minimize the sum of their $\ell_2$ norms.

> **Why "variational"?**
> 
> The term "variational" comes from the calculus of variations, where we optimize over *functions* or *decompositions* rather than just numbers.
> 
> Here, we're not just computing a value - we're minimizing over all possible ways to decompose $w$ into sparse pieces:
> - The "variables" are the decomposition vectors $v_I$ (infinitely many choices!)
> - We search for the decomposition that minimizes the objective
> - Different decompositions give different values, and we want the minimum
> 
> **Contrast with direct formula:** The gauge definition directly evaluates $\|w\|$, while the variational formula says "find the best decomposition, then compute from that."
> 
> This is common in convex analysis - many norms have both a direct definition and a variational representation that's useful for different purposes (theory vs. computation vs. intuition).

**Key insight:** This is the **group lasso with overlaps** for ALL possible groups of size $k$!

---

## The Closed-Form Formula

Despite having exponentially many groups, we can compute the k-support norm in $O(d \log d)$ time!

### The Formula

For $w \in \mathbb{R}^d$, let $|w|^\downarrow$ denote $w$ with entries sorted by absolute value in descending order. Then:

$$\|w\|_k^{\text{sp}} = \sqrt{\sum_{i=1}^{k-r-1} (|w|_i^\downarrow)^2 + \frac{1}{r+1}\left(\sum_{i=k-r}^d |w|_i^\downarrow\right)^2}$$

where $r \in \{0, \ldots, k-1\}$ is the unique integer satisfying:

$$|w|_{k-r-1}^\downarrow > \frac{1}{r+1}\sum_{i=k-r}^d |w|_i^\downarrow \geq |w|_{k-r}^\downarrow$$

(with the convention that $|w|_0^\downarrow = +\infty$)

---

## What Does This Mean?

The k-support norm **partitions the sorted vector** into two parts:

1. **Head** (largest $k-r-1$ entries): These get full $\ell_2$ treatment
2. **Tail** (remaining entries): These get averaged together

**The intuition:** 
- Large entries aren't penalized much (like $\ell_2$)
- Small entries get shrunk together (like $\ell_1$)
- It's a smooth interpolation that favors keeping the $k$ most important features

---

## Step-by-Step Algorithm

### To compute $\|w\|_k^{\text{sp}}$:

**Step 1:** Sort the absolute values in descending order:
$$|w|_1^\downarrow \geq |w|_2^\downarrow \geq \cdots \geq |w|_d^\downarrow$$

**Step 2:** Find the partition point $r$ by checking the condition:
$$|w|_{k-r-1}^\downarrow > \frac{\sum_{i=k-r}^d |w|_i^\downarrow}{r+1} \geq |w|_{k-r}^\downarrow$$

Start with $r=0$ and increase until this holds.

**Step 3:** Plug into the formula:
$$\|w\|_k^{\text{sp}} = \sqrt{\sum_{i=1}^{k-r-1} (|w|_i^\downarrow)^2 + \frac{1}{r+1}\left(\sum_{i=k-r}^d |w|_i^\downarrow\right)^2}$$

---

## Worked Example

Let $w = [3, 1, -4, 2, -5]$ and compute $\|w\|_3^{\text{sp}}$.

### Step 1: Sort absolute values
$$|w|^\downarrow = [5, 4, 3, 2, 1]$$

### Step 2: Find $r$ (try $r=0,1,2$)

Try $r=0$:
- Need: $|w|_2^\downarrow = 4 > \frac{5+4+3+2+1}{1} = 15$ ✗

Try $r=1$:
- Need: $|w|_1^\downarrow = 5 > \frac{4+3+2+1}{2} = 5$ ✗

Try $r=2$:
- Need: $5 > \frac{3+2+1}{3} = 2$ ✓
- And: $2 \geq |w|_1^\downarrow = 5$ ✗ (Wait, this doesn't work either)

Actually, let me recalculate with $k=3$:

Try $r=0$:
- Need: $|w|_3^\downarrow = 3 > \frac{3+2+1}{1} = 6$ ✗

Try $r=1$:
- Need: $|w|_2^\downarrow = 4 > \frac{3+2+1}{2} = 3$ ✓
- And: $3 \geq |w|_3^\downarrow = 3$ ✓

So $r=1$.

### Step 3: Compute
$$\|w\|_3^{\text{sp}} = \sqrt{5^2 + \frac{1}{2}(4+3+2+1)^2} = \sqrt{25 + \frac{100}{4}} = \sqrt{25 + 25} = \sqrt{50} \approx 7.07$$

---

## The Dual Norm

The dual of the k-support norm has an even simpler form!

$$\|u\|_{k}^{\text{sp}*} = \|u\|_{(k)}^{(2)} = \sqrt{\sum_{i=1}^k (|u|_i^\downarrow)^2}$$

**Translation:** Just take the $\ell_2$ norm of the $k$ largest entries!

**Special cases:**
- When $k=1$: $\|u\|_1^{\text{sp}*} = \|u\|_\infty$ (just the largest entry)
- When $k=d$: $\|u\|_d^{\text{sp}*} = \|u\|_2$ (full $\ell_2$ norm)

This dual norm is called the **2-k symmetric gauge norm**.

---

## How Does It Compare to Elastic Net?

The elastic net uses the norm:
$$\|w\|_k^{\text{el}} = \max\left\{\|w\|_2, \frac{\|w\|_1}{\sqrt{k}}\right\}$$

### The Key Results

**Theorem:** 
$$\|w\|_k^{\text{el}} \leq \|w\|_k^{\text{sp}} < \sqrt{2} \|w\|_k^{\text{el}}$$

**What this means:**
1. The k-support norm is **always tighter** than elastic net
2. But the gap is **at most a factor of $\sqrt{2}$**
3. This translates to at most a **factor of 2 in sample complexity**

So the k-support norm is provably better, but elastic net is still pretty good!

---

## Geometric Intuition

Looking at the unit balls in $\mathbb{R}^3$ for $k=2$:

**Elastic Net ball:** Has sharp corners at sparse points, flat faces between them

**k-Support ball:** More "rounded" - smooth except at the sparsest points

**Key difference:** The k-support norm is less biased toward sparsity. It's differentiable at points with cardinality less than $k$, while elastic net isn't.

This means: k-support is **better for prediction** when correlated features should all be included, even if elastic net is **better for inducing sparsity**.

---

## Using It for Learning

### Regression with k-support norm:

$$\min_{w \in \mathbb{R}^d} \left\{\frac{1}{2}\|Xw - y\|^2 + \frac{\lambda}{2}(\|w\|_k^{\text{sp}})^2\right\}$$

**Two hyperparameters to tune (via cross-validation):**
1. $\lambda > 0$: regularization strength
2. $k \in \{1, \ldots, d\}$: sparsity level

**Important:** $k$ doesn't directly control the number of non-zeros in the solution! It controls how much we relax the sparsity constraint.

---

## Optimization: Proximal Methods

We can solve the learning problem efficiently using accelerated proximal gradient methods (like FISTA).

**Key requirement:** Computing the proximal operator:

$$\text{prox}_{\frac{1}{2L}(\|\cdot\|_k^{\text{sp}})^2}(v) = \arg\min_u \left\{\frac{1}{2}\|u-v\|^2 + \frac{1}{2L}(\|u\|_k^{\text{sp}})^2\right\}$$

**Good news:** This can be computed in $O(d(k + \log d))$ time using a specialized algorithm!

**Result:** After $T$ iterations of FISTA, we get $O(1/T^2)$ convergence - optimal for first-order methods.

---

## When Should You Use k-Support vs Elastic Net?

### Use k-Support Norm when:
- You want the **tightest possible convex relaxation**
- **Prediction accuracy** is your primary goal
- Features are **correlated** and you want to include groups of them
- You can afford slightly more computation

### Use Elastic Net when:
- You want **more sparsity** in your solution
- Computation speed is critical
- Interpretability requires fewer features
- The $\sqrt{2}$ factor doesn't matter for your application

---

## Special Cases

### When $k=1$:
$$\|w\|_1^{\text{sp}} = \|w\|_1$$ 
(recovers Lasso)

### When $k=d$:
$$\|w\|_d^{\text{sp}} = \sqrt{d} \cdot \|w\|_2$$ 
(just scaled Ridge regression)

### Intermediate $k$:
Smooth interpolation between these extremes!

---

## Empirical Results from the Paper

The authors tested on three datasets:

1. **Synthetic data** (correlated features): k-support reduced MSE by 14.7% vs elastic net
2. **Heart disease data**: All methods performed identically
3. **20 Newsgroups** (text classification): k-support achieved 73.40% accuracy vs 72.53% for elastic net

**Takeaway:** Gains are modest but real, especially when features are correlated.

---

## Summary: The Main Ideas

1. **k-support norm = tightest convex relaxation** of sparsity + $\ell_2$ constraint

2. **Better than elastic net** by up to $\sqrt{2}$ factor (at most 2× in sample complexity)

3. **Computable in $O(d \log d)$** despite exponentially many groups

4. **Encourages grouped selection** of correlated features (good for prediction)

5. **Less sparse than elastic net** but often more accurate

6. **Dual norm is simple:** just $\ell_2$ norm of top $k$ entries

The k-support norm beautifully balances the competing goals of sparsity, bounded magnitude, and predictive accuracy!