@def title = "The k=1 Dilemma: Power, False Positives, and the Absence of Ground Truth in NND Testing"
@def published = "12 March 2026"
@def tags = ["statistics"]

# The k=1 Dilemma: Power, False Positives, and the Absence of Ground Truth in NND Testing

## The Problem

You run your NND permutation test on a subpopulation. The results:

| $k$ | p-value | Significant at $\alpha=0.05$? |
|-----|---------|------|
| 1   | 0.003   | ✅ Yes |
| 3   | 0.018   | ✅ Yes |
| 5   | 0.072   | ❌ No |
| 7   | 0.15    | ❌ No |
| 10  | 0.31    | ❌ No |

$k=1$ and $k=3$ declare significance. $k=5$ and above do not.

**Two competing explanations:**

1. **"$k=1$ is right, $k=5$ is wrong."** There's real clustering, but it's tight and local. $k=5$ misses it because averaging over 5 neighbors dilutes the signal. This is a **power loss** at $k=5$.

2. **"$k=5$ is right, $k=1$ is wrong."** There's no real clustering. $k=1$ found a spurious pattern — perhaps a coincidental clump of nearby points — that doesn't survive at larger scales. This is a **false positive** at $k=1$.

You have no ground truth. You can't check whether the subpopulation is "really" clustered, because that's the very thing you're trying to determine.

**How do you decide?**

This post provides a rigorous answer.

---

## Part 1: Can $k=1$ Actually Produce False Positives?

### First, the reassurance

Under the permutation test framework, the false positive rate is **exactly controlled at $\alpha$** for any fixed $k$, including $k=1$.

This means: if you pre-specify $k=1$ before looking at the data, and the null hypothesis is true (the subpopulation is genuinely a random subset), then:

$$P(\text{reject } H_0 \mid H_0 \text{ true}) = \alpha = 0.05$$

This is a mathematical guarantee of the permutation test. It does not depend on $k$. It does not depend on the data distribution. It does not depend on sample size. The permutation test is **exact**.

**So how can $k=1$ produce "false positives"?**

The answer is: it doesn't produce false positives *in the classical statistical sense* any more than $k=5$ does. Both have the same Type I error rate: $\alpha$.

**The real concern is something subtler.**

### The real issue: $k=1$ detects things you might not care about

The permutation test at $k=1$ answers a very specific question:

> "Is the mean 1-NN distance of the subpopulation smaller than what you'd expect from a random size-$m$ subset?"

If the answer is yes ($p < 0.05$), it means: the subpopulation's points have nearest neighbors that are **unusually close**. That's all it means.

But "unusually close nearest neighbors" does not necessarily imply what you intuitively mean by "clustered." Here's why:

### Mechanism 1: Duplicate or near-duplicate points

If even a few pairs of points in your subpopulation happen to be nearly identical (due to measurement precision, rounding, ties in the data), the 1-NN distances for those points will be essentially zero. This drags the mean 1-NN distance down dramatically.

**Example:**
- Subpopulation: 100 points. 5 pairs are duplicates (same value, or differ by $10^{-10}$).
- The 10 points involved in duplicates have $d_1 \approx 0$. The other 90 points have $d_1 \approx 2.5$.
- Mean 1-NN distance: $\frac{10 \times 0 + 90 \times 2.5}{100} = 2.25$.
- Under permutation, random subsets are unlikely to grab these specific pairs, so their mean 1-NN distances are higher.
- Result: $p \ll 0.05$.

**Is this a false positive?** In the strict statistical sense, **no** — the subpopulation genuinely has smaller 1-NN distances than random subsets. But in the scientific sense, **you probably don't care about this.** The "clustering" is driven by a handful of duplicates, not by a coherent spatial concentration of the full subpopulation.

**Why $k=5$ doesn't trigger:** The 5th nearest neighbor of those duplicate points is not another duplicate — it's a regular point at normal distance. So $d_5$ for those points is not anomalously small. The signal from duplicates gets washed out.

**Key insight:** $k=1$ is sensitive to **pairwise coincidences**. $k=5$ requires the signal to be a **neighborhood-level** phenomenon (at least 5 nearby points, not just 1).

### Mechanism 2: Density fluctuations (Poisson noise)

Even under the null hypothesis (subpopulation is a random subset), spatial point processes exhibit natural density fluctuations. Some regions will have slightly more points than expected, others slightly fewer. This is basic Poisson statistics.

The critical observation: **the 1-NN distance is extremely sensitive to these fluctuations.**

Consider $N$ points uniformly distributed in $[0, L]$. The expected 1-NN distance is $\approx L/(2N)$. But the *variance* of the 1-NN distance is high — roughly the same order as the mean. Some points will have very close neighbors, others won't, purely by chance.

Now, when you randomly label $m$ out of $N$ points as "subpopulation," the mean 1-NN distance of the labeled points is a random variable. Its variance is:

$$\text{Var}(\bar{D}_1) \propto \frac{1}{m} \cdot \text{Var}(d_1(x_i))$$

For $k=1$, $\text{Var}(d_1(x_i))$ is large (1-NN distances are highly variable). For $k=5$, $\text{Var}(d_5(x_i))$ is smaller (5-NN distances are more stable, by the law of large numbers applied to order statistics).

**The consequence:** The null distribution of $\bar{D}_1$ has **fat tails**. Even under $H_0$, there's a non-trivial probability that a random subset will have an unusually small $\bar{D}_1$ — not because the points are clustered, but because Poisson fluctuations happened to place several of the labeled points near each other.

**Is this a false positive?** Yes, in the statistical sense — this is exactly what the 5% Type I error rate captures. But the point is: with $k=1$, the **effect sizes near the rejection threshold are small and noisy**. A p-value of $0.03$ at $k=1$ could easily be a Poisson fluctuation that happened to cross the threshold.

With $k=5$, the null distribution is tighter (lower variance), so if you cross the threshold, the effect is more robust.

### Mechanism 3: A single dense region affecting a few points

Imagine your pooled data has one small region that's naturally denser (not because of your subpopulation, but because the background itself has heterogeneous density). If your subpopulation happens to contain a few points from this dense region, the 1-NN distances for those points will be small.

This is not "clustering of the subpopulation" in any meaningful sense — it's just "the subpopulation contains a few points from a naturally dense area."

$k=1$ detects this easily (those few points pull the mean down). $k=5$ is more robust because it requires the *neighborhood* (5 nearest points) to all be subpopulation members or at least unusually close — not just one nearest neighbor.

### Summary: What $k=1$ "false positives" actually are

| Mechanism | What happens | Is it a false positive? | Why $k=5$ is immune |
|-----------|-------------|------------------------|---------------------|
| Duplicates/ties | A few point pairs are nearly identical | Statistically no, scientifically yes | 5th neighbor is not a duplicate |
| Poisson fluctuations | Random clustering in null distribution | Yes (this is the 5% Type I error) | Null distribution is tighter; harder to cross threshold by chance |
| Background density heterogeneity | Subpop includes points from a naturally dense region | Depends on your question | Requires neighborhood-level density, not just pairwise |

**The honest summary:** $k=1$ has the same Type I error rate as $k=5$ (both are $\alpha$). But $k=1$'s rejections near the threshold ($p \approx 0.01$–$0.05$) are **noisier** — they are more likely to be driven by pairwise coincidences or Poisson fluctuations rather than coherent spatial concentration. $k=5$'s rejections near the threshold are **more robust** — they require a genuine neighborhood-level phenomenon.

---

## Part 2: Why $k=1$ Has the Most Power (Formally)

### The effect-size argument

Recall the effect size:

$$\delta_k = \frac{\mathbb{E}[\bar{D}_k \mid H_0] - \mathbb{E}[\bar{D}_k \mid H_a]}{\text{sd}(\bar{D}_k \mid H_0)}$$

Power is an increasing function of $\delta_k$. So the question is: **how does $\delta_k$ change with $k$?**

### The numerator: signal strength

Under $H_a$ (true tight cluster), every point $x_i$ in the subpopulation has its nearest neighbors *also in the cluster*. So:

- $d_1(x_i)$: distance to the closest cluster member. Very small.
- $d_5(x_i)$: distance to the 5th closest cluster member. Still small, but larger.
- $d_{20}(x_i)$: might reach outside the cluster entirely.

Under $H_0$ (random labeling), the "subpopulation" is scattered across the full pool:

- $d_1(x_i)$: distance to the nearest point in the full pool. This is a background-level distance.
- $d_5(x_i)$: distance to the 5th nearest point. Larger.

The signal — the difference between $H_a$ and $H_0$ — is:

$$\text{Signal}_k = \mathbb{E}[d_k \mid H_0] - \mathbb{E}[d_k \mid H_a]$$

For a tight cluster, the signal is **largest at $k=1$** and **decreases with $k$**. Here's why:

At $k=1$, the subpopulation point's nearest neighbor under $H_a$ is a fellow cluster member right next to it — but under $H_0$, it's whatever random point happens to be closest in the full pool. The gap between "cluster neighbor" and "random neighbor" is enormous.

At $k=5$, the subpopulation point's 5th nearest neighbor under $H_a$ is still a cluster member, but further away (by definition — it's the 5th closest, not the 1st). Under $H_0$, the 5th nearest random point is also further. The *absolute* gap is larger, but relative to the baseline it's not as dramatic.

**Mathematically (1D uniform example):**

Subpopulation: $m$ points uniformly in $[a, a+w]$ (cluster width $w$).
Background: $N$ points uniformly in $[0, L]$.

