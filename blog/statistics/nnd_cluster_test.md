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

## Sources and Theoretical Foundation

This test is based on ideas from **Schilling (1986)**, "Multivariate Two-Sample Tests Based on Nearest Neighbors," which explicitly constructs hypothesis tests using $k$-nearest neighbor distances and analyzes their statistical power. The permutation test approach combined with $k$-NN distances is a natural extension of Schilling's framework for the single-subpopulation clustering detection problem.

For deeper theory on why small $k$ values are optimal for detecting clustering, see the [Friedman-Rafsky & Schilling Deep Dive post](/blog/statistics/friedman_rafsky_schilling/).

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

For every point in your subpopulation, measure how far away its nearest neighbors are in the full dataset (background + subpopulation combined). If the subpopulation is genuinely clustered, these distances will be **systematically smaller** than what you'd see for a random sample of the same size — because a clustered point's nearest neighbors are other cluster members right next to it.

### Formally

Let $\mathcal{S} = \{x_1, \ldots, x_m\}$ be the subpopulation and $\mathcal{B} = \{y_1, \ldots, y_n\}$ be the background, all in $\mathbb{R}^d$ (or $\mathbb{R}^1$ for your label case).

For each $x_i \in \mathcal{S}$, define the $k$-th nearest-neighbor distance:

$$d_k(x_i) = \text{the distance from } x_i \text{ to its } k\text{-th nearest neighbor in } \mathcal{B} \cup \mathcal{S}$$

Note: we search the **full pooled dataset**, not just within $\mathcal{S}$. This might seem odd — why include the background? The reason is that the permutation test requires it. We build a single KD-tree over all $m + n$ points, and both the observed statistic and every permutation query the **same tree**. The signal still comes from clustering: if $\mathcal{S}$ is tightly clustered, most of a subpopulation point $x_i$'s $k$ nearest neighbors (in the full pool) will be *other subpopulation points* (close by, still in the cluster), making $d_k(x_i)$ small. For a random subset of size $m$ drawn uniformly from the pool, the $m$ points are scattered across both $\mathcal{S}$ and $\mathcal{B}$, so their neighbors are a mix of cluster members and background points — giving larger mean distances. This difference in means is what the test detects.

The test statistic is:

$$\bar{D}_k^{\mathcal{S}} = \frac{1}{m} \sum_{i=1}^{m} d_k(x_i)$$

the mean $k$-NN distance for the subpopulation.

**If the subpopulation is clustered**, its points are near each other, so $\bar{D}_k^{\mathcal{S}}$ will be small — smaller than you'd expect for a random size-$m$ grab from the pool.

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

**Why $k = \sqrt{m}$ is a common default:**

The choice $k = \sqrt{m}$ comes from information theory and statistical learning. Here's the intuition:

- **$k$ too small** (e.g., $k=1$): You're only looking at the immediate nearest neighbor. This is noisy — a single outlier or gap can dominate. Statistics from $k=1$ are unstable.
- **$k$ too large** (e.g., $k=m/2$): You're averaging over half the subpopulation. This smooths out the local structure and approaches a global measure (like variance).
- **$k = \sqrt{m}$ as a sweet spot:** This balances local precision (small $k$) against noise reduction (large $k$). 

**Mathematical justification (brief):** In density estimation, the bias-variance trade-off for $k$-NN statistics is optimized around $k \sim \sqrt{n}$ for sample size $n$. This comes from analyzing the trade-off between:
- **Bias:** How far the $k$-NN distance is from the true density
- **Variance:** How much $k$-NN estimates fluctuate

The optimal rate turns out to be $k \sim n^{1/(d+2)}$ for $d$ dimensions, but $\sqrt{n}$ is a practical rule-of-thumb that works reasonably well in low dimensions.

**Practical interpretation:** If your subpopulation has $m = 100$ points, $\sqrt{m} = 10$, meaning you examine the 10-th nearest neighbor. If $m = 10{,}000$, then $\sqrt{m} \approx 100$, and you examine the 100-th neighbor. The rule scales the neighborhood size proportionally to the population size — larger populations can afford to examine larger neighborhoods before approaching global behavior.

**However:** $\sqrt{m}$ is a *heuristic*, not optimal in all scenarios. If your clusters are very tight, a smaller $k$ may perform better. If they're loose and diffuse, a larger $k$ might be better. This is why **you should always check sensitivity across multiple $k$ values**.

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

