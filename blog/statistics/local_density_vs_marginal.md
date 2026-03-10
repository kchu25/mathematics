@def title = "Local Density vs. Marginal Distributions: Why Global Tests Are Blind to Clustering"
@def published = "10 March 2026"
@def tags = ["statistics"]

# Local Density vs. Marginal Distributions: Why Global Tests Are Blind to Clustering

## The Core Problem

You have two datasets:
- **Background:** A scattered collection of points
- **Subpopulation:** A tight cluster sitting *inside* the background

Your eyes see the cluster clearly. Every statistical test you throw at the data says "no difference." What is going on?

The answer: **You and the tests are measuring different things.**

Your eyes measure **local density** — whether points are close together in space.  
The tests measure **marginal distributions** — the shape and spread of the data along each axis independently.

These are fundamentally different quantities, and this post explores why, how to quantify each one, and why one can be true while the other is false.

---

## Part 1: Marginal Distributions

### Definition

A **marginal distribution** is the univariate distribution of a single variable, *ignoring all other variables*.

**Formally:** If your data is in $\mathbb{R}^d$ (d-dimensional), the marginal distribution of dimension $j$ is:

$$P(X_j) = \int_{\mathbb{R}^{d-1}} P(X_1, \ldots, X_d) \, dX_{-j}$$

In plain English: ignore all dimensions except $j$, and look at what dimension $j$ alone looks like.

### Example

Imagine 2D data:

```
Background:                    Subpopulation:
  y                             y
  │                             │
10├────●──●────────           10├─────────────
  │  ●    ●  ●                 │
 5 ├●──────────●────           5 ├──●●●●●●──
  │ ●    ●   ●                 │  ●●●●●●●●
 0 ├────────────────           0 ├──────────
  │ ●  ●  ●  ●                 │
  └─────────────── x           └─────────── x
  0  5  10  15  20             0  5  10  15  20
```

**Marginal distribution on x-axis (ignoring y):**
- Background: Points scattered across $[0, 20]$, roughly uniform
- Subpopulation: Points also spread across $[0, 20]$, but denser in $[5, 15]$

**Marginal distribution on y-axis (ignoring x):**
- Background: Points scattered across $[0, 10]$, roughly uniform
- Subpopulation: Points concentrated in $[4, 6]$, much tighter

If you only looked at the **y-marginal**, you'd see a clear difference: the subpopulation is more concentrated. But the **x-marginal** looks similar.

**What global tests do:** They summarize each marginal distribution into a single number (mean, variance, CDF, etc.) and compare those numbers.

### Quantifying Marginal Distributions

There are many ways to summarize a marginal:

| Statistic | Formula | What it captures |
|-----------|---------|------------------|
| Mean | $\mu = \frac{1}{n}\sum x_i$ | Central location |
| Variance | $\sigma^2 = \frac{1}{n}\sum (x_i - \mu)^2$ | Spread around the center |
| Skewness | $\gamma = \frac{\mathbb{E}[(X - \mu)^3]}{\sigma^3}$ | Asymmetry of the tail |
| Kurtosis | $\kappa = \frac{\mathbb{E}[(X - \mu)^4]}{\sigma^4}$ | Heavy-tailedness |
| Quantiles | $F^{-1}(p)$ | Value at percentile $p$ |
| CDF | $F(x) = P(X \leq x)$ | Full cumulative distribution |
| PDF | $f(x)$ | Density at each point |

**Example:** Suppose:
- Background y-values: $[0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10]$ (11 points, uniform)
  - Mean: $\mu_{\text{bg}} = 5$
  - Variance: $\sigma^2_{\text{bg}} = 9.17$
  
- Subpopulation y-values: $[4, 4.2, 4.5, 5, 5.5, 5.8, 6]$ (7 points, concentrated)
  - Mean: $\mu_{\text{sub}} = 5$
  - Variance: $\sigma^2_{\text{sub}} = 0.41$

**What a variance test sees:** $\sigma^2_{\text{sub}} < \sigma^2_{\text{bg}}$ (different variances). This *could* be detected.

But here's the issue: **if the subpopulation's cluster sits in the middle of the background's spread, the marginal distributions can look nearly identical**.

---

## Part 2: Local Density and Spatial Concentration

### Definition

**Local density** is how tightly packed points are *in the neighborhood of a given point*.

**Formally:** The local density at point $x$ with neighborhood radius $r$ is:

$$\rho(x, r) = \frac{\text{number of points within distance } r \text{ of } x}{V_d(r)}$$

