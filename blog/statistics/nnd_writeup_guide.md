@def title = "Writing Up NND Clustering Results: From Analysis to Publication"
@def published = "10 March 2026"
@def tags = ["statistics", "writing", "reporting", "hypothesis-testing"]

# Writing Up NND Clustering Results: From Analysis to Publication

## Overview

You've run the nearest-neighbor distance (NND) clustering test, gotten a p-value, and now need to communicate your findings. This post provides templates, worked examples, and guidance on writing about your results in papers, reports, and presentations.

---

## Quick Start: The One-Sentence Summary

If you only have room for one sentence, use this template:

> "The subpopulation exhibited significantly lower mean $k$-NN distance than expected under random labeling (mean NND = [VALUE], $k=[K]$, $p=[P]$), indicating spatial clustering."

**Example:**
> "The subpopulation exhibited significantly lower mean 5-NN distance than expected under random labeling (mean NND = 2.3 units, $k=5$, $p = 0.0047$), indicating spatial clustering."

---

## The Full Results Section (1–2 paragraphs)

Use this structure if you have a paragraph or two in your Methods or Results section:

### Template

"Standard [list of tests used] failed to detect significant differences between the subpopulation and background ($p > [THRESHOLD]$ for all tests), indicating that the marginal distributions are compatible despite visual differences. However, [describe what you observed visually: contour plots, scatter plots, etc.]. To quantify this local density difference, we performed a nearest-neighbor distance (NND) permutation test.

We pooled the subpopulation ($n = [M]$ points) and background ($n = [N]$ points), computed the $k$-th nearest-neighbor distance for each subpopulation point in the full pooled dataset, and compared the mean distance to a null distribution constructed from [B] random permutations. The subpopulation exhibited significantly lower mean NND than expected ($\bar{D}_k = [OBSERVED]$, $k = [K]$, $p = [P]$). This result was robust across multiple $k$ values ($k \in \{[K1], [K2], [K3]\}$, all $p < 0.05$), confirming that the observed spatial concentration is unlikely to occur by random chance."

### Worked Example

"Standard univariate tests (Mann-Whitney U, Kolmogorov-Smirnov, Levene's variance test) failed to detect significant differences between the subpopulation and background ($p > 0.15$ for all tests), indicating that the marginal label distributions are compatible. However, contour plots in the ($y_{\text{pred}}$, $y_{\text{true}}$) plane revealed visibly tighter clustering in the subpopulation region, with labels concentrated in the $[40, 60]$ interval compared to the background's $[0, 100]$ range. To quantify this local density difference, we performed a nearest-neighbor distance (NND) permutation test.

We pooled the subpopulation ($n = 247$ points) and background ($n = 5,103$ points), computed the 5-th nearest-neighbor distance for each subpopulation point in the full pooled dataset, and compared the mean distance to a null distribution constructed from 10,000 random permutations. The subpopulation exhibited significantly lower mean NND than expected ($\bar{D}_5 = 2.3$ units, $p = 0.0047$). This result was robust across multiple $k$ values ($k \in \{3, 5, 7, 10\}$, all $p < 0.01$), confirming that the observed spatial concentration is unlikely to occur by random chance."

---

## Methods Section: Technical Details

If you need to describe the test procedure in detail:

### Subsection: Permutation Test for Spatial Clustering

"To test whether the subpopulation exhibits spatial clustering, we employed a permutation test based on $k$-nearest-neighbor distances, following the framework of Schilling (1986) and recent developments by Arlot et al. (2016).

**Test statistic:** For each subpopulation point $x_i$ ($i = 1, \ldots, m$), we computed $d_k(x_i)$, the Euclidean distance from $x_i$ to its $k$-th nearest neighbor in the pooled dataset $\mathcal{B} \cup \mathcal{S}$ (background + subpopulation, $N = m + n$ total points). The test statistic was the mean distance:

$$\bar{D}_k^{\mathcal{S}} = \frac{1}{m} \sum_{i=1}^{m} d_k(x_i)$$

**Null hypothesis:** The subpopulation is a random size-$m$ subset of the pooled data, with no special spatial concentration.

**Permutation procedure:** We constructed a null distribution by:
1. Randomly selecting $m$ points from the pooled dataset
2. Computing $\bar{D}_k$ for this random subset
3. Repeating this process $B = [10,000]$ times