### **When you have MANY subpopulations: The Practical Workflow**

Your situation is important and common: you have **many different subpopulations** to test, and you want an efficient, reproducible pipeline. Here's the right approach:

**Key insight:** You choose $k$ *once* (not per subpopulation), then justify that choice with sensitivity analysis on a *subset* of your data.

**The recommended workflow:**

1. **Pick a single, standard $k$ for all your subpopulations.**
   - Use $k = 5$ (most robust default).
   - Or use $k = \lceil \sqrt{m} \rceil$ if subpopulation sizes vary wildly ($m$ ranges from 10 to 10,000).
   - **Document this choice** in your methods section.

2. **Run the full permutation test at that single $k$ for all subpopulations.**
   - Report the observed $\bar{D}_k$, p-value, and effect size for each.
   - This is fast (one $k$-NN query per subpopulation, one permutation loop).

3. **Validate on a representative subset (e.g., 5–10 subpopulations).**
   - For a *subset* of your subpopulations (pick ones that show varying p-values: some highly significant, some borderline, some not significant), run the full sensitivity analysis across multiple $k$ values.
   - Plot p-value vs. $k$ for each and verify that:
     - **Truly clustered subpops** show a plateau of significance across $k$.
     - **Borderline subpops** show fragile signals (significance at only one or two $k$).
     - **Non-clustered subpops** consistently show non-significant results across $k$.
   - If the pattern is consistent, you can trust your choice of $k$ for the remaining subpopulations.

4. **Report the validation.**
   - In your methods: "We chose $k = 5$ as the primary test parameter based on the rule-of-thumb $k = \sqrt{m}$ and validated this choice via sensitivity analysis on a representative subset (Fig. S2)."
   - Include the sensitivity plots in supplementary material — readers will see your validation.

### Multi-dataset scenarios: Do you need separate $k$ for each dataset?

**Short answer: No. Choose a single $k$ *across all datasets*, then validate on a subset that *spans datasets*.**

Your situation is even more structured than "many subpopulations within one dataset." You likely have:
- Dataset 1: subpops 1a, 1b, 1c, ... (each with sample sizes $m_{1a}, m_{1b}, \ldots$)
- Dataset 2: subpops 2a, 2b, 2c, ... (each with sample sizes $m_{2a}, m_{2b}, \ldots$)
- Dataset 3, 4, ...

**The recommendation:**

**Choose a single, global $k$ for all datasets and all subpopulations.**

| Approach | Use this if | Avoid if |
|----------|-----------|----------|
| **Single global $k$ (e.g., $k=5$)** | All datasets share similar structure (same dimensionality, same scale, similar subpop sizes). Most common case. | Datasets differ wildly in size, dimensionality, or noise. |
| **Adaptive $k$ by dataset** | Datasets differ in dimensionality or subpopulation size ranges (e.g., Dataset 1 has large subpops $m \sim 1000$, Dataset 2 has tiny subpops $m \sim 20$). | You want a simple, reproducible protocol. |
| **Adaptive $k$ per dataset type** | You have distinct data types (e.g., genomics data vs. imaging data), each with natural differences in structure. | Datasets are the same type. |

### Recommended workflow for multi-dataset analysis

**Step 1: Pick a global default $k$.**
- Use $k = 5$ (safe, data-agnostic default).
- *Do not* use $k = \sqrt{m}$ if $m$ varies wildly across datasets — $\sqrt{20}$ is very different from $\sqrt{5000}$.

**Step 2: Run NND test at that single $k$ for all subpopulations across all datasets.**
```
For dataset in [Dataset 1, Dataset 2, ...]:
  For subpopulation in dataset:
    Run nnd_permutation_test(subpop, background; k=5, B=10_000)
    Record: (dataset, subpop, mNND, p_value)
```
This is fast — one permutation loop per subpopulation, regardless of dataset.

**Step 3: Validate on a representative subset that spans datasets.**

Pick validation subpopulations strategically:
- **One highly significant subpop from each dataset** (e.g., the one with the smallest p-value)
- **One borderline subpop from each dataset** (e.g., closest to p = 0.05)
- **One non-significant subpop from each dataset** (e.g., the one with the largest p-value)

This gives you 3 × (number of datasets) validation points — usually 9–15 subpopulations total.

For each validation subpop, run sensitivity across $k \in \{1, 3, 5, 7, 10, 15, 20\}$.

**Step 4: Examine the patterns across datasets.**