where $V_d(r)$ is the volume of a ball of radius $r$ in $\mathbb{R}^d$.

**Intuition:** Imagine standing at point $x$ and drawing a sphere of radius $r$ around yourself. Count how many points fall inside. If that count is high, the local density is high (points are packed tightly). If the count is low, the local density is low (points are spread out).

### Example (1D)

Consider a 1D line:

```
Background:
●  ●   ●    ●   ●  ●    ●
0  5   10   15  20  25  30

Subpopulation (points within [12, 18]):
             ● ●● ●●●● ●
            12 13 14 15 16 17 18
```

**Local density within the subpopulation:** High. Points are close together.

**Local density in the background gaps:** Low. Points are far apart.

**The key insight:** The background and subpopulation have very different local densities *in their respective regions*, even though their marginal distributions (overall range and spread) might be similar.

### Quantifying Local Density

#### Option 1: k-Nearest Neighbor Distance (k-NN)

For each point $x_i$, measure the distance to its $k$-th nearest neighbor:

$$d_k(x_i) = \text{distance from } x_i \text{ to its } k\text{-th nearest neighbor}$$

**Interpretation:**
- **Small $d_k$:** Points are tightly packed (high local density)
- **Large $d_k$:** Points are spread out (low local density)

**Example:**
- Subpopulation point at $x = 15$: Its 5-NN distance is $d_5 = 0.5$ (the 5th nearest neighbor is 0.5 units away)
- Background point at $x = 5$: Its 5-NN distance is $d_5 = 4$ (the 5th nearest neighbor is 4 units away)

The subpopulation has **smaller k-NN distances**, indicating higher local density.

#### Option 2: Local Density Estimation (LDE)

Estimate the density $\hat{f}(x)$ at each point using a kernel or k-NN method:

**Kernel density estimation:**
$$\hat{f}(x) = \frac{1}{nh}\sum_{i=1}^n K\left(\frac{x - x_i}{h}\right)$$

where $K$ is a kernel (e.g., Gaussian) and $h$ is bandwidth.

**k-NN density:**
$$\hat{f}(x) = \frac{k}{n \cdot V_d(d_k(x))}$$

This directly estimates the density at each point. Points in tight clusters have high estimated density.

#### Option 3: Pairwise Distances Within Each Group

**Average pairwise distance in subpopulation:**
$$\bar{D}_{\text{sub}} = \frac{1}{m(m-1)/2} \sum_{i < j} \|x_i - x_j\|$$

**Average pairwise distance in background:**
$$\bar{D}_{\text{bg}} = \frac{1}{n(n-1)/2} \sum_{i < j} \|y_i - y_j\|$$

**Clustering coefficient:**
$$\text{CC} = \frac{\bar{D}_{\text{bg}}}{\bar{D}_{\text{sub}}}$$

If $\text{CC} > 1$, the subpopulation is more tightly clustered than the background.

**Example:**
- Subpopulation: 7 points in $[4, 6]$, average pairwise distance $\approx 0.8$
- Background: 11 points in $[0, 10]$, average pairwise distance $\approx 5.0$
- Clustering coefficient: $\text{CC} = 5.0 / 0.8 = 6.25$ (subpop is 6.25× more tightly clustered)

---

## Part 3: The Disconnect — Why One Can Be True Without the Other

### Scenario 1: Similar Marginals, Different Local Densities

This is the most common failure case.

**Setup:**
- Both background and subpopulation have the same **marginal distribution** (e.g., roughly uniform on $[0, 100]$)
- But the subpopulation's points are clustered in a small region (e.g., $[40, 60]$), while the background is scattered

**Why this happens:** The background is spread evenly across $[0, 100]$. The subpopulation is concentrated in $[40, 60]$, which is a subset of the background's range.

**Marginal view:**
```
Background:   ●●●●●●●●●●●●●●●●●●●●●
              0 ........................ 100

Subpopulation: ● ● ●●●●●●● ● ●
              0 ........................ 100
```

When you squint and look at just the 1D line, both look roughly like they're spread across $[0, 100]$. The variance might be different, but not dramatically so.

**Local density view:**
```
Background density:   scattered sparse scattered sparse scattered
                      0 ........................ 100

Subpopulation density: sparse sparse sparse dense region sparse sparse
                       0 ........................ 100
                                      [40-60]
```

The subpopulation has a **dense region** where background points are **sparse**. This is invisible in the marginal distribution, but obvious in local density.