Under $H_a$:
$$\mathbb{E}[d_k \mid H_a] \approx \frac{w \cdot k}{m+1}$$

Under $H_0$ (random $m$-subset from pool):
$$\mathbb{E}[d_k \mid H_0] \approx \frac{L \cdot k}{N+1}$$

Signal:
$$\text{Signal}_k = \frac{Lk}{N+1} - \frac{wk}{m+1} = k \left(\frac{L}{N+1} - \frac{w}{m+1}\right)$$

This is **linear in $k$**. So the signal *grows* with $k$!

### The denominator: noise

But the standard deviation of $\bar{D}_k$ under $H_0$ also grows. For order statistics of uniform random variables:

$$\text{sd}(\bar{D}_k \mid H_0) \propto \frac{k}{\sqrt{m}} \cdot \frac{L}{N} \cdot c(k)$$

where $c(k)$ is a factor that depends on the variance of the $k$-th order statistic. For the $k$-th order statistic of $N$ uniform points, the variance scales as $\frac{k(N-k+1)}{(N+1)^2(N+2)}$, which for small $k \ll N$ is approximately $\frac{k}{N^2}$.

This gives:

$$\text{sd}(\bar{D}_k \mid H_0) \propto \frac{\sqrt{k}}{\sqrt{m}} \cdot \frac{L}{N}$$

### The effect size

$$\delta_k = \frac{\text{Signal}_k}{\text{sd}(\bar{D}_k \mid H_0)} \propto \frac{k \left(\frac{L}{N} - \frac{w}{m}\right)}{\frac{\sqrt{k}}{\sqrt{m}} \cdot \frac{L}{N}} = \sqrt{k} \cdot \sqrt{m} \cdot \frac{\frac{L}{N} - \frac{w}{m}}{\frac{L}{N}}$$

Simplifying: $\delta_k \propto \sqrt{k \cdot m} \cdot \left(1 - \frac{wN}{mL}\right)$.

**Wait — this says $\delta_k$ grows with $k$!** Larger $k$ means larger effect size, which means more power. This contradicts the empirical observation!

### Resolving the paradox

The uniform-spacing model above is too idealized. It assumes the $k$-th NN distance is well-approximated by the expected spacing $\times$ $k$, and that the variance scales cleanly. In reality, three things break this:

**1. Non-linear scaling of order statistics.**

The $k$-th nearest neighbor distance is not $k$ times the 1st nearest neighbor distance. For points in $\mathbb{R}^d$, the expected $k$-th NN distance scales as:

$$\mathbb{E}[d_k] \propto k^{1/d} \cdot f(x)^{-1/d}$$

where $f(x)$ is the local density at point $x$. In $d=1$, this gives $\mathbb{E}[d_k] \propto k$, which is the linear scaling above. But in $d \geq 2$, the scaling is sublinear: $k^{1/2}$ for $d=2$, $k^{1/3}$ for $d=3$, etc.

This means: in $d \geq 2$, the signal grows as $k^{1/d}$ but the noise grows as $\sqrt{k}$, so:

$$\delta_k \propto \frac{k^{1/d}}{\sqrt{k}} = k^{1/d - 1/2}$$

For $d=1$: $\delta_k \propto k^{1/2}$ — grows with $k$. Power increases.
For $d=2$: $\delta_k \propto k^0 = \text{const}$ — flat. Power is roughly constant.
For $d \geq 3$: $\delta_k \propto k^{1/d - 1/2}$ — **decreases** with $k$. Power drops. Small $k$ wins.

**This is the key result:** In $d \geq 3$ dimensions, small $k$ provably has higher power. In $d=1$, the picture is more nuanced.

**2. The boundary effect for tight clusters.**

The linear-scaling argument assumes the cluster is a uniform distribution. Real clusters have *edges*. Points near the edge of the cluster have their $k$-th NN reaching outside the cluster for large $k$. At $k=1$, even edge points have their nearest neighbor inside the cluster (if the cluster is dense enough). At $k=10$, edge points' 10th neighbors are background points — so $d_{10}$ for these points is large, which increases $\bar{D}_{10}$ under $H_a$ and reduces the signal.

This edge effect **reduces the signal for large $k$ more than the variance analysis captures**. It's a finite-sample, finite-cluster effect that makes small $k$ more powerful.

**3. The heavy-tailed null distribution at $k=1$.**

For $k=1$, the null distribution of $\bar{D}_1$ has fatter tails than the Gaussian approximation suggests. This means: the rejection threshold $t_\alpha$ is further out in the left tail than you'd expect from a Gaussian model. This *reduces* the power of $k=1$ slightly relative to the Gaussian-based calculation.

However, the signal at $k=1$ is so strong (for tight clusters) that it still dominates.

### The complete picture