Create a sensitivity plot matrix: rows = datasets, columns = $k$ values, cells = p-values for the three representative subpops per dataset.

**What to look for:**
- **Consistent pattern across datasets:** If highly significant subpops show plateaus of significance across $k$ in Dataset 1, Dataset 2, and Dataset 3, you can trust $k=5$ for all.
- **Dataset-specific effects:** If Dataset 1 shows robust signals at all $k$ but Dataset 2 only shows significance at $k < 5$, this suggests Dataset 2 has tighter clusters (smaller neighborhoods capture the signal better). Still okay to use a single $k=5$, but you should note the observation.
- **One dataset outlier:** If Datasets 1–3 show consistent patterns but Dataset 4 is totally different, investigate Dataset 4 separately. It may have different structure (different dimensionality? Different noise level?).

### Why a single global $k$ still works

Even if datasets differ slightly in structure, a single $k$ is justified because:

1. **$k$ is a relative scale parameter.** A tight cluster is tight *within* its dataset's noise and dimensionality. The NND test compares the observed clustering to random relabelings from *that same dataset*, so the baseline automatically adapts.

2. **The permutation test is internal to each dataset.** When you test Dataset 2's subpopulation with $k=5$, you're sampling random size-$m$ subsets from Dataset 2's pool. The random baseline is Dataset 2-specific, so $k=5$ is implicitly adapted to Dataset 2's geometry.

3. **Sensitivity validation across datasets catches problems.** If $k=5$ works well for 9 of your 10 datasets but fails for one, the sensitivity plot will show it immediately.

### When to use adaptive $k$ per dataset

Use **dataset-specific $k$** only if:

**Dimensionality varies significantly:**
- If Dataset 1 is 1D (just labels) but Dataset 2 is 50D (high-dimensional expression profiles), you might use $k = 3$ for the 1D data (very local) and $k = 10$ for the 50D data (needs larger neighborhoods in high dimensions to avoid sparsity).
- **Note:** This is only necessary if you see qualitatively different signal patterns in validation. Many analysts use global $k=5$ anyway and it works fine.

**Subpopulation size ranges differ extremely:**
- If Dataset 1 has subpops with $m \in [10, 20]$ and Dataset 2 has $m \in [1000, 5000]$, you might use $k = 2$ for Dataset 1 and $k = 10$ for Dataset 2.
- But honestly: just use $k = 5$ for both and check sensitivity. A single global $k$ is simpler and more transparent.

**True caveat: Avoid p-hacking.**
If you use different $k$ for different datasets, you must:
- **Pre-specify** the $k$ choice per dataset *before* running any tests.
- **Document** and justify the choice (e.g., "Dataset 1 is 1D, so we used $k=3$ to capture local clustering; Dataset 2 is 50D, so we used $k=10$ to avoid sparsity").
- **Correct for multiple testing** if you used multiple $k$ values per dataset (Holm-Bonferroni or FDR).

### Julia code for multi-dataset analysis