### Scenario 2: Different Marginals, Same Local Densities

This is rarer but possible.

**Setup:**
- Background: scattered points in $[0, 100]$
- Subpopulation: scattered points in $[40, 60]$ (different marginal, but same *relative* sparsity)

Both have low local density (points are far apart within each group), but the subpopulation occupies a smaller region.

**Marginal tests would catch this** (different ranges), but **local density tests would not** (both are equally sparse).

---

## Part 4: Concrete 2D Example

Let's make this concrete with a realistic 2D scenario.

### The Setup

Imagine a predictive model where:
- $X$-axis: predicted value $\hat{y}$
- $Y$-axis: true value $y$
- **Background:** 5000 points, scattered across $[0, 100] \times [0, 100]$
- **Subpopulation:** 200 points, clustered in $[45, 55] \times [45, 55]$ (the "ideal prediction" region)

### Visual

```
Background (5000 points):          Subpopulation (200 points):
y                                  y
│                                  │
100├●●●●●●●●●●●●●●●●●●●●       100├────────────
   │●●●●●●●●●●●●●●●●●●●●          │
 75├●●●●●●●●●●●●●●●●●●●●        75├────────────
   │●●●●●●●●●●●●●●●●●●●●          │
 50├●●●●●●●●●●●●●●●●●●●●        50├──●●●●●●──
   │●●●●●●●●●●●●●●●●●●●●          │  ●●●●●●●●
 25├●●●●●●●●●●●●●●●●●●●●        25├──●●●●●●──
   │●●●●●●●●●●●●●●●●●●●●          │
  0└────────────────────           0└───────────
    0  25  50  75  100              0  25  50  75  100
                   ŷ                              ŷ
```

### Marginal Distributions

**Background x-axis ($\hat{y}$):** Roughly uniform on $[0, 100]$
- Mean: $\mu_x = 50$
- Variance: $\sigma^2_x \approx 833$ (high spread)

**Subpopulation x-axis ($\hat{y}$):** Concentrated on $[45, 55]$
- Mean: $\mu_x \approx 50$
- Variance: $\sigma^2_x \approx 8$ (low spread)

**Difference:** The variances are different! A variance test might detect this.

**Background y-axis ($y$):** Roughly uniform on $[0, 100]$
- Mean: $\mu_y = 50$
- Variance: $\sigma^2_y \approx 833$

**Subpopulation y-axis ($y$):** Concentrated on $[45, 55]$
- Mean: $\mu_y \approx 50$
- Variance: $\sigma^2_y \approx 8$

**Again, the variances differ.**

**So why don't variance tests work?**

**Answer:** In many real scenarios, the subpopulation's marginal distributions are *not that different* from the background's. The clustering signal is in 2D geometry, not in the 1D margins.

### Local Density (2D)

Now look at **pairwise distances** (a measure of local density):

**Background:** 5000 points scattered in $[0,100]^2$
- Average inter-point distance: $d_{bg} \approx 10$ (rough estimate)

**Subpopulation:** 200 points clustered in $[45, 55]^2$ (a 10×10 square)
- Average inter-point distance: $d_{sub} \approx 0.5$

**Ratio:** $d_{bg} / d_{sub} = 10 / 0.5 = 20$

The subpopulation is **20 times more densely packed** than the background. This is a **local density signal** that jumps out immediately.

### Why Global Tests Fail

**Mann-Whitney U test on $\hat{y}$:**
- Compares the medians and overall ordering
- If both are roughly centered at 50, the U-test sees "no difference"

**KS test on $\hat{y}$:**
- Compares the CDFs
- If the subpop's CDF is a steeper version in the middle of the background's CDF, the **max gap is small**
- The test sees "no significant difference"

**Variance test on $\hat{y}$:**
- **Might** catch the difference in variances (subpop lower, background higher)
- But depends on sample sizes and how much lower the subpop variance is

**2D local density test (k-NN):**
- **Immediately catches** the clustering
- The subpop's k-NN distances are uniformly small, the background's are large
- Permutation test gives $p \ll 0.001$

---

## Part 5: Quantifying the Difference Formally

### Information-Theoretic View

Think of **entropy** (Shannon entropy) as a measure of "spreadedness":

$$H(X) = -\sum_i p_i \log p_i$$

**Low entropy:** Data is concentrated in few bins (high density)  
**High entropy:** Data is spread across many bins (low density)