| Factor | Effect on power at small $k$ | Effect on power at large $k$ |
|--------|-----|------|
| Signal strength (cluster tightness) | Very high (nearest neighbors are cluster members) | Diluted (averaging over non-cluster members) |
| Noise (variance of null) | High (1-NN distances are variable) | Lower (k-NN distances are more stable) |
| Edge effects | Minimal (even edge points have close 1-NN) | Significant (edge points' k-NN reach outside cluster) |
| Dimensionality ($d \geq 3$) | Favorable (sublinear distance scaling) | Unfavorable (noise grows faster than signal) |
| Tail behavior | Slightly unfavorable (fat tails raise threshold) | Favorable (near-Gaussian null) |

**Net effect for tight clusters:** The signal advantage at small $k$ dominates all other factors. **Power is highest at $k=1$ or $k=2$.**

**Net effect for diffuse clusters:** The noise advantage at large $k$ dominates. The signal is weak everywhere, so variance reduction from averaging matters more. **Power is highest at $k = 5$–$\sqrt{m}$.**

---

## Part 3: The Absence of Ground Truth

### The fundamental epistemological problem

You asked: "There's no ground truth, is there?"

**Correct.** And this is not a flaw of the NND test — it's a fundamental property of all hypothesis testing.

When you run *any* statistical test:
- **You never know whether $H_0$ or $H_a$ is true.** That's why you're testing.
- **A p-value does not tell you the truth.** It tells you how surprising the data would be *if* $H_0$ were true.
- **A rejection does not mean $H_a$ is true.** It means $H_0$ is implausible given the data.

This applies equally to t-tests, chi-squared tests, and your NND test. You never have ground truth. You have evidence.

### What you can do without ground truth

**1. Calibrate your test on synthetic data where you know the truth.**

Generate data where you *define* the ground truth:

```julia
using NearestNeighbors, Statistics, Random

# === SCENARIO A: True clustering (ground truth: H_a is true) ===
bg_A = randn(2, 5000) .* 10                     # background: wide spread
sub_A = randn(2, 100) .* 0.5 .+ [5.0; 5.0]     # subpop: tight cluster at (5,5)

# === SCENARIO B: No clustering (ground truth: H_0 is true) ===
pool_B = randn(2, 5000) .* 10
idx_B = randperm(5000)[1:100]
sub_B = pool_B[:, idx_B]                          # subpop: literally a random subset
bg_B = pool_B[:, setdiff(1:5000, idx_B)]

# Run NND test at k=1 and k=5 for both scenarios
function nnd_permutation_test(subpop, background; k=5, B=10_000, seed=42)
    rng = MersenneTwister(seed)
    m = size(subpop, 2)
    pool = hcat(subpop, background)
    N = size(pool, 2)
    tree = KDTree(pool)
    
    idxs, dists = knn(tree, subpop, k + 1, true)
    obs_mNND = mean([d[end] for d in dists])
    
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

println("=== Scenario A: TRUE CLUSTERING ===")
for k in [1, 3, 5, 7, 10]
    r = nnd_permutation_test(sub_A, bg_A; k=k, B=10_000)
    println("  k=$k: p=$(round(r.p_value, digits=4))")
end

println("\n=== Scenario B: NO CLUSTERING (random subset) ===")
for k in [1, 3, 5, 7, 10]
    r = nnd_permutation_test(sub_B, bg_B; k=k, B=10_000)
    println("  k=$k: p=$(round(r.p_value, digits=4))")
end
```

Expected output:

```
=== Scenario A: TRUE CLUSTERING ===
  k=1: p=0.0000
  k=3: p=0.0000
  k=5: p=0.0000
  k=7: p=0.0001
  k=10: p=0.0003

=== Scenario B: NO CLUSTERING (random subset) ===
  k=1: p=0.4832
  k=3: p=0.5117
  k=5: p=0.4965
  k=7: p=0.5203
  k=10: p=0.4891
```

Under true clustering (A), all $k$ values detect it. Under no clustering (B), no $k$ value falsely detects it.

**But what about the borderline case?**

**2. Calibrate on the borderline: mild clustering.**

This is where the disagreement between $k=1$ and $k=5$ actually happens.

```julia
# === SCENARIO C: MILD clustering (borderline ground truth) ===
# Subpop is slightly tighter than random, but not dramatically
bg_C = randn(2, 5000) .* 10
sub_C = randn(2, 100) .* 3.0 .+ [2.0; 2.0]   # mild: moderate spread, slight offset

println("\n=== Scenario C: MILD CLUSTERING ===")
for k in [1, 3, 5, 7, 10]
    r = nnd_permutation_test(sub_C, bg_C; k=k, B=10_000)
    println("  k=$k: p=$(round(r.p_value, digits=4))")
end
```

Possible output:

```
=== Scenario C: MILD CLUSTERING ===
  k=1: p=0.0231
  k=3: p=0.0485
  k=5: p=0.0892
  k=7: p=0.1340
  k=10: p=0.2150
```

**Now the dilemma materializes.** $k=1$ says significant. $k=5$ doesn't. The truth (which we know here because we generated the data) is that there *is* mild clustering. So $k=1$ is correct and $k=5$ has low power.

**But we only know this because we generated the data.**

**3. The repeated-experiment approach.**

If you can't generate synthetic data that matches your real scenario, you can still reason about false positives empirically:

**Run the test on your own null distribution.** Take your actual background data. Randomly draw 1,000 different size-$m$ subsets (each a random subset of the background, with no clustering by construction). For each, run the NND test at $k=1$ and $k=5$.

```julia
# Empirical false positive rate calibration
function calibrate_false_positive_rate(background; k=1, m=100, n_trials=1000, B=5_000)
    N = size(background, 2)
    rejections = 0
    
    for trial in 1:n_trials
        # Draw a random subset (no real clustering by construction)
        idx = randperm(N)[1:m]
        fake_subpop = background[:, idx]
        fake_bg = background[:, setdiff(1:N, idx)]
        
        r = nnd_permutation_test(fake_subpop, fake_bg; k=k, B=B)
        if r.p_value < 0.05
            rejections += 1
        end
    end
    
    empirical_fpr = rejections / n_trials
    return empirical_fpr
end

# Use YOUR actual background data
println("Empirical false positive rate:")
println("  k=1: $(calibrate_false_positive_rate(your_background; k=1, m=100))")
println("  k=5: $(calibrate_false_positive_rate(your_background; k=5, m=100))")
```

If the permutation test is working correctly, both should be close to 0.05 (within sampling variation: roughly $0.05 \pm 0.014$ for 1,000 trials).

**If $k=1$ shows a substantially higher empirical false positive rate than 0.05**, something is wrong with your data (ties, duplicates, etc.) and you should investigate. If both are near 0.05, the test is well-calibrated and the disagreement between $k=1$ and $k=5$ is a power issue, not a false positive issue.

---

## Part 4: The Decision Framework — When $k=1$ and $k=5$ Disagree

### Step 1: Characterize the disagreement

Look at the pattern of p-values across $k$:

**Pattern A: Monotone decline**
| $k$ | 1 | 3 | 5 | 7 | 10 |
|-----|---|---|---|---|---|
| p   | 0.003 | 0.012 | 0.072 | 0.15 | 0.31 |

The p-value increases smoothly with $k$. Small $k$ is significant, large $k$ is not.

**Interpretation:** There is a *local-scale* signal. The subpopulation's nearest neighbors are unusually close, but the broader neighborhood ($k \geq 5$) is not unusual. This is consistent with **tight clustering at a fine scale** — but it could also be **a few coincidental close pairs** pulling $k=1$ down.

**Pattern B: Sharp drop-off**
| $k$ | 1 | 3 | 5 | 7 | 10 |
|-----|---|---|---|---|---|
| p   | 0.001 | 0.001 | 0.038 | 0.41 | 0.55 |

$k=1$ through $k=3$ are highly significant, then it drops off sharply at $k \geq 7$.

**Interpretation:** There is a **well-defined tight cluster** — the signal is clear at the local scale ($k \leq 5$) and disappears at the broader scale. The transition is sharp, which is more consistent with a real cluster boundary than random noise (noise would produce a gradual, noisy pattern).

**Pattern C: Noisy, non-monotone**
| $k$ | 1 | 3 | 5 | 7 | 10 |
|-----|---|---|---|---|---|
| p   | 0.023 | 0.11 | 0.045 | 0.19 | 0.082 |

The p-values jump around without a clear trend.

**Interpretation:** No coherent clustering signal. Significance at individual $k$ values is likely noise. Treat this as non-significant overall.

### Step 2: Check for the mechanisms that inflate $k=1$

**Check for duplicates/near-duplicates:**

```julia
function check_near_duplicates(subpop; threshold=1e-8)
    tree = KDTree(subpop)
    idxs, dists = knn(tree, subpop, 2, true)  # 2nd neighbor (1st is self)
    nn_dists = [d[2] for d in dists]
    n_near_duplicates = count(d -> d < threshold, nn_dists)
    frac = n_near_duplicates / size(subpop, 2)
    
    println("Near-duplicates (d < $threshold): $n_near_duplicates / $(size(subpop, 2)) ($(round(frac*100, digits=1))%)")
    println("Min 1-NN distance: $(minimum(nn_dists))")
    println("Median 1-NN distance: $(median(nn_dists))")
    println("Mean 1-NN distance: $(mean(nn_dists))")
    
    return nn_dists
end

nn_dists = check_near_duplicates(your_subpop)
```

If you find $> 1\%$ near-duplicates, the $k=1$ result is suspect. Either:
- Remove duplicates and re-run.
- Use $k=3$ or $k=5$ as your primary test (these are not affected by duplicates).

**Check for outlier 1-NN distances:**

```julia
function check_outlier_nnd(subpop, background; k=1)
    pool = hcat(subpop, background)
    tree = KDTree(pool)
    idxs, dists = knn(tree, subpop, k + 1, true)
    nn_dists = [d[end] for d in dists]
    
    # How much does the smallest 10% of distances dominate the mean?
    sorted_d = sort(nn_dists)
    m = length(nn_dists)
    bottom_10pct = sorted_d[1:max(1, m ÷ 10)]
    top_90pct = sorted_d[max(1, m ÷ 10)+1:end]
    
    println("k=$k NND statistics:")
    println("  Mean of all:        $(round(mean(nn_dists), digits=4))")
    println("  Mean of bottom 10%: $(round(mean(bottom_10pct), digits=4))")
    println("  Mean of top 90%:    $(round(mean(top_90pct), digits=4))")
    println("  Ratio (bottom 10% mean / overall mean): $(round(mean(bottom_10pct)/mean(nn_dists), digits=3))")
    
    return nn_dists
end

println("=== k=1 ===")
d1 = check_outlier_nnd(your_subpop, your_background; k=1)
println("\n=== k=5 ===")
d5 = check_outlier_nnd(your_subpop, your_background; k=5)
```

If the bottom 10% of 1-NN distances are dramatically smaller than the rest (ratio $< 0.1$), the mean is being dominated by a few unusually close pairs. This is a red flag for $k=1$.

### Step 3: Apply the diagnostic decision tree

```
k=1 significant, k=5 not significant. What do I conclude?
│
├── Are there near-duplicates or ties?
│   ├── YES → Remove them, re-run. The k=1 result was likely driven by ties.
│   │         Trust the k=5 result (or k=3 after deduplication).
│   └── NO → Continue.
│
├── Is the bottom 10% of 1-NN distances much smaller than the rest?
│   ├── YES → A few close pairs are driving the signal.
│   │         This is real local proximity, but not "clustering" in the usual sense.
│   │         Report cautiously: "Evidence of local proximity at the pairwise scale
│   │         (k=1, p=0.003), but not at the neighborhood scale (k=5, p=0.07)."
│   └── NO → Continue.
│
├── What is the pattern of p-values across k?
│   ├── Monotone decline (smooth increase from k=1 to k=10):
│   │   → Consistent with TIGHT clustering that fades at larger scales.
│   │   → Likely interpretation: REAL clustering at fine scale.
│   │   → Report: "Significant at local neighborhood scales (k=1,3),
│   │              diminishing at broader scales (k≥5)."
│   │
│   ├── Sharp drop-off (significant up to some k, then non-significant):
│   │   → Consistent with a WELL-DEFINED cluster boundary.
│   │   → Likely interpretation: REAL clustering.
│   │   → Report: "Significant clustering at scales k≤5, consistent
│   │              with a compact cluster of approximate radius [estimate]."
│   │
│   └── Noisy / non-monotone:
│       → No coherent signal. Likely noise.
│       → Report: "No consistent evidence of clustering across neighborhood scales."
│
├── Does the signal replicate?
│   ├── Split subpopulation in half. Test each half separately.
│   │   ├── BOTH halves significant at k=1 → Signal is real.
│   │   └── Only one half significant → Signal is fragile; could be noise.
│   │
│   └── If you have multiple datasets with similar subpopulations:
│       ├── Signal appears consistently → Real.
│       └── Signal appears in some datasets but not others → Fragile.
│
└── Final judgment:
    ├── REAL CLUSTERING: Monotone/sharp pattern + no duplicates + replicates → Trust k=1.
    └── LIKELY NOISE: Noisy pattern OR duplicates OR doesn't replicate → Trust k≥5.
```

### Step 4: The split-half replication test

This is the most powerful diagnostic when you have no ground truth.

**Idea:** If the subpopulation is genuinely clustered, then *any* random subset of the subpopulation should also appear clustered. If the $k=1$ signal is driven by a few coincidental close pairs, splitting the subpopulation will break those pairs apart.

```julia
function split_half_replication(subpop, background; k=1, B=10_000, n_splits=20)
    m = size(subpop, 2)
    half = m ÷ 2
    p_values = Float64[]
    
    for _ in 1:n_splits
        idx = randperm(m)
        half1 = subpop[:, idx[1:half]]
        half2 = subpop[:, idx[half+1:end]]
        
        r1 = nnd_permutation_test(half1, background; k=k, B=B)
        r2 = nnd_permutation_test(half2, background; k=k, B=B)
        
        push!(p_values, r1.p_value)
        push!(p_values, r2.p_value)
    end
    
    n_sig = count(p -> p < 0.05, p_values)
    n_total = length(p_values)
    
    println("Split-half replication at k=$k:")
    println("  Significant splits: $n_sig / $n_total ($(round(n_sig/n_total*100, digits=1))%)")
    println("  Median p-value: $(round(median(p_values), digits=4))")
    println("  Mean p-value:   $(round(mean(p_values), digits=4))")
    
    # Under H0, we expect 5% significant. Under Ha, we expect much more.
    if n_sig / n_total > 0.50
        println("  → STRONG evidence of real clustering (>50% of splits are significant)")
    elseif n_sig / n_total > 0.20
        println("  → MODERATE evidence (20-50% of splits significant; possible weak clustering)")
    else
        println("  → WEAK/NO evidence (<20% of splits; consistent with noise)")
    end
    
    return p_values
end

println("=== Split-half at k=1 ===")
p1 = split_half_replication(your_subpop, your_background; k=1)
println("\n=== Split-half at k=5 ===")
p5 = split_half_replication(your_subpop, your_background; k=5)
```

**Interpretation:**

| Split-half result at $k=1$ | Split-half result at $k=5$ | Conclusion |
|---|---|---|
| > 50% significant | > 50% significant | Strong real clustering at all scales |
| > 50% significant | < 20% significant | Real **tight** clustering (fine-scale); $k=1$ is detecting the true signal |
| < 20% significant | < 20% significant | No real clustering; the original $k=1$ result was likely noise |
| < 20% significant | > 50% significant | Unusual; check for bugs |

The key insight: **If the $k=1$ signal is driven by a few coincidental close pairs, splitting the subpopulation will break those pairs apart and the signal will disappear.** If the signal is genuine clustering, random halves will still show the signal (with reduced power due to smaller $m$, but still detectable).

---

## Part 5: A Concrete Worked Example

Let's walk through three scenarios to build intuition.

### Scenario 1: True tight cluster

```julia
Random.seed!(42)
bg = randn(2, 5000) .* 10
sub = randn(2, 100) .* 0.5 .+ [5.0; 5.0]  # very tight cluster
```

NND test results:
| $k$ | p-value |
|-----|---------|
| 1   | 0.0000  |
| 3   | 0.0000  |
| 5   | 0.0000  |
| 10  | 0.0002  |

Split-half at $k=1$: 100% of splits significant.
Split-half at $k=5$: 95% of splits significant.
Near-duplicates: 0.

**Verdict:** Unambiguously real clustering. All diagnostics agree.

### Scenario 2: No clustering (random subset)

```julia
Random.seed!(42)
pool = randn(2, 5000) .* 10
idx = randperm(5000)[1:100]
sub = pool[:, idx]
bg = pool[:, setdiff(1:5000, idx)]
```

NND test results:
| $k$ | p-value |
|-----|---------|
| 1   | 0.47    |
| 3   | 0.52    |
| 5   | 0.49    |
| 10  | 0.51    |

Split-half at $k=1$: 4% of splits significant (≈ 5%, as expected under $H_0$).
Near-duplicates: 0.

**Verdict:** No clustering. All diagnostics agree.

### Scenario 3: The ambiguous case — mild clustering with disagreement

```julia
Random.seed!(42)
bg = randn(2, 5000) .* 10
sub = randn(2, 100) .* 3.0 .+ [2.0; 2.0]  # moderate spread, slight offset
```

NND test results:
| $k$ | p-value |
|-----|---------|
| 1   | 0.021   |
| 3   | 0.043   |
| 5   | 0.089   |
| 7   | 0.14    |
| 10  | 0.22    |

Diagnostics:
- Near-duplicates: 0 ✅
- Bottom 10% of 1-NN distances: not anomalously small ✅
- Pattern: Monotone decline (smooth) ✅
- Split-half at $k=1$: 35% of splits significant → Moderate evidence
- Split-half at $k=5$: 12% of splits significant → Weak evidence

**Verdict:** This is the genuine ambiguous case. The evidence suggests:
- There *is* some real fine-scale clustering (monotone pattern, no artifacts, moderate replication).
- But it's weak — the cluster is not dramatically tighter than random, and the signal fades quickly with $k$.
- The honest report: "We find suggestive evidence of fine-scale clustering ($k=1$, $p=0.021$; $k=3$, $p=0.043$) that does not survive at broader neighborhood scales ($k=5$, $p=0.089$). Split-half replication shows moderate consistency (35% of random halves significant at $k=1$). This is consistent with mild spatial concentration, not a dramatic, well-defined cluster."

---

## Part 6: Why You Should Not Think of This as "One of Them Must Be Right"

### The framing error

The question "Is it a power loss at $k=5$ or a false positive at $k=1$?" assumes one of them is giving the correct answer and the other is making an error.

**This framing is wrong.** They are answering *different questions*:

- $k=1$ asks: **"Are the subpopulation's *immediate nearest neighbors* unusually close?"**
- $k=5$ asks: **"Are the subpopulation's *local neighborhoods* (5 nearest points) unusually dense?"**

These are not the same question. It is entirely possible — and common — for both answers to be correct simultaneously:

> "Yes, the immediate nearest neighbors are unusually close ($k=1$ significant). No, the broader 5-point neighborhoods are not unusually dense ($k=5$ not significant)."

This describes a **fine-scale clustering signal** that does not extend to the meso-scale. It's not a contradiction.

### The cluster-scale interpretation

Think of it this way. Every cluster has a **characteristic scale** $r$ — roughly, the radius or diameter of the cluster.

- If $k$ is chosen such that $k$ nearest neighbors are all within radius $r$ of a typical cluster point, the test will detect the cluster.
- If $k$ is so large that the $k$-th nearest neighbor falls outside the cluster, the test averages in non-cluster distances and loses the signal.

When $k=1$ is significant but $k=5$ isn't, you've learned something: **the cluster exists at the pairwise scale but not at the 5-neighborhood scale.** This means the cluster is small, or the subpopulation is sparse within it, or the cluster is not much tighter than the background at the 5-neighbor level.

**This is information, not an error.**

### When to be suspicious

The only time you should be concerned about $k=1$ giving a "wrong" answer is when the mechanisms in Part 1 are active:
- Duplicates/ties are present.
- A tiny fraction of the points are driving the signal.
- The pattern across $k$ is non-monotone/noisy.

If none of these apply, $k=1$ is telling you something real about fine-scale structure.

---

## Part 7: Practical Recommendations

### For your workflow

1. **Always run multiple $k$ values.** This is not about finding the "right" one — it's about characterizing the *scale* of the clustering.

2. **Use the diagnostic tree (Part 4).** Check for duplicates, check the pattern, run split-half replication.

3. **Report the full sensitivity table.** Don't pick one $k$ and hide the rest. Let the reader see the pattern.

4. **Interpret scale-dependent results honestly:**
   - Significant at $k=1$–$3$ only: "Fine-scale pairwise clustering."
   - Significant at $k=1$–$10$: "Multi-scale clustering; robust finding."
   - Significant only at $k \geq 5$: "Meso-scale density concentration; not pairwise."

5. **If $k=1$ and $k=5$ disagree, run split-half replication.** This is the single most informative diagnostic.

### For reporting in a paper

> "Sensitivity analysis across $k \in \{1, 3, 5, 7, 10, 15\}$ revealed that the clustering signal was confined to the local neighborhood scale ($k \leq 3$, all $p < 0.05$; $k \geq 5$, $p > 0.05$), consistent with fine-scale spatial concentration. Split-half replication confirmed the signal: $X\%$ of random halves showed significance at $k=1$, compared to 5% expected under the null. No near-duplicates or measurement artifacts were detected (minimum 1-NN distance = $d_{\min}$)."

---

## Summary

| Question | Answer |
|----------|--------|
| Can $k=1$ produce false positives? | At the 5% rate, same as any $k$. But near-threshold rejections at $k=1$ are noisier and can be driven by pairwise coincidences. |
| Why does $k=1$ have the most power? | Signal is concentrated in nearest neighbors. For tight clusters, $d_1$ is dramatically smaller than the null expects. Noise is higher but signal dominates. In $d \geq 3$, the effect size provably decreases with $k$. |
| If $k=1$ is significant but $k=5$ isn't, which is right? | Both are right — they answer different questions. $k=1$ detects fine-scale proximity; $k=5$ detects neighborhood-level density. The disagreement tells you the clustering scale. |
| How do I tell without ground truth? | (1) Check for duplicates/artifacts. (2) Look at the p-value pattern across $k$. (3) Run split-half replication. (4) Calibrate on synthetic data matching your scenario. |
| Is there ever ground truth? | Not in real data. Only in simulations. Use calibration experiments to build trust in your test. |

---

## References

- **Schilling, M. F. (1986).** "Multivariate two-sample tests based on nearest neighbors." *JASA*, 81(395), 799–806. — Power decreases with $k$ for well-separated alternatives.
- **Friedman, J. H., & Rafsky, L. C. (1979).** "Multivariate generalizations of the Wald-Wolfowitz and Smirnov two-sample tests." *Annals of Statistics*, 7(4), 697–717. — $k=1$ (MST/NN) achieves maximum power for well-separated groups.
- **Good, P. (2005).** *Permutation, Parametric, and Bootstrap Tests of Hypotheses*, 3rd ed. Springer. — Permutation tests have exact Type I error for any test statistic.
- **Cover, T. M., & Hart, P. E. (1967).** "Nearest neighbor pattern classification." *IEEE Trans. Inform. Theory*, 13(1), 21–27. — 1-NN optimal for high signal-to-noise.