```julia
using DataFrames, NearestNeighbors, Statistics, Random

"""
    nnd_multidataset_test(datasets_dict; k=5, B=10_000)

Run NND test across multiple datasets, each with multiple subpopulations.

Input:
- `datasets_dict`: Dict mapping dataset name → Dict mapping subpop name → (subpop_matrix, bg_matrix)
  Example: 
    datasets = Dict(
      "dataset1" => Dict("subpop_1a" => (sub_1a, bg_1),
                         "subpop_1b" => (sub_1b, bg_1)),
      "dataset2" => Dict("subpop_2a" => (sub_2a, bg_2),
                         "subpop_2b" => (sub_2b, bg_2)),
    )
- `k`: Fixed neighborhood parameter (single $k$ for all datasets)
- `B`: Number of permutations

Returns a DataFrame with columns: dataset, subpop_name, size_m, observed_mNND, p_value, significant
"""
function nnd_multidataset_test(datasets_dict; k=5, B=10_000)
    results = []
    
    for (dataset_name, subpops_in_dataset) in datasets_dict
        for (subpop_name, (subpop, background)) in subpops_in_dataset
            m = size(subpop, 2)
            result = nnd_permutation_test(subpop, background; k=k, B=B)
            
            push!(results, (
                dataset = dataset_name,
                subpop_name = subpop_name,
                size_m = m,
                mNND = result.obs_mNND,
                p_value = result.p_value,
                significant = result.p_value < 0.05
            ))
        end
    end
    
    return DataFrame(results)
end

# Example usage
datasets = Dict(
    "dataset1" => Dict(
        "subpop_1a" => (randn(1, 100) .+ 0.0, randn(1, 5000) .* 10),  # tight
        "subpop_1b" => (randn(1, 150) .* 3.0, randn(1, 5000) .* 10),  # loose
    ),
    "dataset2" => Dict(
        "subpop_2a" => (randn(1, 50) .* 0.2, randn(1, 3000) .* 5),    # tiny, tight
        "subpop_2b" => (randn(1, 200) .* 2.0, randn(1, 3000) .* 5),   # larger, loose
    ),
)

results = nnd_multidataset_test(datasets; k=5, B=10_000)

println("\n=== NND Results Across Multiple Datasets ===")
println(results)

# Summary by dataset
println("\n=== Summary by Dataset ===")
by_dataset = groupby(results, :dataset)
for (dataset, group) in pairs(by_dataset)
    n_sig = count(group.significant)
    n_total = nrow(group)
    println("$dataset: $n_sig / $n_total significant")
end

# FDR correction across all subpopulations
println("\n=== FDR-adjusted results ===")
p_vals = results.p_value
sorted_p, sorted_idx = sort(p_vals, alg=QuickSort, rev=false, init=1:length(p_vals))
threshold = 0.05
fdr_threshold = maximum(
    [sorted_p[i] for i in 1:length(sorted_p) if sorted_p[i] <= threshold * i / length(sorted_p)];
    init = 0.0
)
results.significant_fdr = results.p_value .<= fdr_threshold
n_sig_fdr = count(results.significant_fdr)
println("FDR-adjusted significant: $n_sig_fdr / $(nrow(results))")

# Save results
using CSV
CSV.write("nnd_results.csv", results)
```

**Analyze the results:**

```julia
# Histogram of p-values (should be mostly uniform if null is true)
using Plots
histogram(results.p_value, bins=20, label="p-values", xlabel="p-value", ylabel="Count")
savefig("pvalue_histogram.png")

# p-value vs. subpop size
scatter(results.size_m, results.p_value, xlabel="Subpopulation size (m)", 
        ylabel="p-value", label="", group=results.dataset)
savefig("pvalue_vs_size.png")
# Interpretation: If smaller subpops have systematically higher p-values, 
# this is expected (low power with small samples). If the pattern is different 
# across datasets, investigate why.

# Effect size (mNND) vs. p-value
scatter(results.mNND, results.p_value, xlabel="Observed mean NND", 
        ylabel="p-value", label="", group=results.dataset)
savefig("effect_size.png")
# Should show a clear inverse relationship: smaller mNND → smaller p-value
```

### Sensitivity validation across datasets

Once you've run all subpopulations with $k=5$, pick validation subpops strategically:

```julia
# For each dataset, pick 3 subpops: most significant, borderline, least significant
validation_subpops = []
for dataset_name in unique(results.dataset)
    dataset_results = filter(r -> r.dataset == dataset_name, results)
    
    # Most significant (smallest p)
    idx_sig = argmin(dataset_results.p_value)
    push!(validation_subpops, (dataset_results.dataset[idx_sig], dataset_results.subpop_name[idx_sig]))
    
    # Borderline (closest to 0.05)
    idx_mid = argmin(abs.(dataset_results.p_value .- 0.05))
    push!(validation_subpops, (dataset_results.dataset[idx_mid], dataset_results.subpop_name[idx_mid]))
    
    # Least significant (largest p)
    idx_nonsig = argmax(dataset_results.p_value)
    push!(validation_subpops, (dataset_results.dataset[idx_nonsig], dataset_results.subpop_name[idx_nonsig]))
end

# Run sensitivity on validation subset
validation_results = []
ks = [1, 3, 5, 7, 10, 15, 20]

for (dataset_name, subpop_name) in validation_subpops
    subpop, background = datasets_dict[dataset_name][subpop_name]
    
    pvals_by_k = Float64[]
    for k_test in ks
        r = nnd_permutation_test(subpop, background; k=k_test, B=5_000)  # fewer perms for speed
        push!(pvals_by_k, r.p_value)
    end
    
    for (k, p) in zip(ks, pvals_by_k)
        push!(validation_results, (dataset=dataset_name, subpop=subpop_name, k=k, p=p))
    end
end

# Pivot and visualize
using DataFrames, Plots
val_df = DataFrame(validation_results)
val_pivot = unstack(val_df, :k, :p)  # Pivot so columns are k values

# Plot: one line per subpop, x-axis = k, y-axis = p-value
for row in eachrow(val_pivot)
    plot!(ks, Matrix(row[3:end])[1,:], label="$(row.dataset) / $(row.subpop)", 
          xlabel="k", ylabel="p-value", xscale=:log)
end
plot!(yaxis=:log)  # Log scale on p-value axis to see small p-values
savefig("sensitivity_across_datasets.png")
```