The subpopulation's 1D marginal might have **lower entropy** than the background (concentrated values), but this is often a weak signal in practice because both are spread over the same overall range.

The **2D entropy** is very different:
- Background: entropy is high (points scattered everywhere)
- Subpopulation: entropy is low (points in a small region)

But global tests on 1D marginals don't see the 2D entropy difference.

### Divergence Measures

**Kullback-Leibler divergence (KL divergence)** measures how different two distributions are:

$$D_{KL}(P || Q) = \sum_i p_i \log \frac{p_i}{q_i}$$

**On 1D marginals:**
$$D_{KL}(\text{sub}_x || \text{bg}_x) = \text{small or moderate}$$

because both are spread over the same range.

**On 2D densities:**
$$D_{KL}(\text{sub}_{2D} || \text{bg}_{2D}) = \text{very large}$$

because the subpopulation's 2D density (high in $[45, 55]^2$, zero elsewhere) is very different from the background's (uniform over $[0, 100]^2$).

### Wasserstein Distance (Optimal Transport)

The **Wasserstein distance** (Earth Mover's Distance) measures the minimum cost to transport one distribution to another:

$$W(P, Q) = \inf_{\pi} \mathbb{E}_{(x,y) \sim \pi}[\|x - y\|]$$

**On 1D margins:**
$$W(\text{sub}_x, \text{bg}_x) = \text{small}$$

because the subpop is inside the background's support; you only need to "move" points a little.

**This is why EMD fails!** The subpop is a *subset* of the background's support, so the transport cost is inherently low.

**On 2D:**
$$W(\text{sub}_{2D}, \text{bg}_{2D}) = \text{larger, but still can be small}$$

because again, the subpop sits inside the background.

**Local density (k-NN) overcomes this:** It doesn't compare distributions; it asks "are these points close together *relative to random*?" which is a different question entirely.

---

## Part 6: A Taxonomy of Clustering Scenarios

### Type A: Different Marginals + Different Local Densities

**Easiest to detect.**

Example: Background in $[0, 50]$, subpopulation in $[40, 60]$ with tight clustering

- Global tests: **Will likely detect** (different range, variance, etc.)
- Local density tests: **Will definitely detect** (tight clustering)

### Type B: Same Marginals + Different Local Densities

**Hardest to detect with global tests.**

Example: Background uniform on $[0, 100]$, subpopulation clustered in $[40, 60]$

- Global tests: **Might not detect** (marginals are similar)
- Local density tests: **Will definitely detect** (tight clustering)

**This is your case.** Your eyes see Type B clustering. Global tests are blind to Type B.

### Type C: Different Marginals + Same Local Densities

**Rare in practice.**

Example: Background scattered in $[0, 100]$, subpopulation scattered in $[40, 60]$ with similar sparsity

- Global tests: **Will detect** (different ranges)
- Local density tests: **Might not detect** (both equally sparse)

### Type D: Same Marginals + Same Local Densities

**No clustering at all.** Both are random subsets of the same distribution.

- Global tests: **Might find small differences** due to sampling
- Local density tests: **Will not detect** (correct)

---

## Part 7: How to Measure Local Density in Practice

### Method 1: k-Nearest Neighbor Distance (Simple & Non-Parametric)

For each point, compute distance to $k$-th nearest neighbor. Compare distributions.

**Code sketch (1D):**
```julia
# Subpopulation and background as 1D arrays
sub = [4.0, 4.2, 4.5, 5.0, 5.5, 5.8, 6.0]
bg = [0:10...]  # spread across 0-10

# Pool all points
pool = vcat(sub, bg)

# For each subpop point, find distance to 5th nearest neighbor
sub_nnd = [knn_distance(x, pool, k=5) for x in sub]
# Result: sub_nnd ≈ [0.3, 0.3, 0.3, 0.3, 0.3, 0.3, 0.3] (all close)

# For each bg point, find distance to 5th nearest neighbor
bg_nnd = [knn_distance(y, pool, k=5) for y in bg]
# Result: bg_nnd ≈ [1, 1, 1, ..., 1] (all far apart)

# Compare
mean(sub_nnd)  # ≈ 0.3 (tight)
mean(bg_nnd)   # ≈ 1.0 (sparse)
```

**Interpretation:** The subpopulation has **significantly smaller k-NN distances**, indicating **higher local density**.

### Method 2: Kernel Density Estimation (Smooth & Interpretable)

Estimate the density function at each point, then compare.

**Code sketch:**
```julia
using KernelDensity

# Estimate density in subpop vs background
dens_sub = pdf(kde(sub), sub)  # density at each subpop point
dens_bg = pdf(kde(bg), bg)     # density at each bg point

mean(dens_sub)  # ≈ 0.5 (points are in dense regions)
mean(dens_bg)   # ≈ 0.05 (points are in sparse regions)

# Ratio: 0.5 / 0.05 = 10× higher density in subpopulation
```

**Interpretation:** Subpopulation points are in **regions of high density** (each other's neighborhoods), while background points are in **regions of low density** (spread out).

### Method 3: Pairwise Distance Distribution

Compare the distributions of all pairwise distances within each group.

**Code sketch:**
```julia
using Distances

# All pairwise distances in subpopulation
D_sub = pairwise(Euclidean(), sub)
dists_sub = D_sub[triu!(trues(size(D_sub)), 1)]  # upper triangle only

# All pairwise distances in background
D_bg = pairwise(Euclidean(), bg)
dists_bg = D_bg[triu!(trues(size(D_bg)), 1)]

# Compare distributions
mean(dists_sub)  # ≈ 0.7 (average distance between subpop points)
mean(dists_bg)   # ≈ 4.5 (average distance between bg points)

# Histogram
histogram([dists_sub, dists_bg], label=["Subpop", "Background"])
# Subpop distances cluster near 0-2
# Background distances spread across 0-14
```

**Interpretation:** The subpopulation's **pairwise distance distribution is concentrated** (most pairs are close), while the background's is **spread** (pairs are far apart on average).

---

## Part 8: Combining Marginal + Local Density Analysis

In practice, you should report **both**:

1. **Marginal distributions:** Report means, variances, CDFs, quantiles. Use global tests (U, KS, Levene's).
   - Shows: location, spread, overall shape
   - Weakness: misses clustering

2. **Local density:** Report k-NN distances, density estimates, pairwise distances. Use permutation tests.
   - Shows: clustering, spatial concentration
   - Strength: catches tight clusters inside broad backgrounds

### Example Report

> "The subpopulation and background have similar marginal distributions on the predicted value $\hat{y}$ (U-test, $p = 0.18$), both centered at 50 and spanning $[0, 100]$. However, their spatial distributions differ dramatically: subpopulation points cluster in a $10 \times 10$ region ([45, 55]²) with mean pairwise distance of 0.5, while background points are scattered across [0, 100]² with mean pairwise distance of 10. A permutation test on k-NN distances confirms this clustering is highly significant ($k=5$, mean NND: sub = 0.4 vs. bg = 2.1, $p < 0.001$). The subpopulation exhibits ~25× higher local density than the background, despite indistinguishable marginal distributions."

---

## Part 9: Key Takeaways

1. **Marginal distributions** answer: "What does each axis (independently) look like?"
   - Quantified by: mean, variance, CDF, quantiles, entropy

2. **Local density** answers: "How close together are points in each group?"
   - Quantified by: k-NN distances, density estimates, pairwise distances, clustering coefficient

3. **These are independent axes of variation.** You can have:
   - Similar marginals + different local densities (your case)
   - Different marginals + similar local densities
   - Different on both axes
   - Similar on both axes

4. **Global tests see only marginals.** They compare distributions along each axis, then combine the results.

5. **Local density tests see what your eyes see.** They measure whether points are unusually close together.

6. **Clustering is fundamentally a local phenomenon.** It lives in the inter-point distances, not in the marginal summaries.

7. **To detect clustering, you must measure it directly.** Use k-NN, density estimation, or pairwise distances.

---

## Summary Figure

```
                 Marginal Dist.  |  Local Density
                 ───────────────────────────────
Quantification   Mean, Var, CDF  |  k-NN Dist, Density
Test Type        Global, summary |  Local, inter-point
Detects          Location shift  |  Clustering
Blind to         Clustering      |  Marginal diffs
Your scenario    Similar         |  Different ← THIS
                                 |  (tightly packed)
```

---

## Next Steps

If you found Type B clustering (same marginals, different local densities), you should:

1. **Visualize** your data in 2D (predicted vs. true, or any relevant 2D plane)
2. **Quantify local density** using k-NN distances or kernel density estimation
3. **Permutation test** to get a p-value for the clustering signal
4. **Report both marginal and local density analyses** to give a complete picture

See the companion post "When Global Tests Fail: Detecting Subpopulation Clustering via Local Geometry" for implementation details and code.
