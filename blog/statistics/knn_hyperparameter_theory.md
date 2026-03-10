@def title = "The k-NN Hyperparameter: Theory, Optimization, and When to Use Smaller k"
@def published = "10 March 2026"
@def tags = ["statistics", "density-estimation", "nearest-neighbors"]

# The k-NN Hyperparameter: Theory, Optimization, and When to Use Smaller k

## Introduction: Why This Matters

When applying the k-NN permutation test for clustering detection, you face an immediate decision: **which value of $k$ should you use?**

The conventional recommendation is $k = \sqrt{m}$ (where $m$ is the subpopulation size), but this rule-of-thumb is rarely explained beyond "it comes from information theory." More problematically, it's often presented as if it's optimal for all situations — which it isn't.

This post dives into the mathematical foundations:

1. **Why $k \sim n^{1/(d+2)}$ emerges from bias-variance analysis** in density estimation.
2. **Why $k = \sqrt{n}$ is the practical simplification** for most cases.
3. **When and why smaller $k$ values (like $k=1, 3, 5$) can outperform $\sqrt{m}$**.
4. **How to choose $k$ principally** based on your specific clustering scenario.
5. **Rigorous statistical framework** for comparing competing $k$ values.

The key insight: **There is no universal optimal $k$.** The best choice depends on the *geometry and scale* of the clustering you're trying to detect. Understanding the trade-offs — rather than blindly applying a rule — is essential.

---

## Part 1: The Theoretical Foundation for k-NN Scaling

### 1.1 Density Estimation and the Bias-Variance Trade-off

The motivation for $k \sim \sqrt{n}$ comes from **kernel density estimation (KDE)** theory, where k-NN statistics are intimately connected to density estimates.

#### Setting Up the Problem

Suppose we have $n$ i.i.d. points drawn from an unknown density $f(x)$ in $\mathbb{R}^d$. We want to estimate the density at a point $x_0$:

$$\hat{f}(x_0) = \text{estimate of } f(x_0) \text{ using the k-NN distance}$$

**The k-NN density estimator** is:

$$\hat{f}_k(x_0) = \frac{k}{n \cdot V_d(d_k(x_0))}$$

where:
- $k$ = number of neighbors
- $n$ = sample size
- $d_k(x_0)$ = distance to the $k$-th nearest neighbor at $x_0$
- $V_d(r) = c_d \cdot r^d$ = volume of a $d$-dimensional ball of radius $r$ (with $c_d$ depending on the distance metric)

**Intuition:** If the $k$-th neighbor is far away (large $d_k$), the density is low. If it's close (small $d_k$), the density is high. This is encoded in the inverse relationship between $\hat{f}$ and $V_d(d_k)$.

#### The Bias Component

The bias of $\hat{f}_k(x_0)$ is:

$$\text{Bias}[\hat{f}_k(x_0)] = E[\hat{f}_k(x_0)] - f(x_0)$$

**Key result from asymptotic analysis (Rice, Rosenblatt, others):**

$$\text{Bias}[\hat{f}_k] = \frac{1}{2} f''(x_0) \cdot \frac{k}{n \cdot f(x_0)^2} \cdot \mathcal{C}_d + O\left(\frac{k^2}{n^2}\right)$$

where $\mathcal{C}_d$ is a constant depending on dimension $d$ and the specific distance metric.

**Key observation 1:** Bias decreases as $k$ increases.

- Larger $k$ means we average over more neighbors, smoothing out local fluctuations.
- Bias is $O(k/n)$: doubling $k$ halves the bias.
- **For small $k$, bias is small but non-zero.** For large $k$, bias becomes negligible.

#### The Variance Component

The variance of $\hat{f}_k(x_0)$ is:

$$\text{Var}[\hat{f}_k(x_0)] = \frac{f(x_0)^2 \cdot (1 - f(x_0)^2 \cdot V_d(d_k(x_0)))}{n \cdot V_d(d_k(x_0))^2}$$

This simplifies (under standard asymptotics) to:

$$\text{Var}[\hat{f}_k] = \frac{f(x_0)}{n \cdot k \cdot V_d(d_k(x_0))} + O\left(\frac{1}{n^2}\right)$$

**Key observation 2:** Variance increases as $k$ decreases.

- Smaller $k$ means we're averaging over fewer neighbors, so estimates fluctuate more.
- Variance is $O(1/k)$: doubling $k$ halves the variance.
- **For small $k$, variance is large. For large $k$, variance becomes small.**

#### The Bias-Variance Trade-off