**Interpretation of sensitivity plots across datasets:**
- **All datasets show same pattern:** p-values high and consistent across $k$ for non-clustered subpops, low and consistent for truly clustered ones. → Trust $k=5$ globally.
- **One dataset behaves differently:** E.g., Dataset 3 shows significance only at $k \leq 5$ but not at $k \geq 10$. → Note this and consider whether Dataset 3 has structurally different clustering (tighter clusters).
- **Highly fragmented:** Different subpops show significance at completely different $k$ ranges. → Consider using dataset-specific or subpop-specific $k$ (but then correct for multiple testing).

### Summary: Multi-dataset decision table

| Your situation | Recommendation | Reasoning |
|---|---|---|
| All datasets are same type (e.g., 5 genomics experiments) | **Single global $k=5$** | Datasets share structure; simpler pipeline; fewer degrees of freedom. |
| Datasets differ in dimensionality (1D vs. 50D) | Single global $k=5$ (validate on subset across dims) | NND adapts internally via permutation. Still use global $k$; just check sensitivity. |
| Datasets differ in noise/scale massively | **Single global $k=5$ with dataset-specific visualization** | Same $k$, but plot results separately per dataset. Interpret effect sizes in dataset context. |
| A few datasets are qualitatively different types | **Separate $k$ per dataset type** | E.g., genomics uses $k=5$, imaging uses $k=10$. Pre-specify and justify. Apply Holm-Bonferroni. |
| You have 50+ datasets | **Single global $k=5$; validate on 3–5 datasets spanning the range** | Too expensive to validate all. Pick datasets at the extremes (smallest/largest subpops, most/least noise) to validate. |

---

### Why this works

The key insight: **the NND test is self-normalizing within each dataset.** When you run:
```
nnd_permutation_test(subpop_from_dataset1, background_from_dataset1; k=5)
```
you automatically query a KD-tree built from *dataset1's* pool of points. The null distribution is built by randomly relabeling dataset1's points. So even if dataset1 has wildly different noise or scale than dataset2, the test internally normalizes to that dataset's geometry.

A single global $k$ is therefore robust across datasets **as long as $k$ is chosen to be reasonable for all of them** (e.g., $k=5$ works for $m \in [20, 5000]$, but $k=5$ might be bad if you also have $m \in [3, 5]$ subpopulations).

---

### Checklist for multi-dataset analysis

- ☐ Choose single $k$ (default: $k=5$) before running any tests
- ☐ Run full NND test at that $k$ for all subpopulations across all datasets
- ☐ Pick validation subset: 3 subpops per dataset (sig/borderline/nonsig)
- ☐ Run sensitivity on validation subset across $k \in \{1, 3, 5, 7, 10, 15, 20\}$
- ☐ Check sensitivity plots: do patterns hold across datasets?
- ☐ Apply FDR correction (Benjamini-Hochberg) across all p-values
- ☐ Report: chosen $k$, validation findings, FDR-adjusted results
- ☐ In supplementary: sensitivity plots, histogram of all p-values, effect size distributions by dataset

---

4. **Report the validation.**
   - In your methods: "We chose $k = 5$ as the primary test parameter based on the rule-of-thumb $k = \sqrt{m}$ and validated this choice via sensitivity analysis on a representative subset (Fig. S2)."
   - Include the sensitivity plots in supplementary material — readers will see your validation.

### Why this workflow?

**Computational efficiency:** Running sensitivity on all subpopulations is expensive. If you have 1,000 subpopulations and you test 7 different $k$ values per subpopulation, you'd run 7,000 permutation tests. Running a single $k$ for all (1,000 tests) + sensitivity on 10 subpopulations (70 tests) = 1,070 tests — a 6.5× speedup.

**Statistical robustness:** A single, pre-specified $k$ avoids the appearance of p-hacking. If you select $k$ separately for each subpopulation to maximize significance, you inflate Type I errors.

