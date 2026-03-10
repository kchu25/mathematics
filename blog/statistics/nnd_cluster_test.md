@def title = "When Global Tests Fail: Detecting Subpopulation Clustering via Local Geometry"
@def published = "10 March 2026"
@def tags = ["statistics"]

# When Global Tests Fail: Detecting Subpopulation Clustering via Local Geometry

## The Situation

You have a predictive model with $R^2 \approx 0.6$–$0.7$. You select a subpopulation by some index and look at the corresponding **labels** (the 1D response values). The labels of this subpopulation visually "cluster" — they occupy a tighter, more concentrated region than the background. Contour plots confirm it.

You throw every test you know at the 1D label vectors:
- Mann-Whitney U test → not significant
- KS test → not significant
- Variance tests (Levene's, Brown-Forsythe, Fligner-Killeen) → not significant
- Earth Mover's Distance → small, not significant

Nothing works. But your eyes are not lying. **What is going on?**

---

## Diagnosis: Why Every Global Test Fails

### The core issue: global summaries wash out local structure

Every test listed above computes a **single global summary** of each distribution and compares them:

| Test | What it compares | Blind to |
|------|-----------------|----------|
| U test | $P(X > Y)$ — stochastic dominance | Spread, shape, clustering |
| KS test | $\sup_t \|F(t) - G(t)\|$ — max CDF gap | Local density, spatial tightness |
| Variance tests | $s_1^2$ vs $s_2^2$ — global dispersion | Where the mass is concentrated |
| EMD / Wasserstein | Minimum transport cost | Whether points are near *each other* |

Now consider a concrete scenario. Your background labels span $[0, 100]$ with high variance. Your subpopulation labels cluster in $[40, 60]$ — a tight island in the middle of the sea.

- **U test:** The subpopulation's median ($\approx 50$) is close to the background's median. No location shift → not significant.
- **KS test:** The subpopulation's CDF is a slightly steeper version of the background's CDF in the $[40, 60]$ region, but the maximum gap is small because the background also has plenty of mass there. Not significant.
- **Variance test:** The subpopulation has *lower* variance than the background. But the background has such high variance that the subpopulation's tightness gets drowned out — especially if the background is heavy-tailed or the subpopulation is small.
- **EMD:** Transporting the subpopulation mass to the background mass is cheap, because the subpopulation sits inside the background's support. The cost is low.

**The fundamental problem:** All of these tests compare your subpopulation *against the background distribution*. They ask "does the subpopulation come from a different distribution?" But a tight cluster sitting *inside* a broad background is consistent with being a subsample of that background — the marginal distributions can look compatible even when the local density is very different.

Your eyes see something these tests cannot: the points are **close to each other**.

---

## The Right Question

The tests above all ask variations of:

> "Does the subpopulation's distribution differ from the background's distribution?"

But what you actually observe is:

> "The subpopulation's points are **unusually close together** compared to what you'd expect from a random sample of the background."

This is a question about **local density** or **spatial concentration** — not about marginal distributional differences. You need a test that directly measures inter-point proximity.

---

## The Solution: Nearest-Neighbor Distance (NND) Analysis

### The Hypotheses

**Null Hypothesis ($H_0$):**
> The subpopulation is just a random size-$m$ sample from the pooled set of background + subpopulation points. There is no special clustering — the subpopulation points are no closer together than any other random subset of the same size would be.

**Alternative Hypothesis ($H_a$):**
> The subpopulation points are **spatially concentrated** — they are significantly closer to each other (smaller inter-point distances) than what you would observe if you randomly selected $m$ points from the pooled data.

In other words:
- $H_0$: The subpopulation **is just a random grab-bag** from the background. The observed clustering is consistent with random chance.
- $H_a$: The subpopulation **is a genuine cluster** — an unusually dense clump of points that would be rare under random labeling.

**Which tail?** This is a **one-tailed test** (lower tail). We reject $H_0$ if the observed mean k-NN distance is *unusually small*, not if it's large. A p-value of 0.001 means: "there's a 0.1% chance of observing such tight clustering if the subpopulation were just a random sample."

### The idea

For every point in your subpopulation, measure how far away its nearest neighbors are (within the full dataset or within the background). If the subpopulation is genuinely clustered, these distances will be **systematically smaller** than what you'd see for a random sample of background points.

### Formally

Let $\mathcal{S} = \{x_1, \ldots, x_m\}$ be the subpopulation and $\mathcal{B} = \{y_1, \ldots, y_n\}$ be the background, all in $\mathbb{R}^d$ (or $\mathbb{R}^1$ for your label case).

For each $x_i \in \mathcal{S}$, define the $k$-th nearest-neighbor distance:

$$d_k(x_i) = \text{the distance from } x_i \text{ to its } k\text{-th nearest neighbor in } \mathcal{B} \cup \mathcal{S}$$

The test statistic is:

$$\bar{D}_k^{\mathcal{S}} = \frac{1}{m} \sum_{i=1}^{m} d_k(x_i)$$

the mean $k$-NN distance for the subpopulation.

**If the subpopulation is clustered**, its points are near each other, so $\bar{D}_k^{\mathcal{S}}$ will be small — smaller than you'd expect for a random size-$m$ grab from the background.

### Getting a p-value: the permutation test

There is no closed-form null distribution for $\bar{D}_k^{\mathcal{S}}$, so we use a **permutation test**:

1. Pool all $m + n$ points together.
2. Randomly draw $m$ points (same size as the subpopulation).
3. Compute $\bar{D}_k$ for this random subset.
4. Repeat $B$ times (e.g., $B = 10{,}000$) to build a null distribution.
5. The p-value is:

$$p = \frac{\#\{b : \bar{D}_k^{(b)} \leq \bar{D}_k^{\mathcal{S}}\}}{B}$$

### Interpreting the p-value: What does it really mean?

The p-value is:

> **The probability of observing a mean k-NN distance as small as (or smaller than) the observed one, *if the null hypothesis were true*.**

Breaking this down:

**What the null hypothesis says:** Your subpopulation is just a random $m$-sized subset of the pooled data. Under this assumption, repeated random relabelings would produce a distribution of mean k-NN distances.

**What the p-value quantifies:** How extreme (how small) your observed $\bar{D}_k^{\mathcal{S}}$ is *relative to that null distribution*.

**What it does NOT mean:**
- ❌ It is NOT "the probability that the subpopulation is clustered"
- ❌ It is NOT "the probability that $H_0$ is true"
- ❌ It is NOT "the probability that $H_a$ is false"

### Concrete interpretations

| p-value | Interpretation |
|---------|---|
| $p = 0.001$ | If the subpopulation were random, only **0.1% of relabelings** would produce clustering this tight. Very strong evidence against $H_0$. |
| $p = 0.01$ | Only **1% of random relabelings** would be this tight. Strong evidence. |
| $p = 0.05$ | **5% of random relabelings** would be this tight. Marginal evidence (threshold for $\alpha = 0.05$). |
| $p = 0.15$ | **15% of random relabelings** are this tight. Not rare under randomness — cannot reject $H_0$. |
| $p = 0.50$ | **50% of random relabelings** are equally tight or tighter. The observed clustering is completely ordinary under randomness. |

### Example: interpreting a p-value

Suppose your subpopulation has $\bar{D}_5 = 2.3$ (mean 5-NN distance = 2.3 units). You run 10,000 permutations and find that 47 of them have $\bar{D}_5 \leq 2.3$.

$$p = \frac{47}{10{,}000} = 0.0047$$

**What to report:** "The subpopulation exhibits significantly higher local density than expected under random labeling ($k=5$, mean NND $= 2.3$, $p = 0.0047$)."

**What this means:** If you randomly selected 5,000 different size-$m$ subsets of the pooled data and computed their mean 5-NN distances, only about 47 of them would be as tight or tighter than your subpopulation. This is rare — less than 0.5% — so it's unlikely the subpopulation is just a random grab-bag.

### Common misinterpretations to avoid

**Mistake 1:** "p = 0.001 means there's a 99.9% chance the subpopulation is clustered."

**Correct:** p = 0.001 means there's a 0.1% chance of observing this tight clustering *if the subpopulation were random*. The actual probability the subpopulation is clustered depends on your prior belief (Bayesian reasoning), not just the p-value.

**Mistake 2:** "p = 0.10 means the result is not significant, so there's no clustering."

**Correct:** p = 0.10 means "10% of random relabelings produce this tightness or tighter." This is not extremely rare, but it's also not common. Whether you consider this "significant" depends on your chosen threshold $\alpha$. Convention is $\alpha = 0.05$, but $\alpha = 0.10$ is sometimes used for exploratory work.

**Mistake 3:** "A larger p-value means the effect is smaller."

**Partially correct, but backwards causality:** A larger p-value means the observed clustering is *less extreme under the null*. This could be because:
- The clustering is genuinely weak (true effect size is small).
- Your sample size is small (low power to detect clustering).
- You chose an unfortunate $k$ (not the right scale for the clustering).

---

## Why NND Succeeds Where Global Tests Fail

Consider again: subpopulation in $[40, 60]$, background in $[0, 100]$.

- A random subset of size $m$ from the background will have points scattered across $[0, 100]$, so their nearest-neighbor distances will be large (on average $\sim 100/m$).
- The subpopulation has all $m$ points in a band of width 20, so nearest-neighbor distances are $\sim 20/m$ — much smaller.

The permutation test catches this because it compares the subpopulation's NND to what you'd get by randomly labeling $m$ background points as "subpopulation." A genuinely clustered subset will have smaller NND than virtually all permutations.

**Why global tests missed it:** Global tests compare *distributional summaries* (mean, variance, CDF shape). NND compares *inter-point geometry* — it asks whether the points are close to *each other*, not whether they come from a different marginal distribution.

---

## Addressing the Two Practical Concerns

### 1. Is this computationally feasible?

**Yes.** The dominant cost is building a spatial index and querying it.

- **Building a KD-tree** over $N = m + n$ points: $O(N \log N)$
- **Querying** $k$ nearest neighbors for $m$ points: $O(m \cdot k \cdot \log N)$
- **Permutation loop** ($B$ iterations): $O(B \cdot m \cdot k \cdot \log N)$

For typical sizes ($N \sim 10^4$, $m \sim 10^2$–$10^3$, $k \leq 10$, $B = 10{,}000$), this runs in seconds. Even for $N \sim 10^6$, KD-trees keep the per-query cost at $O(\log N)$, so the permutation test remains tractable.

**In 1D** it's even cheaper: sort the data once ($O(N \log N)$) and find $k$-NN by scanning neighbors in the sorted array ($O(1)$ per query). The entire permutation test is $O(B \cdot m)$.

| $N$ (total points) | $m$ (subpop) | $B$ (perms) | Approximate time |
|---|---|---|---|
| $10^3$ | $100$ | $10{,}000$ | < 1 second |
| $10^4$ | $500$ | $10{,}000$ | ~1–5 seconds |
| $10^5$ | $1{,}000$ | $10{,}000$ | ~10–30 seconds |
| $10^6$ | $5{,}000$ | $10{,}000$ | ~1–5 minutes |

### 2. Isn't the choice of $k$ a heuristic?

**Yes — but it's a well-understood one**, and you can make it principled:

**Why it doesn't matter much in practice.** If the subpopulation is genuinely clustered, the signal shows up across a wide range of $k$. A cluster that is significant at $k = 5$ will almost always also be significant at $k = 3$ and $k = 10$. The choice of $k$ controls the *scale* of the clustering you're looking for — essentially the size of the neighborhood you examine.

| $k$ | What it measures | Pros | Cons |
|-----|-----------------|------|------|
| $k = 1$ | Nearest-pair tightness: are consecutive neighbors close? | Captures very tight, local clustering | Noisy, sensitive to duplicates and gaps |
| $k = 3$–$5$ | **Tight local neighborhoods** | Sweet spot; detects well-defined clusters | May miss looser clusters |
| $k = 5$–$10$ | Neighborhood density | Robust, interpretable scale | Less sensitive to very tight clusters |
| $k = \sqrt{m}$ | Meso-scale neighborhoods | Good default if $m$ is known; stable | Somewhat arbitrary |
| $k = \log(m)$ or $k = m/10$ | Increasingly global | More stable estimates | Less sensitive to fine structure |
| $k \geq m/2$ | Approaching global distribution | Starts to measure overall spread | Defeats the purpose; becomes a variance test |

### Should you use a single $k$ or multiple?

**Short answer: Start with a single well-chosen $k$, then validate with multiple $k$.**

**The strategy:**

1. **Choose a primary $k$.**
   - **If you have no prior knowledge:** Use $k = 5$ (the "default" that works well for most datasets).
   - **If $m$ (subpopulation size) is small** ($m < 50$): Use $k = 3$ or $k = \lceil \sqrt{m} \rceil$.
   - **If $m$ is large** ($m > 500$): You can use $k = 10$ or even $k = 20$ (more stable).
   - **If you expect very tight clusters:** Use $k = 1$ or $k = 3$ (more sensitive).
   - **If you expect loose, diffuse clusters:** Use $k = 10$ or $k = 15$ (less noisy).

2. **Report your primary result.**
   - State $k$ clearly: "We tested clustering using the 5-th nearest neighbor distance ($k=5$)."
   - Report the observed $\bar{D}_k$, p-value, and effect size.

3. **Validate with a sensitivity plot.**
   - Run the test across $k \in \{1, 3, 5, 7, 10, 15, 20\}$ (or a subset).
   - Plot p-value vs. $k$.
   - **Interpretation:**
     - **Robust signal:** p-value stays significant across a wide range of $k$ → true clustering detected.
     - **Fragile signal:** p-value is significant at only one or two $k$ values → may be noise or an artifact of that specific scale.
     - **Scale-dependent:** Different $k$ ranges show different significance → suggests clustering at a specific scale (e.g., tight within $k=3$ but not within $k=20$).

### Example: Sensitivity analysis

Suppose you run your NND test at multiple $k$ and get:

| $k$ | $\bar{D}_k$ | p-value | Conclusion |
|-----|-------------|---------|-----------|
| 1 | 0.5 | 0.003 | ✅ Significant |
| 3 | 1.2 | 0.001 | ✅ Significant |
| 5 | 2.3 | 0.002 | ✅ Significant |
| 7 | 3.1 | 0.008 | ✅ Significant |
| 10 | 4.5 | 0.062 | ⚠️ Borderline |
| 15 | 6.2 | 0.25 | ❌ Not significant |

**Interpretation:** Your subpopulation is **genuinely clustered** at a local scale. The tight clustering is detectable with $k=1$–$10$, but at larger neighborhood sizes ($k=15$), the effect disappears. This suggests the cluster is relatively compact, not a large diffuse group.

### Principled multi-$k$ strategies

If you want to be more formal about using multiple $k$ values:

**Strategy 1: Report multiple $k$ values with Bonferroni correction**
If you run tests at $K$ different values of $k$, you inflate the Type I error rate. Correct by using significance threshold $\alpha / K$ instead of $\alpha$. For example, if testing at $K = 5$ different $k$ values, use $\alpha = 0.05 / 5 = 0.01$ per test. This is conservative (loses power) but controls false positives.

**Strategy 2: Aggregate statistic across $k$**
Combine information from multiple $k$ values using a single aggregated statistic:
$$\bar{D}_{\text{agg}} = \frac{1}{K} \sum_{k=1}^{K} \bar{D}_k^{\mathcal{S}}$$
Run the permutation test *once* on $\bar{D}_{\text{agg}}$ instead of multiple times. This smooths over scale choice and avoids multiple-testing inflation. Still valid under permutation testing.

**Strategy 3: Use Holm-Bonferroni (less conservative)**
If you run tests at $K$ different $k$ values, sort the p-values from smallest to largest: $p_1 \leq p_2 \leq \cdots \leq p_K$. Use thresholds:
- Test 1 (smallest p): Reject if $p_1 < \alpha / K$
- Test 2: Reject if $p_2 < \alpha / (K-1)$
- ...
- Test K: Reject if $p_K < \alpha$

This is less conservative than Bonferroni and is the recommended approach.

### Practical recommendations

**For exploratory work (e.g., your case):**
- Start with $k = 5$ as default.
- Always plot sensitivity across $k = \{1, 3, 5, 7, 10, 15, 20\}$ or similar.
- If you see a plateau of significance, report it as a robust finding.
- If significance is scattered, be cautious — the clustering may be weak or scale-dependent.

**For a final publication:**
- Choose $k$ *before* running the analysis (not by p-hacking).
- Report your pre-specified $k$ prominently.
- Include a sensitivity table (p-value vs. $k$) in supplementary material.
- If you reported multiple $k$, use Holm-Bonferroni correction or the aggregate statistic.

**For a single-shot analysis:**
- Use $k = \lceil \sqrt{m} \rceil$ if $m$ is known (standard rule).
- Or use $k = 5$ if you have no information (robust default).
- The permutation test is valid for any fixed $k$ chosen before seeing the results.

---

## About Distance Metrics

**Important clarification:** The NND test itself is not a distance metric. Rather, it *uses* a distance metric to compute nearest neighbors, and the **mean k-NN distance** is the test statistic.

### Which distance metric should you use?

By default, `NearestNeighbors.jl` uses **Euclidean distance** ($L_2$ norm):

$$d_{\text{Euclidean}}(x, y) = \sqrt{\sum_{j=1}^{d} (x_j - y_j)^2}$$

Other common choices:

| Metric | Formula | When to use |
|--------|---------|-----------|
| Euclidean ($L_2$) | $\sqrt{\sum (x_j - y_j)^2}$ | Default; most interpretable; assumes isotropic geometry |
| Manhattan ($L_1$) | $\sum \|x_j - y_j\|$ | City-block distances; robust to outliers; useful for mixed variable types |
| Chebyshev ($L_\infty$) | $\max_j \|x_j - y_j\|$ | Max component distance; useful for bounded domains |
| Mahalanobis | $(x-y)^T \Sigma^{-1} (x-y)$ | Accounts for correlations; requires inverting covariance matrix |
| Cosine | $1 - \frac{x \cdot y}{\|x\| \|y\|}$ | Angle-based; useful for high-dim normalized data or text |

**Your choice of distance metric matters for interpretation:**
- If your data is **standardized** (zero mean, unit variance per dimension), Euclidean is fine.
- If you have **mixed scales** (e.g., one axis is 0–100, another is 0–1), **standardize first** or use Manhattan distance.
- If you have **directional data** (angles, normalized vectors), use cosine distance.
- If dimensions are **correlated**, Mahalanobis accounts for this — but is more complex.

### Specifying the distance metric in Julia

```julia
using NearestNeighbors, Distances

# Euclidean (default)
tree = KDTree(data)

# Manhattan
tree = KDTree(data, Cityblock())

# Chebyshev
tree = KDTree(data, Chebyshev())

# Mahalanobis (requires precomputed inverse covariance)
using LinearAlgebra, Statistics
Σ = cov(data')  # covariance matrix
M = Mahalanobis(inv(Σ))
tree = KDTree(data, M)
```

**Recommendation for your use case:** If your data is in 1D (just labels) or 2D (pred vs. true), Euclidean is fine. If you standardize each axis first (as shown in the code), the distance is directly interpretable as "standard deviations away."

---

## Julia Implementation

```julia
using NearestNeighbors, Statistics, Random

"""
    nnd_permutation_test(subpop, background; k=5, B=10_000, seed=42)

Test whether `subpop` points are more tightly clustered than
expected under random labeling, using k-NN distances and a
permutation test.

Returns (observed_mNND, null_distribution, p_value).
"""
function nnd_permutation_test(
    subpop::AbstractMatrix,    # d × m  (columns = points)
    background::AbstractMatrix; # d × n
    k::Int = 5,
    B::Int = 10_000,
    seed::Int = 42
)
    rng = MersenneTwister(seed)
    m = size(subpop, 2)
    pool = hcat(subpop, background)  # d × (m+n)
    N = size(pool, 2)

    # Build a single KD-tree over the entire pool
    tree = KDTree(pool)

    # Observed: mean k-NN distance for the subpopulation
    # Use k+1 because each point's 0th neighbor is itself
    idxs, dists = knn(tree, subpop, k + 1, true)
    obs_mNND = mean([d[end] for d in dists])

    # Null distribution: random subsets of size m
    null_mNNDs = zeros(B)
    for b in 1:B
        perm_idx = randperm(rng, N)[1:m]
        rand_pts = pool[:, perm_idx]
        _, dists_perm = knn(tree, rand_pts, k + 1, true)
        null_mNNDs[b] = mean([d[end] for d in dists_perm])
    end

    p_value = count(≤(obs_mNND), null_mNNDs) / B

    return (; obs_mNND, null_mNNDs, p_value)
end
```

### Usage (1D labels)

```julia
# Your data: labels as 1D vectors
bg_labels = randn(5000) .* 30 .+ 50    # background: wide spread around 50
sub_labels = randn(200) .* 5 .+ 50     # subpop: tight cluster around 50

# Reshape to d × n matrices (d=1 for 1D)
bg = reshape(bg_labels, 1, :)
sub = reshape(sub_labels, 1, :)

result = nnd_permutation_test(sub, bg; k=5, B=10_000)

println("Observed mean 5-NN distance:  $(round(result.obs_mNND, digits=4))")
println("Null distribution mean:       $(round(mean(result.null_mNNDs), digits=4))")
println("p-value:                      $(result.p_value)")
# Expected: p ≈ 0.0 (the subpopulation is clearly clustered)
```

### Multi-$k$ sensitivity check

```julia
"""
Run the NND test across multiple k values and return a table.
"""
function nnd_sensitivity(subpop, background; ks=1:2:15, B=10_000)
    results = []
    for k in ks
        r = nnd_permutation_test(subpop, background; k=k, B=B)
        push!(results, (k=k, mNND=r.obs_mNND, p=r.p_value))
    end
    return results
end

for r in nnd_sensitivity(sub, bg; ks=[1, 3, 5, 7, 10, 15, 20])
    println("k=$(lpad(r.k,2)):  mNND=$(round(r.mNND, digits=4)),  p=$(r.p)")
end
# If p < 0.05 across a range of k → robust clustering signal
```

### Extending to 2D (predict vs. label space)

If your contour plots are in 2D (e.g., $y_{\text{pred}}$ vs. $y_{\text{true}}$), use both dimensions:

```julia
# bg_pred, bg_true: background predictions and labels
# sub_pred, sub_true: subpopulation predictions and labels

bg_2d  = vcat(reshape(bg_pred, 1, :),  reshape(bg_true, 1, :))   # 2 × n
sub_2d = vcat(reshape(sub_pred, 1, :), reshape(sub_true, 1, :))   # 2 × m

# Optionally standardize each axis to [0,1] so distances are comparable
using StatsBase
for row in 1:2
    all_vals = vcat(bg_2d[row, :], sub_2d[row, :])
    μ, σ = mean(all_vals), std(all_vals)
    bg_2d[row, :]  .= (bg_2d[row, :] .- μ) ./ σ
    sub_2d[row, :] .= (sub_2d[row, :] .- μ) ./ σ
end

result_2d = nnd_permutation_test(sub_2d, bg_2d; k=5, B=10_000)
println("2D NND test:  p = $(result_2d.p_value)")
```

---

## The Full Strategy: A Decision Ladder

```
You have a subpopulation and a background. Are they different?
│
│ Step 1: Global 1D tests (quick sanity check)
│  ├─ U test       → detects location shift
│  ├─ KS test      → detects any distributional difference
│  └─ Variance test → detects spread difference
│
│ All non-significant? Your signal is NOT in the marginal distribution.
│
│ Step 2: Residual structure
│  └─ Plot (y_pred, y_true) or (y_true, residual) as 2D contours.
│     If contours look different → the signal is in the joint geometry.
│
│ Step 3: Local geometry tests
│  ├─ NND permutation test (this post)
│  │   → "Are subpopulation points closer together than random?"
│  │   → Works in 1D or multi-D. No distributional assumptions.
│  │
│  ├─ Energy test / MMD (see KS test post)
│  │   → "Do the multivariate distributions differ in any way?"
│  │   → More general, but less specifically targeted at clustering.
│  │
│  └─ Classifier-based test (AUROC)
│      → Train a classifier to distinguish subpop from background.
│      → AUROC > 0.6 ≈ the groups are separable.
│      → Feature importance tells you which dimensions matter.
│
│ Step 4: Report
│  └─ Use NND p-value + sensitivity plot across k values
│     to quantify the clustering your contour plots revealed.
```

---

## How to Write This Up

> "Standard univariate tests (Mann-Whitney U, KS, Levene's) failed to distinguish the subpopulation from the background ($p > 0.10$ in all cases), indicating that the marginal label distributions are compatible. However, contour plots in the ($y_{\text{pred}}$, $y_{\text{true}}$) plane revealed that the subpopulation occupies a visibly denser region. To quantify this, we performed a nearest-neighbor distance analysis: for each subpopulation point, we computed the distance to its $k$-th nearest neighbor in the full dataset, and compared the mean NND against a null distribution from $10{,}000$ random permutations. The subpopulation exhibited significantly lower mean NND than expected ($k=5$, $p < 0.001$), and this result was stable across $k \in \{3, 5, 7, 10, 15\}$. This confirms that the subpopulation forms a spatially concentrated cluster that global distributional tests are structurally unable to detect."

---

## Key Takeaways

1. **Global tests (U, KS, variance, EMD) compare distributions.** If your subpopulation is a tight cluster *inside* the background's support, the marginal distributions can look compatible — these tests have no power.
2. **Clustering is an inter-point property**, not a distributional property. You need a test that measures whether points are *close to each other*, not whether they come from a different CDF.
3. **NND + permutation testing** directly asks "are these points closer together than random?" It is non-parametric, assumption-free, works in any dimension, and is computationally cheap (KD-trees keep it fast).
4. **The choice of $k$ is a heuristic, but a manageable one.** Report results across a range of $k$. A genuine cluster produces a plateau of significance. A fragile signal at one specific $k$ is suspect.
5. **For 2D data** (the predict-vs-label plane where you see the contour differences), run NND directly in 2D — this captures exactly the geometric structure your eyes detect.