We want to minimize the **Mean Squared Error (MSE)**:

$$\text{MSE}[\hat{f}_k] = (\text{Bias}[\hat{f}_k])^2 + \text{Var}[\hat{f}_k]$$

Substituting:

$$\text{MSE}[\hat{f}_k] \approx \left(\mathcal{O}\left(\frac{k}{n}\right)\right)^2 + \mathcal{O}\left(\frac{1}{k}\right) = \mathcal{O}\left(\frac{k^2}{n^2}\right) + \mathcal{O}\left(\frac{1}{k}\right)$$

To minimize, take the derivative with respect to $k$ and set it to zero:

$$\frac{d}{dk}\left[\frac{k^2}{n^2} + \frac{1}{k}\right] = \frac{2k}{n^2} - \frac{1}{k^2} = 0$$

$$\frac{2k}{n^2} = \frac{1}{k^2}$$

$$2k^3 = n^2$$

$$k^* = \left(\frac{n^2}{2}\right)^{1/3} = \frac{n^{2/3}}{2^{1/3}} \approx 0.8 \cdot n^{2/3}$$

**This is the optimal $k$ for 1D density estimation:** $k^* \sim n^{2/3}$.

### 1.2 Generalizing to Multiple Dimensions

In $d$ dimensions, the analysis is more complex because the volume of a ball scales as $r^d$. Following similar bias-variance minimization:

$$\text{MSE}[\hat{f}_k] \approx \left(\frac{k}{n}\right)^{2} + \frac{1}{k \cdot n^{d/(d+1)}}$$

Minimizing this (details omitted for brevity) yields:

$$k^* \sim n^{2/(d+2)}$$

**This is the famous result from density estimation theory:**

| Dimension $d$ | Optimal $k$ | Formula | Example: $n=10{,}000$ |
|---|---|---|---|
| 1 | $k \sim n^{2/3}$ | $n^{2/3}$ | $k^* \approx 215$ |
| 2 | $k \sim n^{1/2}$ | $n^{1/2}$ | $k^* \approx 100$ |
| 3 | $k \sim n^{2/5}$ | $n^{2/5}$ | $k^* \approx 63$ |
| 4 | $k \sim n^{1/3}$ | $n^{1/3}$ | $k^* \approx 21$ |
| 5+ | $k \sim n^{2/(d+2)} \to n^{1/3}$ | Decreases with $d$ | Smaller still |

**Key insight:** As dimension increases, the optimal $k$ *decreases* (relative to $n$). This is the **curse of dimensionality**: in high dimensions, you need fewer neighbors to avoid averaging over an extremely large region.

### 1.3 Why $k = \sqrt{n}$ is the Practical Standard

The formula $k^* = n^{2/(d+2)}$ is asymptotically optimal, but it has practical drawbacks:

1. **It depends on unknown $d$ (the effective dimensionality)**, which is hard to determine in practice.
2. **Different $d$ give different powers of $n$**, making it awkward to remember and apply.
3. **For $d = 2$ (common in practice — your pred-vs-true plots), you get exactly $k = \sqrt{n}$.**

So $k = \sqrt{n}$ became the convention:
- Simple to remember and implement.
- Optimal for $d=2$ (a very common case).
- Reasonable approximation for $d=1$ (uses $n^{1/2}$ instead of optimal $n^{2/3}$, losing some efficiency).
- Reasonable approximation for $d \geq 3$ (uses $n^{1/2}$ instead of smaller optimal powers, erring on the side of more neighbors).

**In summary:** $k = \sqrt{n}$ is a **robust all-purpose heuristic**, not the universally optimal choice.

---

## Part 2: Applying This to the k-NN Clustering Test

Now let's reconnect this to the clustering detection problem. Your null hypothesis is:

> The subpopulation is a random $m$-sized sample from the pooled background + subpopulation.

Under this null, the mean k-NN distance should follow some distribution. We estimate this via permutation testing.

### 2.1 What Dimension $d$ Means in Your Context

When you compute nearest neighbors, you're doing so in some feature space:

- **1D case:** Labels are the only feature. $d = 1$.
- **2D case:** Pred vs. true (or residual vs. true). $d = 2$.
- **Higher dimensions:** Multiple features or coordinates.

The theory above applies directly: the optimal $k$ depends on $d$.

**For your typical use case (1D or 2D labels):**