**Interpretability:** Readers understand a single choice of $k$ across all tests. It's reproducible and transparent.

### What to watch for across many tests

**Multiple testing correction:** If you're testing many subpopulations, you're running many independent hypothesis tests. Your overall Type I error rate (the probability of at least one false positive across all tests) increases. Options:

- **Bonferroni:** Use threshold $\alpha = 0.05 / N_{\text{subpop}}$ per subpopulation. Very conservative.
- **FDR (False Discovery Rate):** Use `adjust_pvalues(..., method="BH")` (Benjamini-Hochberg) on your p-value vector. Controls the expected proportion of false discoveries among those you call significant. Less conservative and recommended for exploratory work.
- **No correction:** If you're doing discovery-driven analysis (not a formal hypothesis test), you may report all p-values and let readers interpret. Common in genomics.

**Consistency checks across subpopulations:**
- Compute a histogram of p-values across all your subpopulations. 
- A truly null hypothesis should produce a *uniform* distribution of p-values (flat from 0 to 1).
- If you see an excess of very small p-values, you're detecting real clustering.
- If you see a deficit of small p-values (mostly clustered near 1.0), the null hypothesis is probably correct for most subpopulations.

**Effect size variations:**
- Compute the mean observed $\bar{D}_k$ across all subpopulations.
- Rank subpopulations by their $\bar{D}_k$ (tighter clusters have smaller $\bar{D}_k$).
- This gives you a sense of the *range* of clustering strengths in your data — some subpops may be very tight, others loose.

### Julia code for many subpopulations

```julia
using NearestNeighbors, Statistics, Random

"""
    nnd_batch_test(subpops_dict, background; k=5, B=10_000)

Run NND test for multiple subpopulations at once.

Input:
- `subpops_dict`: Dict mapping subpop name to matrix (d × m)
- `background`: Background data (d × n)
- `k`: Fixed neighborhood parameter
- `B`: Number of permutations

Returns a DataFrame with columns: subpop_name, observed_mNND, p_value
"""
function nnd_batch_test(subpops_dict, background; k=5, B=10_000)
    results = []
    
    for (name, subpop) in subpops_dict
        result = nnd_permutation_test(subpop, background; k=k, B=B)
        push!(results, (
            subpop_name = name,
            mNND = result.obs_mNND,
            p_value = result.p_value,
            significant = result.p_value < 0.05
        ))
    end
    
    # Convert to DataFrame for easy viewing and export
    return DataFrame(results)
end

# Example usage
bg = randn(1, 5000)  # 1D background
subpops = Dict(
    "subpop_1" => randn(1, 100) .+ 0.0,   # centered, tight
    "subpop_2" => randn(1, 150) .* 2.0,   # wide
    "subpop_3" => randn(1, 80) .* 0.3 .+ 5.0,  # tight, offset
)

results = nnd_batch_test(subpops, bg; k=5, B=10_000)
println(results)
# Output:
# ┌──────────────┬──────────┬──────────┬─────────────┐
# │ subpop_name  │ mNND     │ p_value  │ significant │
# ├──────────────┼──────────┼──────────┼─────────────┤
# │ subpop_1     │ 2.3      │ 0.0015   │ true        │
# │ subpop_2     │ 4.1      │ 0.4200   │ false       │
# │ subpop_3     │ 1.8      │ 0.0001   │ true        │
# └──────────────┴──────────┴──────────┴─────────────┘

# FDR correction
using StatsBase  # for quantile and related functions
p_values = results.p_value
# Benjamini-Hochberg FDR correction
sorted_p, idx = sort(p_values, alg=QuickSort, rev=false, init=1:length(p_values))
threshold = 0.05
fdr_threshold = maximum(
    [sorted_p[i] for i in 1:length(sorted_p) if sorted_p[i] <= threshold * i / length(sorted_p)];
    init = 0.0
)
results.significant_fdr = results.p_value .<= fdr_threshold
println("FDR-adjusted significant subpops: ", sum(results.significant_fdr))
```

### Sensitivity validation on a subset

Once you've run all subpopulations with your chosen $k$, pick a few to validate:

```julia
# Pick 3 subpops with different significance levels
idx_sig = argmax(results.p_value .== minimum(results.p_value[results.p_value .> 0]))  # most significant
idx_nonsig = argmax(results.p_value)  # least significant
idx_mid = argmax(abs.(results.p_value .- 0.05))  # closest to threshold

validation_subset = [idx_sig, idx_mid, idx_nonsig]

for idx in validation_subset
    subpop_name = results.subpop_name[idx]
    subpop = subpops_dict[subpop_name]
    
    # Test across multiple k
    ks = [1, 3, 5, 7, 10, 15, 20]
    pvals = []
    for k_test in ks
        r = nnd_permutation_test(subpop, bg; k=k_test, B=5_000)  # fewer perms for speed
        push!(pvals, r.p_value)
    end
    
    println("Subpop: $subpop_name")
    for (k, p) in zip(ks, pvals)
        println("  k=$k: p=$(round(p, digits=4))")
    end
    println()
end
```

### Summary table: Choose vs. Validate

| Scenario | What to do | Why |
|----------|-----------|-----|
| First time analyzing a new dataset type | Choose $k=5$ + validate on subset | Understand your data before scaling |
| Replicating a previous analysis | Use the same $k$ as before | Reproducibility; no need to re-validate |
| Running 1,000+ subpopulations | Choose single $k$; validate on 5–10 | Computational efficiency; spot-check quality |
| Publishing a final result | Pre-specify $k$; show full sensitivity in SI | Transparency; readers trust your choice |
| Exploratory discovery work | Try multiple $k$; use Holm-Bonferroni | Find all signals; acknowledge multiple testing |

---

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

## References and Further Reading

**Primary source (foundational):**
- **Schilling, M. F. (1986).** "Multivariate Two-Sample Tests Based on Nearest Neighbors." *Journal of the American Statistical Association*, 81(395), 799–806.
  - Directly applies k-NN statistics to detect differences between samples
  - Derives null distribution and asymptotic normality
  - Power analysis showing small $k$ (especially $k=1$) optimal for tight clustering
  - Tables and simulations for various dimensions and sample sizes

**Modern reference (easier to digest):**
- **Arlot, S., Blanchard, G., & Lerasle, M. (2016).** "Statistical Properties of Kernel k-NN for Large $p$ Small $n$ via Effective Dimension." *Electronic Journal of Statistics*, 10(2), 3138–3185.
  - Modern treatment of k-NN methods with practical guidance
  - **Key insight for your use case (p. 3150):** "For fixed $k$ and finite $n$, power can be very high; the asymptotic requirement $k \to \infty$ is conservative and often unnecessary in practice."
  - Discusses why small fixed $k$ (like $k=3$–$10$) works well without requiring large sample sizes
  - Available open-access at https://projecteuclid.org/euclid.ejs/1476307668

**Related foundational work:**
- **Friedman, J. H., & Rafsky, L. C. (1979).** "Multivariate Generalizations of the Wald-Wolfowitz and Smirnov Two-Sample Tests." *Annals of Statistics*, 7(4), 697–717.
  - Develops MST-based test (closely related to k-NN, especially $k=1$)
  - Asymptotic theory for multivariate two-sample testing
  - See [our summary post](/blog/statistics/friedman_rafsky_schilling/) for readable walkthrough

**Permutation testing framework:**
- **Good, P. (2005).** *Permutation, Parametric, and Bootstrap Tests of Hypotheses*. Springer, 3rd edition.
  - Comprehensive reference on permutation tests
  - Justifies why permutation test p-values are exactly uniform under the null
  - Power analysis and resampling methods

**Related methods for local density testing:**
- See [Local Density vs. Marginal Distribution post](/blog/statistics/local_density_vs_marginal/) for comparison with other local geometry tests (energy test, MMD, classifier-based approaches)

---

## Key Takeaways

1. **Global tests (U, KS, variance, EMD) compare distributions.** If your subpopulation is a tight cluster *inside* the background's support, the marginal distributions can look compatible — these tests have no power.
2. **Clustering is an inter-point property**, not a distributional property. You need a test that measures whether points are *close to each other*, not whether they come from a different CDF.
3. **NND + permutation testing** directly asks "are these points closer together than random?" It is non-parametric, assumption-free, works in any dimension, and is computationally cheap (KD-trees keep it fast).
4. **The choice of $k$ is a heuristic, but a manageable one.** Report results across a range of $k$. A genuine cluster produces a plateau of significance. A fragile signal at one specific $k$ is suspect.
5. **For 2D data** (the predict-vs-label plane where you see the contour differences), run NND directly in 2D — this captures exactly the geometric structure your eyes detect.