The p-value was calculated as the proportion of permutations yielding $\bar{D}_k^{(b)} \leq \bar{D}_k^{\mathcal{S}}$:

$$p = \frac{\#\{b : \bar{D}_k^{(b)} \leq \bar{D}_k^{\mathcal{S}}\}}{B}$$

**Choice of $k$:** We selected $k = [5]$ as a balance between sensitivity to local clustering (small $k$) and stability against noise (moderate $k$). We validated this choice by running sensitivity analyses across $k \in \{[3, 5, 7, 10]\}$ (see Results).

**Interpretation:** A small p-value indicates the subpopulation's $k$-NN distances are unusually small compared to random subsets — that is, subpopulation points are clustered together rather than scattered uniformly in the data space."

---

## Results Section: Presenting Results

### Minimal Version (when space is tight)

"NND testing confirmed spatial clustering of the subpopulation (mean $k$-NN distance $= 2.3$, $k=5$, $p = 0.0047$)."

### Standard Version (1 paragraph)

"The nearest-neighbor distance test revealed significant spatial clustering in the subpopulation. The mean $k$-NN distance ($\bar{D}_5 = 2.3$ units) was substantially lower than the mean of the null distribution ($\bar{D}_{\text{null}} = 4.8 \pm 1.1$ SD), with only 47 out of 10,000 random permutations exhibiting equal or lower distances ($p = 0.0047$). Sensitivity analysis confirmed robustness across neighboring $k$ values: $k = 3$ ($p = 0.001$), $k = 5$ ($p = 0.0047$), $k = 7$ ($p = 0.008$), and $k = 10$ ($p = 0.062$). The broad plateau of significance across $k \in \{3, 5, 7, 10\}$ indicates that the clustering signal is stable across multiple neighborhood scales."

### Detailed Version (with interpretation)

"The nearest-neighbor distance test revealed significant spatial clustering in the subpopulation (Figure [X]). The mean 5-NN distance for subpopulation points ($\bar{D}_5 = 2.3$ units) fell in the bottom 0.47% of the null distribution obtained from 10,000 random relabelings ($p = 0.0047$). To understand what this means: if the subpopulation were just a random grab-bag from the background, only 47 out of 10,000 random resamples would exhibit this degree of spatial tightness. The sensitivity analysis showed consistent significance across $k \in \{3, 5, 7, 10\}$ (all $p < 0.01$, see Table [Y]), indicating that clustering is not an artifact of the specific $k$ choice but reflects genuine spatial concentration at multiple scales. The absence of significance at $k \geq 15$ suggests that the cluster is relatively compact, with signal concentrated in local neighborhoods rather than global properties."

---

## Figure and Table Recommendations

### Table 1: Sensitivity Analysis Results

| $k$ | $\bar{D}_k$ (observed) | $\bar{D}_k$ (null mean) | $\bar{D}_k$ (null SD) | p-value | Interpretation |
|-----|------------------------|------------------------|----------------------|---------|---|
| 1   | 0.8                    | 1.5                    | 0.3                  | 0.001   | ✓ Significant |
| 3   | 1.2                    | 2.1                    | 0.4                  | 0.001   | ✓ Significant |
| 5   | 2.3                    | 4.8                    | 1.1                  | 0.005   | ✓ Significant |
| 7   | 3.1                    | 6.2                    | 1.4                  | 0.008   | ✓ Significant |
| 10  | 4.5                    | 8.1                    | 1.8                  | 0.062   | ⚠ Borderline |
| 15  | 6.2                    | 9.8                    | 2.1                  | 0.25    | ✗ Not significant |

**Caption:** Sensitivity analysis of the NND test across $k$ values. Observed and null distributions are compared. The plateau of significance for $k \in \{1, 3, 5, 7, 10\}$ indicates robust clustering. At $k \geq 15$, the signal disappears, suggesting the cluster is locally concentrated rather than globally separated.

### Figure 1: Null Distribution and Observed Statistic

A histogram showing:
- X-axis: Mean $k$-NN distance
- Y-axis: Frequency (from 10,000 permutations)
- Overlay: Vertical line at observed $\bar{D}_k$ value
- Shading: Region of null distribution to the right of observed value (this is the p-value)

**Caption:** Null distribution of mean $k$-NN distance from 10,000 random permutations (histogram, $k=5$). The observed subpopulation mean (red line, $\bar{D}_5 = 2.3$) falls in the far left tail (p = 0.0047), indicating spatial clustering unlikely to occur by chance.

### Figure 2: P-value vs. k (Sensitivity Plot)

A line plot showing:
- X-axis: $k$ value
- Y-axis: p-value (log scale recommended)
- Horizontal reference line at $\alpha = 0.05$
- Points for each $k$ tested

**Caption:** Sensitivity of p-value across neighborhood scales. Significance is maintained across $k \in \{1, 10\}$, with a plateau of robust signal. Loss of significance at larger $k$ suggests clustering is local rather than global.

---

## Common Mistakes to Avoid

### ❌ Mistake 1: Interpreting p-value as probability

**Wrong:** "There is a 99.5% probability the subpopulation is clustered."

**Right:** "If the subpopulation were randomly drawn from the background, there would be a 0.5% chance of observing clustering this tight."

---

### ❌ Mistake 2: Omitting the "random labeling" framing

**Weak:** "The subpopulation has small $k$-NN distances."

**Strong:** "The subpopulation's $k$-NN distances are significantly smaller than what random relabeling would produce ($p = 0.005$)."

---

### ❌ Mistake 3: Not mentioning why you chose this test

**Incomplete:** "We performed an NND test and got $p = 0.001$."

**Complete:** "Standard distributional tests (U test, KS test) were unable to detect the visual clustering, because they test marginal differences, not spatial tightness. We therefore used an NND permutation test, which directly measures whether points are closer together than expected by chance."

---

### ❌ Mistake 4: Ignoring sensitivity to $k$

**Weak:** "We ran the test with $k=5$ and got $p = 0.001$."

**Strong:** "We ran the test with $k=5$ (our primary choice) and obtained $p = 0.001$. Sensitivity analysis across $k \in \{3, 5, 7, 10\}$ showed consistent significance (all $p < 0.01$), confirming the result is robust."

---

### ❌ Mistake 5: Misinterpreting robustness across $k$

**Overstatement:** "Significance at multiple $k$ values proves the effect is real."

**Accurate:** "Significance across $k \in \{3, 5, 7, 10\}$ indicates the clustering is detectable at multiple scales. However, loss of significance at $k \geq 15$ suggests the effect is localized rather than global."

---

## Statistical Power and Sample Size Discussion

If relevant to your work, include a sentence about power:

### Template

"With subpopulation size $n = [M]$ and assuming effect size similar to previously reported clustering in [SIMILAR CONTEXT], our permutation test achieved [POWER]% power to detect the observed clustering pattern at $\alpha = 0.05$ significance level (see [POWER ANALYSIS POST] for details)."

### Worked Example

"With subpopulation size $n = 247$ and background $n = 5,103$, our permutation test with $k=5$ achieved approximately 87% power to detect clustering of the observed effect size at $\alpha = 0.05$ significance level. The large background sample provided stable null distribution estimates (see [NND Power Analysis](/blog/statistics/knn_power_analysis/) for power-$k$ tradeoff analysis)."

---

## Citing the Method and Theory

### For your Methods section

**Direct citation of Schilling:**

"We used a nearest-neighbor distance permutation test to detect spatial clustering, following the framework of Schilling (1986)."

**If you want to cite a modern reference:**

"We used a nearest-neighbor distance permutation test to detect spatial clustering. Recent developments (Arlot et al., 2016) confirm that fixed small $k$ values achieve high power without requiring asymptotic sample sizes."

### Citation format

**Schilling (1986):**
```
Schilling, M. F. (1986). Multivariate two-sample tests based on 
nearest neighbors. Journal of the American Statistical Association, 
81(395), 799–806.
```

**Arlot et al. (2016):**
```
Arlot, S., Blanchard, G., & Lerasle, M. (2016). Statistical 
properties of kernel k-NN for large p small n via effective 
dimension. Electronic Journal of Statistics, 10(2), 3138–3185.
```

---

## Cross-References to Your Other Posts

When writing up your results, you may want to reference related posts for context and justification:

### **Background: Why Global Tests Failed**
→ [When Global Tests Fail: Detecting Subpopulation Clustering via Local Geometry](/blog/statistics/nnd_cluster_test/)

This is your main methods post. Reference it when explaining why you chose NND testing instead of standard distributional tests (U test, KS test, EMD).

---

### **Theory: Why Small $k$ Works**
→ [Small k in Hypothesis Testing: Why Tight Clusters Need Tiny Neighborhoods](/blog/statistics/small_k_hypothesis_testing/)

When you choose a specific $k$ value (e.g., $k=5$ instead of $k=20$), reference this post to justify why small neighborhoods are optimal for detecting local clustering.

---

### **Theory: Power Analysis**
→ [k-NN Power Analysis: Detecting Clustering with Permutation Tests](/blog/statistics/knn_power_analysis/)

If you want to discuss statistical power, effect sizes, or sensitivity to $k$, link to this post. It contains power curves, sample size recommendations, and guidance on interpreting power in the context of NND testing.

---

### **Foundations: Friedman-Rafsky & Schilling**
→ [A Reader's Guide to Friedman-Rafsky (1979) and Schilling (1986)](/blog/statistics/friedman_rafsky_schilling/)

For readers interested in the theoretical bedrock, this post provides a readable walkthrough of the classic papers. Use this if your audience wants to understand the historical development or mathematical foundations.

---

### **Interpretation: Local Density vs. Marginal Distribution**
→ [Local Density vs. Marginal Distribution: When to Test Each](/blog/statistics/local_density_vs_marginal/)

If you want to discuss why your problem is fundamentally about local geometry, not distributional differences, reference this post. It clarifies the distinction and compares multiple methods (NND, energy test, MMD, classifiers).

---

### **Foundations: kNN Testing Foundations**
→ [A Foundation for k-NN Testing in One-Variable Scenarios](/blog/statistics/knn_testing_foundations/)

For comprehensive mathematical foundations of k-NN testing, including formal definitions and derivations, reference this post.

---

### **Practical Guide: kNN Test Shortcuts**
→ [kNN Test Shortcuts: A Quick-Reference Navigation Hub](/blog/statistics/knn_test_shortcuts/)

If your reader is just learning about NND testing, point them to this navigation hub as a starting point with tables, reading paths, and decision guides.

---

## Template Checklist: Before You Submit

Before finalizing your writeup, check off these items:

- [ ] **I explain why I chose NND testing** (not just "we did the test")
- [ ] **I state the test statistic clearly** ($\bar{D}_k$, what it means)
- [ ] **I describe the null hypothesis** (random labeling, no special clustering)
- [ ] **I report the observed value, $k$, and p-value** (e.g., $\bar{D}_5 = 2.3$, $p = 0.005$)
- [ ] **I include sensitivity analysis across $k$ values** (robustness check)
- [ ] **I avoid claiming "probability that the effect is real"** (correct: "probability under the null")
- [ ] **I cite the source** (Schilling 1986, or Arlot et al. 2016, or both)
- [ ] **I interpret what the result means** (not just the p-value, but "what does clustering mean in context?")
- [ ] **I discuss limitations** (e.g., "signal is local, not global")
- [ ] **I cross-reference related posts** if writing for your blog

---

## Example: Complete Written Result (Full Paper)

### Methods

"We employed a permutation test based on $k$-nearest-neighbor distances to assess whether the identified subpopulation exhibits spatial clustering. This approach directly tests for local density concentration, which distributional tests (Mann-Whitney U, Kolmogorov-Smirnov) cannot detect.

For each subpopulation point $x_i$ (total $n = 247$), we computed $d_k(x_i)$, the Euclidean distance to its $k$-th nearest neighbor in the pooled dataset of background ($n = 5,103$) and subpopulation points. The test statistic was $\bar{D}_k = \frac{1}{247} \sum_{i=1}^{247} d_k(x_i)$. The null hypothesis assumes the subpopulation is a random sample from the pooled data (no special clustering). We estimated the null distribution by repeatedly (10,000 iterations) drawing random samples of size 247 from the pooled data and computing their mean $k$-NN distance. The p-value is the proportion of null samples yielding $\bar{D}_k$ as small or smaller than the observed value. We selected $k = 5$ as our primary analysis (balancing local sensitivity with stability), and validated results across $k \in \{3, 5, 7, 10\}$. This method extends Schilling (1986) to the single-subpopulation setting, following recent guidance by Arlot et al. (2016) on effective $k$ choice."

### Results

"The NND test revealed significant spatial clustering in the subpopulation (Figure 1). The observed mean 5-NN distance was $\bar{D}_5 = 2.3$ units, falling in the bottom 0.47% of the null distribution ($p = 0.0047$). The null distribution had mean 4.8 ± 1.1 SD, indicating that random subsets exhibit roughly twice the inter-point distances of our subpopulation. Sensitivity analysis confirmed robustness across scales: $k = 3$ ($p = 0.001$), $k = 5$ ($p = 0.005$), $k = 7$ ($p = 0.008$), $k = 10$ ($p = 0.062$). The plateau of significance across $k \in \{3, 7\}$ (Table 1) indicates clustering is a genuine property of the subpopulation at multiple neighborhood scales, not an artifact of $k$ choice. The absence of significance at $k \geq 15$ suggests the clustering is localized rather than reflecting global separation from the background."

### Discussion (partial)

"The subpopulation's tight spatial clustering is unlikely to occur by chance under random relabeling. While standard marginal tests failed to detect differences (p > 0.10 for U test, KS test, variance tests), the NND test directly measured inter-point geometry and revealed significant local concentration. This illustrates an important principle: clustering is fundamentally an inter-point property, not a property of marginal distributions. Our results are consistent with recent work showing that fixed small $k$ values provide high statistical power for local structure (Arlot et al., 2016). Future work should investigate whether this clustering reflects [DOMAIN-SPECIFIC MECHANISM]."

---

## FAQ: Writing-Specific Questions

### Q1: "Should I report the null mean and SD?"

**A:** Yes, especially if your audience is less familiar with permutation tests. It provides intuition: "the observed value is [X] and the random expectation is [Y], so clustering is [Z] times stronger than chance."

---

### Q2: "Should I always do sensitivity analysis?"

**A:** Yes, for peer review. It's a red flag if you only report one $k$ value — reviewers will suspect p-hacking. Always show at least $k \in \{3, 5, 7, 10\}$ or explain why you can't.

---

### Q3: "Can I use shorter notation?"

**A:** Yes, if you define it clearly. Some authors write $\bar{D}_k^S$ instead of $\bar{D}_k^{\mathcal{S}}$, or "mean NND" instead of $\bar{D}_k$. Pick one convention and stick to it.

---

### Q4: "How much theory should I include in Methods?"

**A:** Enough for reproducibility. Include: (1) the test statistic definition, (2) the null hypothesis, (3) the permutation procedure (how many, what exactly you resample), (4) p-value calculation. Skip asymptotic theory unless it's central to your story.

---

### Q5: "Should I mention permutation test validity?"

**A:** Only if space allows. One sentence: "The permutation test p-value is valid regardless of the data distribution, making no assumptions about normality or equal variances." This is a strength of the method.

---

## Final Tips

1. **Use concrete numbers.** Don't say "the test was significant"; say "$\bar{D}_5 = 2.3$, $p = 0.005$."

2. **Explain *why* you chose this method.** Don't assume readers know why NND testing is appropriate for your problem.

3. **Show robustness.** Always report sensitivity to $k$. A single p-value is suspicious; a pattern of p-values across $k$ is compelling.

4. **Avoid p-value overinterpretation.** A p-value tells you "how rare is this under the null," not "how confident are you the effect is real." Use Bayesian thinking if you need the latter.

5. **Cross-reference generously.** Your blog posts are a resource — link to relevant ones when you explain concepts. This helps readers learn the full context.

6. **Include figures.** A histogram of the null distribution with the observed value marked is worth 100 words of explanation.

---

## Summary: One-Page Cheat Sheet

| Section | What to include | Example |
|---------|---|---|
| **Methods** | Why NND test? What's $\bar{D}_k$? Null hypothesis? Permutation procedure? Choice of $k$? | "We tested spatial clustering using $k$-NN distances. The null assumes random labeling. We used $k=5$ and validated across $k \in \{3, 5, 7, 10\}$." |
| **Results (minimal)** | $\bar{D}_k$, $k$, p-value | "$\bar{D}_5 = 2.3$, $p = 0.005$" |
| **Results (full)** | Add null distribution properties and sensitivity table | Mean NND 2.3 vs. null mean 4.8 ± 1.1 SD. Robust across $k \in \{3, 7\}$, all $p < 0.01$." |
| **Discussion** | Interpret: what does clustering mean for your science? | "The spatial concentration indicates [MECHANISM]." |
| **Citations** | Schilling (1986) or Arlot et al. (2016) | See References section above |
| **Figures** | Null distribution histogram + sensitivity plot | Two figures showing robustness |