| Your scenario | $d$ | Optimal $k$ formula | For $m=100$ | For $m=1000$ |
|---|---|---|---|---|
| 1D labels only | 1 | $m^{2/3}$ | $m^{2/3} \approx 21$ | $m^{2/3} \approx 100$ |
| 2D (pred, true) | 2 | $m^{1/2} = \sqrt{m}$ | $10$ | $\approx 32$ |
| Residual-based analysis | 2–3 | $m^{2/5}$ to $m^{1/2}$ | 6–10 | 16–32 |

### 2.2 The Role of $k$ in the Permutation Test

In the NND permutation test, $k$ plays a different role than in classical density estimation. You're not trying to estimate density; you're trying to **detect whether a subset is unusually tightly clustered**.

The bias-variance trade-off still applies, but reframed:

- **Small $k$ (e.g., $k=1, 3, 5$):**
  - **Sensitivity:** Detects *very tight, localized* clustering. High power for well-defined, compact clusters.
  - **Specificity:** May produce false positives from random local fluctuations (noise, outliers, gaps).
  - **Interpretation:** Asking "are the nearest neighbors extremely close?"

- **Large $k$ (e.g., $k=m/2, m/3$):**
  - **Sensitivity:** Detects *loose, diffuse* clustering. Approaches a global measure (variance test).
  - **Specificity:** High — fewer false positives.
  - **Interpretation:** Asking "is the overall spread different?"

- **Medium $k$ (e.g., $k=\sqrt{m}$, $k=5$–$10$):**
  - **Balance:** Detects clustering at a neighborhood scale that captures the cluster structure without being too noisy or too global.
  - **Stability:** Robust across different resampling scenarios.
  - **Interpretation:** Asking "are points at an intermediate neighborhood scale closer than random?"

### 2.3 Statistical Power: When Small $k$ Wins

Consider two clustering scenarios:

**Scenario A: Tight, well-separated cluster**

Your subpopulation points form a tight ball (radius $r_{\text{cluster}}$), far from the background. The background is spread out uniformly.

- **Small $k$ (e.g., $k=1, 3$):** Detects this easily. Every pair of subpopulation points is close. Statistical power is high.
- **Large $k$ (e.g., $k=\sqrt{m}$):** Also detects it, but with lower power because the test statistic is noisier (averaging over more neighbors).

**Scenario B: Large, diffuse cluster**

Your subpopulation points are spread across a region of size $R$, but denser than random background points of the same size.

- **Small $k$ (e.g., $k=1$):** May fail to detect clustering because $k=1$ only looks at nearest pairs. Diffuse clustering has larger nearest-neighbor distances.
- **Large $k$ (e.g., $k=\sqrt{m}$):** Better power because it averages over more neighbors, revealing the overall elevated density.

**Scenario C: Hierarchical or multi-scale clustering**

Your subpopulation has clusters-within-clusters (e.g., multiple tight balls within a broader region).

- **$k=1, 3$:** May detect only the tightest sub-clusters.
- **$k = 5$–$10$:** Detects clustering at an intermediate scale.
- **$k = \sqrt{m}$:** Detects clustering at the largest scale.

**The critical insight:** **There is no single $k$ that is optimal for all clustering geometries.** The best choice depends on the *scale and tightness* of the clustering you expect.

---

## Part 3: When and Why Smaller $k$ Might Be Better

### 3.1 Theoretical Argument for Small $k$

There are legitimate theoretical reasons to prefer $k < \sqrt{m}$ in some situations:

#### Argument 1: Variance Reduction for Sparse Clusters

If your cluster is **very tight** (small radius), the inter-point distances within the cluster are tightly distributed around a small mean $\bar{d}_{\text{cluster}}$.

For small $k$, most of the k-NN distances fall in this tight range. The mean k-NN distance $\bar{D}_k$ has low variance because you're averaging over i.i.d. samples from this distribution.

For large $k$ (approaching $\sqrt{m}$ or more), you start averaging together:
- Distances to neighbors within the tight cluster (small, $\approx \bar{d}_{\text{cluster}}$)
- Distances to neighbors at the *edge* of the cluster (larger)
- Possibly distances to outliers (much larger)

This mixing increases variance. **Your test statistic becomes noisier, reducing power.**

**Mathematical perspective:** If the within-cluster distances follow a tight distribution with variance $\sigma_w^2$, then:

$$\text{Var}[\bar{D}_k] = \frac{\sigma_w^2}{k}$$

But this is only true if all $k$ neighbors are within the cluster. If some $k$ neighbors lie outside, you're mixing distributions, which inflates variance.

#### Argument 2: Curse of Dimensionality for Large $k$

Paradoxically, **using too large $k$ can reduce power in high-dimensional spaces.**

In $d$ dimensions, the expected distance to the $k$-th nearest neighbor, under a uniform density, scales as:

$$d_k \sim \left(\frac{k}{n \cdot c_d}\right)^{1/d}$$

As $k$ grows:
- The denominator effect is weak ($\propto k^{1/d}$, so adding one neighbor increases distance by only $(1 + 1/k)^{1/d} \approx 1 + 1/(d \cdot k)$).
- But the distances start including neighbors far outside the cluster.

In high-dimensional spaces ($d \geq 4$), even "nearby" points in the cluster can be surprisingly far away (curse of dimensionality). Using $k = \sqrt{m}$ might force you to include neighbors outside the cluster.

**Practical implication:** In $d \geq 4$, using $k = m^{1/(d+2)}$ (smaller than $\sqrt{m}$) is more principled. For $d=4$, this suggests $k \sim m^{1/3}$.

#### Argument 3: Optimal Rate for Hypothesis Testing

Recent work in statistical learning theory (not classical density estimation) asks: **what $k$ minimizes Type I and Type II error rates simultaneously in a permutation test?**

Rather than density estimation (which minimizes MSE), hypothesis testing has different objectives. Some results suggest:

- **For tight, well-separated clusters:** Smaller $k$ (even $k=1$) can achieve very high power with low Type I error.
- **For diffuse clusters:** Medium $k$ (around 5–10) is more robust.

This is studied in the context of **k-NN classifiers** and **two-sample testing**. The optimal $k$ for classification often differs from the optimal $k$ for density estimation.

### 3.2 Practical Scenarios Where $k < \sqrt{m}$ Wins

Let me give three concrete examples:

#### Example 1: Tight Outlier Cluster

**Your subpopulation:** 150 points tightly clustered in a region of radius 0.5.
**Background:** 10,000 points uniformly scattered in $[0, 100]^2$ (loosely spread).

Here, $\sqrt{m} = \sqrt{150} \approx 12$.

| $k$ | Mean 5-NN for subpop | Mean 5-NN for random $m=150$ sample | Expected p-value |
|-----|---|---|---|
| 1 | 0.3 | 8–12 | $< 0.001$ |
| 3 | 0.6 | 10–15 | $< 0.001$ |
| 5 | 1.0 | 12–18 | $< 0.001$ |
| 12 | 2.5 | 15–25 | $< 0.001$ |

**Conclusion:** All $k$ values detect clustering, but smaller $k$ has larger effect sizes (0.3 vs. 2.5) and likely **higher power** (test statistic is further from the null mean, relative to null variance).

#### Example 2: Hierarchical Structure

**Your subpopulation:** 300 points forming three sub-clusters (100 points each), with:
- Radius within each sub-cluster: 1.0
- Distance between sub-cluster centers: 20.0

**Background:** 15,000 points uniformly in $[0, 100]^2$.

Here, $\sqrt{m} = \sqrt{300} \approx 17$.

| $k$ | Detects? | What's being measured |
|-----|---|---|
| 1–3 | ✅ Yes | **Tight sub-cluster structure.** Nearest neighbors are in the same mini-cluster. |
| 5–10 | ✅ Yes | **Both levels.** Some neighbors in same sub-cluster, some nearby but in different sub-clusters. |
| 17–30 | ✅ Yes, weaker | **Only the overall cluster envelope.** Neighbors span across sub-cluster boundaries; distances are averaged, reducing contrast. |

**Conclusion:** For hierarchical clustering, **using multiple $k$ values** is essential. $k=3$ reveals fine structure; $k=17$ reveals coarse structure. Neither alone captures the full picture.

#### Example 3: The "Contaminated Uniform" Scenario

**Your subpopulation:** 50 points mixed:
- 40 points uniformly in $[40, 60]^2$ (diffuse cluster)
- 10 outlier points scattered randomly

**Background:** 5,000 points uniformly in $[0, 100]^2$.

Here, $\sqrt{m} = \sqrt{50} \approx 7$.

| $k$ | Result |
|---|---|
| 1–2 | Noisy. The 10 outliers have large 1-NN distances; the 40 dense points have small distances. High variance in the mean k-NN statistic. |
| 3–5 | **Best balance.** Large enough to be robust to outliers, small enough to capture the clustering of the 40 core points. |
| 7+ | Outlier distances get averaged in, diluting the signal. Power decreases. |

**Conclusion:** Medium $k$ is most robust for contaminated data.

---

## Part 4: A Principled Framework for Choosing $k$

Rather than a single rule, use a **hierarchical decision tree**:

### 4.1 Decision Tree for k Selection

```
1. Do you know the dimension d of your data?
   │
   ├─ YES: Use k* = m^(2/(d+2))
   │  ├─ d=1: k* ≈ m^(2/3)
   │  ├─ d=2: k* ≈ m^(1/2) = sqrt(m)
   │  ├─ d=3: k* ≈ m^(2/5)
   │  └─ d≥4: k* ≈ m^(1/3)
   │
   └─ NO: Proceed to 2.

2. Do you expect the clustering to be tight and well-separated?
   │
   ├─ YES (tight cluster):
   │  └─ Use SMALL k: k ∈ {1, 3, 5}
   │     └─ Rationale: Detects local compactness; high power for tight clusters
   │
   ├─ NO (diffuse, multi-scale):
   │  └─ Use MEDIUM k: k ∈ {5, 7, 10} or k = sqrt(m)
   │     └─ Rationale: Robust to noise; detects clustering at multiple scales
   │
   └─ UNSURE: Proceed to 3.

3. Run SENSITIVITY ANALYSIS
   └─ Test across k ∈ {1, 3, 5, 7, 10, 15, 20, sqrt(m)}
   └─ Plot p-value vs. k
   └─ Interpretation:
       ├─ If p-values plateau (all small) across k: ROBUST clustering
       ├─ If p-values spike at small k only: TIGHT clustering
       ├─ If p-values spike at medium k: DIFFUSE clustering
       └─ If p-values are universally large: NO clustering detected
```

### 4.2 Formal Statistical Framework: AIC/BIC for k

You can treat the choice of $k$ as a model selection problem and use **Akaike Information Criterion (AIC)** or **Bayesian Information Criterion (BIC)**.

Define the "model" as: "The subpopulation is clustered at neighborhood scale $k$."

The "loss" is: $-\log p_k$, where $p_k$ is the p-value from the NND test at scale $k$.

Rank models by:

$$\text{AIC}_k = -2 \log p_k + 2 \cdot 1 = -2 \log p_k + 2$$
$$\text{BIC}_k = -2 \log p_k + 1 \cdot \log(B) = -2 \log p_k + \log(B)$$

where $B$ is the number of permutations.

**Lower AIC/BIC is better.** This penalizes choosing too many different $k$ values (to avoid p-hacking).

**Interpretation:** The best $k$ is the one that produces the strongest evidence (lowest p-value) while penalizing overfitting.

### 4.3 Empirical Power Analysis: How to Compare k Values

For a given dataset, you can measure **relative power** across $k$ values without knowing the true effect size:

$$\text{Power}_k = 1 - (\text{Type II error rate at } k)$$

**Estimating power empirically:**

1. **Compute the effect size** (Cohen's $d$ or similar):
   $$d_k = \frac{\bar{D}_k^{\text{observed}} - \overline{\bar{D}_k^{\text{null}}}}{\text{sd}(\bar{D}_k^{\text{null}})}$$

2. **Larger effect sizes indicate higher power** at that $k$:
   $$\text{Estimated Power}_k \propto |d_k|$$

3. **Choose the $k$ with the largest effect size** for your primary analysis.

**Example:** If you run tests at $k \in \{3, 5, 7, 10\}$ and measure effect sizes:

| $k$ | $\bar{D}_k^{\text{obs}}$ | Null mean | Null std | Effect size $d_k$ |
|-----|---|---|---|---|
| 3 | 1.2 | 3.5 | 0.8 | $(3.5-1.2)/0.8 = 2.875$ |
| 5 | 2.1 | 4.8 | 1.0 | $(4.8-2.1)/1.0 = 2.700$ |
| 7 | 3.0 | 6.0 | 1.2 | $(6.0-3.0)/1.2 = 2.500$ |
| 10 | 4.5 | 7.5 | 1.5 | $(7.5-4.5)/1.5 = 2.000$ |

**Conclusion:** $k=3$ has the highest effect size. For tight clusters, this suggests $k=3$ is most powerful.

---

## Part 5: The Dimension-Dependent Optimal Rule

Let me formalize the recommended $k$ by dimension, based on asymptotically optimal density estimation theory:

### 5.1 Dimension-Specific Recommendations

| Dimension | Theoretical optimal | Practical rule | When to use smaller |
|---|---|---|---|
| **$d=1$ (1D labels)** | $k^* = m^{2/3}$ | Use $k = m^{2/3}$ or $k=\sqrt{m}$ as approximation | Small, tight clusters: use $k=3$–$5$ |
| **$d=2$ (pred-vs-true, 2D residuals)** | $k^* = m^{1/2}$ | Use $k = \sqrt{m}$ (exact!) | Well-separated clusters: use $k=3$–$7$ |
| **$d=3$ (multivariate labels)** | $k^* = m^{2/5}$ | Use $k = \lceil m^{2/5} \rceil$ or $k=\sqrt{m}$ | Compact clusters: use $k=3$–$5$ |
| **$d \geq 4$ (high-dimensional)** | $k^* = m^{2/(d+2)} \to m^{1/3}$ | Use $k = \lceil m^{1/3} \rceil$ | All cases: use $k < \sqrt{m}$ |

### 5.2 Hybrid Strategy: Using Multiple k Values Strategically

Rather than choosing a single $k$, use a **multi-scale strategy**:

**Phase 1: Coarse exploration**
- Run tests at $k \in \{1, 5, 10, 20\}$ (or log-spaced values).
- Identify which $k$ range(s) show significant p-values.

**Phase 2: Fine investigation**
- Zoom in on promising ranges with more granular $k$ values.
- If $k=1, 5$ are significant but $k=10, 20$ are not → clustering is tight.
- If $k=10, 20$ are significant but $k=1, 5$ are not → clustering is diffuse.
- If all ranges are significant → robust, multi-scale clustering.

**Phase 3: Principled selection**
- Choose your *primary* $k$ based on the clustering scale revealed in Phase 2.
- Report sensitivity results from Phase 1 and 2 in supplementary material.

### 5.3 Concrete Algorithm: Adaptive k Selection

Here's a Julia pseudo-code for automated, data-driven $k$ selection:

```julia
"""
    select_k_adaptively(subpop, background; k_range=1:20, B=10_000)

Run NND test across a range of k values and recommend the best k
based on effect size and stability.
"""
function select_k_adaptively(subpop, background; k_range=1:20, B=10_000)
    results = []
    
    for k in k_range
        obs, null_dist, pval = nnd_permutation_test(subpop, background; k=k, B=B)
        
        # Compute effect size
        null_mean = mean(null_dist)
        null_std = std(null_dist)
        effect_size = (null_mean - obs) / null_std
        
        push!(results, (k=k, pval=pval, effect_size=effect_size, obs=obs))
    end
    
    # Find k with largest effect size (highest power)
    best_idx = argmax([r.effect_size for r in results])
    best_k = results[best_idx].k
    
    # Check stability: is effect size plateau-like (robust)?
    # or sharp peak (scale-dependent)?
    effect_sizes = [r.effect_size for r in results]
    cv = std(effect_sizes) / mean(effect_sizes)  # coefficient of variation
    
    return (
        recommended_k = best_k,
        all_results = results,
        robustness = cv < 0.2 ? "stable" : "scale-dependent"
    )
end
```

---

## Part 6: Case Studies: When Small k Outperforms sqrt(m)

Here are three realistic scenarios where $k < \sqrt{m}$ is objectively better:

### 6.1 Case Study 1: Rare Disease Subtype Detection

**Setup:**
- Predictive model for disease prognosis with 1,000 background patients.
- A rare subtype (40 patients) is suspected to have systematically different outcomes.
- Outcomes measured as residual log-survival (continuous, 1D).

**Analysis:**

| Analysis step | Details |
|---|---|
| **Expected clustering** | Tight, well-defined subtype. Residuals should cluster in a narrow band. |
| **m (subpop size)** | 40 |
| **d (dimensionality)** | 1 (residuals only) |
| **Theory suggests** | $k^* = m^{2/3} = 40^{2/3} \approx 11.7 \to$ use $k \approx 12$ |
| **Alternative: $\sqrt{m}$** | $\sqrt{40} \approx 6.3 \to$ use $k \approx 6$ |
| **Conservative choice** | Use $k=3$–$5$ (exploit tight cluster structure) |

**Simulated results** (hypothetical but realistic):

| $k$ | p-value | Effect size | Conclusion |
|---|---|---|---|
| 1 | 0.0001 | 4.2 | ✅ Very strong |
| 3 | 0.00005 | 4.8 | ✅ Strongest signal |
| 5 | 0.0001 | 4.5 | ✅ Strong |
| 6 | 0.0002 | 4.0 | ✅ Strong |
| 12 | 0.001 | 3.1 | ✅ Moderate |

**Recommendation:** Use $k=3$ as primary (highest effect size), report sensitivity across $k=\{1,3,5,6,12\}$ showing plateau of significance.

### 6.2 Case Study 2: Model Misspecification Detection

**Setup:**
- Regression model with 500 observations, predicting price from features.
- Suspected cluster: 60 observations where the model systematically underpredicts (large positive residuals).
- Analyze in 2D: (residual, fitted value) space.

**Analysis:**

| Analysis step | Details |
|---|---|
| **Expected clustering** | Tight in residual direction, but could be spread in fitted value direction. |
| **m (subpop size)** | 60 |
| **d (dimensionality)** | 2 (residual, fitted) |
| **Theory suggests** | $k^* = m^{1/2} = \sqrt{60} \approx 7.7 \to$ use $k \approx 8$ |
| **Conservative choice** | Use $k=5$–$7$ for tighter detection |

**Simulated results:**

| $k$ | p-value | Finding |
|---|---|---|
| 1–3 | 0.003 | Tight clustering detected in residual direction. |
| 5–8 | 0.0001 | Strong clustering (residuals clustered across 2D). |
| 10–15 | 0.01 | Clustering weakly detected (neighbors starting to include outside region). |

**Recommendation:** $k = \sqrt{m} \approx 8$ is excellent here. No need for smaller $k$ because the clustering is not super-tight.

### 6.3 Case Study 3: Adversarial Scenario (Where Large k Actually Helps)

**Setup:**
- You have 100 suspected cluster members, but they form a **loose, diffuse** cloud (not tight).
- The cluster is defined by a shared feature (e.g., all customers in a certain region), not by tight response values.

**Analysis:**

| Analysis step | Details |
|---|---|
| **Expected clustering** | Diffuse, spread across moderate range. |
| **m (subpop size)** | 100 |
| **d (dimensionality)** | 1 |
| **Theory suggests** | $k^* = m^{2/3} = 100^{2/3} \approx 21.5$ |
| **Conservative choice** | Use $k = \sqrt{m} = 10$ or even $k=15$–$20$ |

**Simulated results:**

| $k$ | p-value | Interpretation |
|---|---|---|
| 1–3 | 0.25 | Too noisy; nearest neighbors are scattered → no signal. |
| 5–7 | 0.08 | Weak signal, but high variance in test statistic. |
| 10–15 | 0.005 | ✅ **Strong signal.** Averaging over more neighbors reveals clustering. |
| 20 | 0.001 | ✅ Even stronger. |

**Recommendation:** Use $k = \sqrt{m}$ or larger for diffuse clustering.

---

## Part 7: Computational Perspective: Time Complexity Trade-offs

Choosing $k$ also has computational implications:

### 7.1 Time Complexity Analysis

For a permutation test with $B$ replications, total time is:

$$T_{\text{total}} = T_{\text{tree}} + B \cdot T_{\text{per-replication}}$$

where:
- $T_{\text{tree}} = O(N \log N)$ = time to build KD-tree over $N = m+n$ points (one-time)
- $T_{\text{per-replication}} = O(m \cdot k \log N)$ = time to find $k$-NN for $m$ random points

In 1D (sorted array lookup):
$$T_{\text{per-replication}} = O(m \cdot k)$$

### 7.2 Practical Implications

| Scenario | Recommendation | Rationale |
|---|---|---|
| **Large $m$, small budget** | $k=1$ or $k=3$ | Reduces per-replication cost. Tree building dominates anyway. |
| **Medium $m$, moderate budget** | $k=5$–$10$ | Good balance. Cost grows linearly with $k$, but test is robust. |
| **Small $m$ (< 50), high budget** | $k=\sqrt{m}$ or larger | Can afford it. No computational bottleneck. |
| **Real-time constraint** | $k=1$ | Minimal cost; use with sensitivity analysis. |

**Bottom line:** Computational cost is rarely the limiting factor. Choose $k$ based on statistical power, not time.

---

## Part 8: Putting It All Together: A Practical Decision Guide

### 8.1 Quick Decision Tree (For Practitioners)

```
START
│
├─ Do you have prior knowledge of the cluster type?
│  │
│  ├─ YES → Tight, well-separated
│  │  └─ Use k = 3–5 as primary
│  │     └─ Report sensitivity across k ∈ {1,3,5,7,10}
│  │
│  ├─ YES → Diffuse, spread-out
│  │  └─ Use k = sqrt(m) as primary
│  │     └─ Report sensitivity across k ∈ {5,7,10,15}
│  │
│  └─ NO → Unsure
│     └─ Proceed to 8.2
│
├─ 8.2: Run sensitivity analysis
│  └─ Compute p-values and effect sizes for k ∈ {1,3,5,7,10,15}
│     └─ Identify the range(s) with smallest p-values
│     └─ Choose primary k from the range with highest effect size
│
├─ 8.3: Report results
│  └─ Primary p-value (chosen k)
│  └─ Sensitivity table (p vs. k)
│  └─ Effect size comparison
│  └─ Interpretation: Tight vs. diffuse vs. robust clustering
│
END
```

### 8.2 Decision Rules by Data Characteristics

| Data characteristic | Suggested k | Reasoning |
|---|---|---|
| **1D labels, n < 100** | $k \in \{1,3,5\}$ | Small sample; exploit tight structure if present. |
| **1D labels, n ≥ 100** | $k = m^{2/3}$ | Asymptotically optimal for 1D. |
| **2D data (pred, true)** | $k = \sqrt{m}$ | Matches optimal theory exactly for $d=2$. |
| **3D+ features** | $k = m^{2/5}$ for $d=3$, or $k = m^{1/3}$ for $d \geq 4$ | Curse of dimensionality. Use smaller k. |
| **High-dimensional (d > 5)** | $k = \lfloor m^{1/3} \rfloor$ | Strongly reduced by dimensionality. |
| **Unsure about d** | $k = 5$ (default) | Reasonable all-purpose choice; validate with sensitivity. |
| **Tight clusters suspected** | $k \leq 5$ | Leverage compactness. |
| **Noisy data** | $k \geq 7$ | Averaging reduces noise impact. |

---

## Part 9: Synthesis and Recommendations

### Summary of Key Insights

1. **$k = \sqrt{m}$ is optimal for $d=2$**, but not universally.

2. **The true optimal $k$ depends on:**
   - Dimensionality $d$ (via $k^* = m^{2/(d+2)}$)
   - Expected cluster tightness (tight → smaller $k$; diffuse → larger $k$)
   - Noise level in data (noisy → larger $k$; clean → smaller $k$)

3. **Small $k$ (1, 3, 5) can outperform $\sqrt{m}$ when:**
   - Clusters are genuinely tight and well-defined
   - You want to detect local compactness (not just overall spread)
   - You have high signal-to-noise ratio

4. **Large $k$ (or $k = \sqrt{m}$) is safer when:**
   - Clusters are diffuse or multi-scale
   - Data is noisy
   - You want robust, stable estimates
   - $d=2$ (2D data)

5. **No single $k$ is best for all scenarios.** Always run sensitivity analysis across multiple $k$ values.

### Final Recommendations for Your k-NN Clustering Test

**For single-$k$ analysis:**
- **Default (if unsure):** $k = 5$ — simple, robust, well-studied.
- **If you expect tight clusters:** $k = 3$ — high power for compact structure.
- **If you expect diffuse clusters:** $k = \sqrt{m}$ — robust averaging.
- **If dimensionality is known:** Use $k^* = m^{2/(d+2)}$.

**For publication-quality analysis:**
1. Specify $k$ before looking at results (avoid p-hacking).
2. Run sensitivity analysis across a grid of $k$ values (e.g., $k \in \{1,3,5,7,10,15\}$ for small-to-medium samples).
3. Report:
   - Primary p-value and effect size (your chosen $k$)
   - Sensitivity table (p-value and effect size vs. $k$)
   - Interpretation of the pattern (tight cluster → dropping p-values; diffuse → plateau; robust → constant small p-values)
4. If reporting multiple $k$ values, use Holm-Bonferroni correction to account for multiple testing.

**For exploratory analysis:**
- Run sensitivity analysis first.
- Report which $k$ range shows strongest signal.
- Interpret the clustering scale: If $k=1,3$ are significant but $k=20$ is not → tight cluster. If all $k$ are significant → robust, multi-scale clustering.

---

## Conclusion

The rule $k = \sqrt{m}$ is a **good starting point**, rooted in asymptotically optimal density estimation theory. But it is not universally optimal.

**The correct approach is to understand the theory** — why $k \sim n^{2/(d+2)}$ emerges from bias-variance analysis — and then apply it flexibly based on:
- Your data's dimensionality
- The expected geometry of your clusters
- The noise level
- The scale you're trying to detect

**In practice, always use sensitivity analysis.** The pattern of p-values across $k$ values tells you as much about your data as any single result. A plateau of small p-values indicates robust clustering; a sharp peak at one $k$ indicates scale-dependent structure; universally large p-values indicate no clustering at any scale.

With this framework, you can move beyond blind rule-following to principled, data-driven $k$ selection.
