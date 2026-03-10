@def title = "Power Analysis for k-NN Clustering Tests"
@def published = "10 March 2026"
@def tags = ["statistics", "hypothesis-testing", "nearest-neighbors", "power-analysis"]

# Power Analysis for k-NN Clustering Tests

## What is Power? The Core Idea

**Power** is your test's ability to **detect a real effect when it actually exists.**

More formally: If the null hypothesis is false (there really is clustering), power is the probability that your test will correctly reject it and declare clustering significant.

$$\text{Power} = P(\text{reject } H_0 \mid H_a \text{ is true})$$

In your clustering context:
- $H_0$: The subpopulation is just a random subset (no clustering)
- $H_a$: The subpopulation is genuinely clustered (closer together than random)

**Power answers:** *If my data really does contain a tight cluster, how likely is my test to catch it?*

---

## Two Sides of Every Test: Type I and Type II Error

To understand power, you need to see the full picture of what can go wrong in hypothesis testing.

### Type I Error: False Positive

$$\text{Type I error} = P(\text{reject } H_0 \mid H_0 \text{ is true})$$

**In words:** You declare clustering when there is none. You're wrong.

**You control this:** You set a significance level $\alpha$ (usually 0.05), which caps your false positive rate at 5%.

**In practice:** If there's no real clustering and you run the test 100 times, about 5 of those tests will incorrectly declare significance. This is acceptable because it's rare.

### Type II Error: False Negative

$$\text{Type II error} = \beta = P(\text{fail to reject } H_0 \mid H_a \text{ is true})$$

**In words:** You fail to detect clustering that is actually there. You're wrong.

**You DON'T directly control this.** But you can influence it by choosing your sample size, test statistic, and $k$.

### Power: The Complement of Type II Error

$$\text{Power} = 1 - \beta = P(\text{reject } H_0 \mid H_a \text{ is true})$$

**In words:** You correctly detect clustering when it's there.

**Example:**
- If Type II error = 0.20 (20% chance of missing a real effect), then Power = 0.80 (80% chance of detecting it).
- If Type II error = 0.05 (5% chance of missing), then Power = 0.95 (95% chance of detecting).

**Higher power is always better.** A test with 95% power is much more useful than one with 50% power.

---

## Why Power Matters in Your Clustering Test

### Scenario 1: High Power (Good)

You run your clustering test on a dataset where there's a **genuine, tight cluster**.

- With **high power (e.g., 90%):** You'll correctly detect it 90% of the time. If you run this analysis on similar datasets, your test works reliably.
- With **low power (e.g., 20%):** You'll detect it only 20% of the time. You might miss real clusters just by bad luck.

**Example from practice:**
- Dataset A has a very tight, obvious cluster (high signal).
- Dataset B has a looser, fuzzier cluster (low signal).
- Same test statistic, same sample size, but power differs dramatically.

### Scenario 2: Sample Size Trade-off

You're designing a study. You want to detect clusters of a certain size.

**Question:** How many background points do I need to confidently detect a cluster of size $m$?

**Answer:** It depends on the power you want.
- If you want 95% power: You might need $n = 1000$ background points.
- If you accept 50% power: You might get away with $n = 100$ background points.

Researchers routinely use power calculations to plan sample sizes.

### Scenario 3: Comparing k Values

You're deciding between $k=1$, $k=3$, and $k=5$ for your test.

**Question:** Which $k$ is best?

**Answer:** The one with the highest power (for the same Type I error rate).

From the previous post, we learned:
- For **tight clusters:** $k=1$ or $k=3$ has highest power.
- For **diffuse clusters:** $k=5$ or $k=\sqrt{m}$ has highest power.

Power is what justifies this choice. Small $k$ wins on power when clusters are tight.

---

## The Power Function: How Effect Size Drives Power

Power isn't a fixed number. It depends on **how different the two groups really are** — the **effect size**.

### Effect Size: Quantifying "How Different"

Recall the effect size from the previous post:

$$\delta = \frac{\mu_{\text{null}} - \mu_{\text{obs}}}{\sigma_{\text{null}}}$$

This measures: How many standard deviations apart are the observed mean $k$-NN distance and what we'd expect under the null hypothesis?

**Interpretation:**
- $\delta = 0.2$: Small effect (subtle difference). Power will be low.
- $\delta = 0.5$: Medium effect (noticeable difference). Power will be moderate.
- $\delta = 0.8$: Large effect (obvious difference). Power will be high.
- $\delta = 2.0$: Huge effect (dramatic difference). Power will be very high.

### The Power Function: Effect Size → Power

For a one-sample (or permutation) test with significance level $\alpha = 0.05$, the relationship is approximately:

$$\text{Power} \approx \Phi(\delta - z_{1-\alpha})$$

where:
- $\Phi$ is the standard normal cumulative distribution function.
- $z_{1-\alpha}$ is the critical value for significance level $\alpha$. For $\alpha = 0.05$ (one-tailed), $z_{1-\alpha} \approx 1.645$.

**In practice:**

| Effect Size $\delta$ | Power (approx) | Interpretation |
|---|---|---|
| 0.5 | 0.15 | Very weak test; likely to miss real effect |
| 1.0 | 0.52 | Slightly better than a coin flip |
| 1.5 | 0.84 | Good; will detect effect most of the time |
| 2.0 | 0.98 | Excellent; almost certain to detect |

**Key insight:** Even a large sample size can't help you if the effect size is tiny. And a small sample size might still give you high power if the effect size is huge.

---

## Computing Power in Your k-NN Test

### Step 1: Estimate the Effect Size from Data

Before running the formal test, estimate $\delta$ from your actual dataset:

$$\delta = \frac{\mu_{\text{null}} - \mu_{\text{obs}}}{\sigma_{\text{null}}}$$

**Where to get these:**

1. **$\mu_{\text{obs}}$:** Compute the mean $k$-NN distance for your subpopulation.
   ```julia
   D_k_subpop = mean([knn_distance(x_i, k) for x_i in subpopulation])
   ```

2. **$\mu_{\text{null}}$ and $\sigma_{\text{null}}$:** Run a quick permutation test (say, $B=1000$ permutations). Compute the mean and standard deviation of the permuted test statistics.
   ```julia
   null_distribution = [mean([knn_distance(x_i, k) for x_i in permuted_subset]) for b in 1:B]
   μ_null = mean(null_distribution)
   σ_null = std(null_distribution)
   ```

3. **Compute $\delta$:**
   ```julia
   δ = (μ_null - μ_obs) / σ_null
   ```

### Step 2: Convert Effect Size to Power

Once you have $\delta$, use the power formula:

$$\text{Power} \approx \Phi(\delta - 1.645)$$

(Assuming one-tailed test with $\alpha = 0.05$.)

```julia
using Distributions
power = cdf(Normal(), δ - 1.645)
```

**Example:**
- If $\delta = 2.5$, then Power $= \Phi(2.5 - 1.645) = \Phi(0.855) \approx 0.80$.
- You have 80% power to detect the clustering.

### Step 3: Interpret

- **Power > 0.80:** Great! Your test is reliable for detecting clusters of this size.
- **Power 0.50–0.80:** Moderate. You'll detect real clusters most of the time, but not always.
- **Power < 0.50:** Weak. You're likely to miss real clusters.

---

## Power for Different k Values: Practical Example

Let's work through a concrete example to see how power changes with $k$.

### Setup

- Background: 500 uniformly distributed points in $[0, 100]^2$.
- Subpopulation: 50 points clustered in $[40, 60] \times [40, 60]$ (tight cluster).
- Significance level: $\alpha = 0.05$ (one-tailed).

### Computing Effect Size for $k=1, 3, 5, 10$

Run permutations for each $k$, compute $\mu_{\text{null}}$, $\sigma_{\text{null}}$, and $\mu_{\text{obs}}$:

| $k$ | $\mu_{\text{obs}}$ | $\mu_{\text{null}}$ | $\sigma_{\text{null}}$ | $\delta$ | Power |
|---|---|---|---|---|---|
| 1 | 1.5 | 10.0 | 1.2 | 7.08 | 0.9999 |
| 3 | 2.8 | 10.0 | 1.5 | 4.80 | 0.9996 |
| 5 | 3.5 | 10.0 | 1.8 | 3.61 | 0.9993 |
| 10 | 5.0 | 10.0 | 2.5 | 2.00 | 0.98 |

**Observations:**
1. **All show high power** (>0.98) because the cluster is very tight and obvious.
2. **Power decreases with $k$**: $k=1$ has the highest power (0.9999), $k=10$ has the lowest (0.98).
3. **The difference is huge for $k=1$ vs $k=10$**: With $k=1$, you'll catch the cluster almost certainly; with $k=10$, still very likely, but slightly less so.

**Interpretation:** For this tight cluster, $k=1$ or $k=3$ is optimal from a power perspective.

---

## How Different Factors Affect Power

### Factor 1: Cluster Tightness (Effect Size)

**Tight cluster → High power. Loose cluster → Low power.**

| Cluster Type | Effect Size | Power |
|---|---|---|
| Very tight (subpop range = 5% of background) | $\delta = 5$ | >0.999 |
| Tight (subpop range = 10% of background) | $\delta = 2$ | ~0.98 |
| Moderate (subpop range = 30% of background) | $\delta = 0.8$ | ~0.76 |
| Loose (subpop range = 60% of background) | $\delta = 0.2$ | ~0.10 |

**Lesson:** You can only detect what's there. If the cluster is genuinely loose, even perfect sampling won't give you high power.

### Factor 2: Choice of k

**For tight clusters: $k$ small → Power high.**

| $k$ | Power (tight cluster) | Power (loose cluster) |
|---|---|---|
| 1 | 0.95 | 0.35 |
| 3 | 0.93 | 0.42 |
| 5 | 0.90 | 0.48 |
| 10 | 0.85 | 0.60 |

**Lesson:** Small $k$ excels for tight clusters but fails for loose ones. Large $k$ is the opposite. Choose based on your data.

### Factor 3: Sample Size

**Larger sample size → Higher power.**

Specifically, as the number of background points $n$ increases, the effect size typically grows (tighter null distribution relative to the signal), so power increases.

| Background Size $n$ | Effect Size $\delta$ | Power |
|---|---|---|
| 100 | 1.5 | 0.84 |
| 500 | 2.0 | 0.98 |
| 1000 | 2.2 | 0.99 |
| 5000 | 2.5 | 0.999 |

**Lesson:** With more background data, you can detect smaller clusters. But once $n$ is large enough, adding more background points helps less than improving the signal (tightness of the actual cluster).

### Factor 4: Subpopulation Size m

**Larger $m$ → Higher power (usually).**

More points in the subpopulation reduce sampling variability in the mean $k$-NN distance.

| Subpop Size $m$ | Effect Size $\delta$ | Power |
|---|---|---|
| 10 | 1.2 | 0.68 |
| 30 | 1.8 | 0.91 |
| 50 | 2.0 | 0.98 |
| 100 | 2.3 | 0.99 |

**Lesson:** More subjects in your subpopulation helps detect clustering.

---

## Power Analysis for Study Design

If you're **designing** a study (before collecting data), you can use power to determine sample size.

### Example: Planning a Study

**Goal:** Detect tight clusters (effect size $\delta = 1.5$) with 90% power at $\alpha = 0.05$.

**Question:** How many background points do I need?

**Answer:** Use the power formula in reverse.

Given desired power (0.90) and effect size ($\delta = 1.5$):

$$0.90 = \Phi(\delta - 1.645)$$
$$\Phi^{-1}(0.90) = 1.28 = \delta - 1.645$$
$$\delta = 2.93$$

Now, estimate what sample size gives you $\delta = 2.93$. From experience or pilot data, if you know:
- Standard deviation under null: $\sigma_{\text{null}} = 2.0$
- Mean difference: $\mu_{\text{null}} - \mu_{\text{obs}} = 5.86$ (needed to achieve $\delta = 2.93$)

Then you know what effect size corresponds to. Adjust background sample size $n$ to achieve this, typically by running simulations.

### Rule of Thumb

For clustering tests, a rough guideline:
- **Small clusters (tight, obvious):** $n = 100$–$300$ background points is often sufficient for 80%+ power.
- **Medium clusters (noticeable but not tight):** $n = 300$–$1000$.
- **Large, diffuse clusters:** $n = 1000+$ or may be impossible to detect reliably.

---

## Power and the Problem of "Underpowered" Studies

### The Warning

An **underpowered study** is one where power $< 0.80$. This is considered risky because:

1. **You're likely to miss real effects.** If there's truly a cluster in your data, you have a >20% chance of not catching it.
2. **When you do find something, it's often inflated.** In underpowered studies, positive findings tend to overestimate effect sizes (publication bias).

### How to Avoid It

1. **Estimate effect size from pilot data or prior studies.** Don't guess.
2. **Aim for power $\geq 0.80$**, ideally 0.90 or higher.
3. **Run sensitivity analysis across $k$ values.** If all show significance, clustering is robust. If only one $k$ shows significance, your power for that choice might be close to the edge.
4. **Report power in your paper.** Include a line like: *"The test achieved 0.87 power to detect clusters of this effect size."*

---

## Power vs. Sensitivity vs. Specificity

You might hear these terms used interchangeably, but they're different:

| Term | Definition | Formula |
|---|---|---|
| **Power** | Probability of rejecting $H_0$ when $H_a$ is true (true positive rate) | $1 - \beta$ |
| **Sensitivity** | Same as power in this context (fraction of true positives you catch) | $\frac{\text{TP}}{\text{TP} + \text{FN}}$ |
| **Specificity** | Probability of accepting $H_0$ when $H_0$ is true (true negative rate) | $\frac{\text{TN}}{\text{TN} + \text{FP}}$ |

**In clustering context:**
- **Power (Sensitivity):** If there's a real cluster, what fraction of the time does your test catch it?
- **Specificity:** If there's no real cluster, what fraction of the time does your test correctly reject clustering? (This is $1 - \text{Type I error}$.)

**Ideally, you want both high power and high specificity.** Your test should be good at detecting real clusters AND good at not crying wolf when there's nothing there.

---

## Practical Checklist: Power Analysis for Your Test

### Before Running the Test

- [ ] **Estimate expected effect size** from pilot data or literature.
- [ ] **Calculate required sample size** to achieve 80%+ power.
- [ ] **Document your assumptions** (cluster tightness, number of background points, subpopulation size).

### When Reporting Results

- [ ] **Report the observed effect size** $\delta$ for your primary $k$ value.
- [ ] **Report the power** you achieved (should ideally be >0.80).
- [ ] **Show sensitivity analysis** across $k$ values with power for each.
- [ ] **Interpret:** Does all $k$ values show high power (robust clustering) or just one (fragile)?

### Example Write-up

> "We detected significant clustering ($p < 0.001$, $k=3$, $\delta = 1.85$, power = 0.92). Sensitivity analysis across $k \in \{1,3,5,7,10\}$ showed consistent significance (all $p < 0.001$, all power $> 0.85$), indicating robust multi-scale clustering. The high power across multiple neighborhood scales suggests genuine, persistent local density concentration."

---

## Summary: What Power Means for Your Clustering Test

**Power is your test's reliability for detecting real clusters.**

1. **Higher power = more likely to catch real clustering when it exists.**
2. **Lower power = higher risk of missing real clusters (Type II error).**
3. **Power depends on:** Effect size (cluster tightness), $k$, sample size, and subpopulation size.
4. **For tight clusters:** Small $k$ (1, 3) has highest power.
5. **For loose clusters:** Larger $k$ (5, $\sqrt{m}$) has highest power.
6. **Aim for power $\geq 0.80$** in your studies; ideally 0.90.

**The practical implication:** When you report your clustering test result, always include:
- The observed effect size $\delta$
- The power you achieved
- Results of sensitivity analysis across $k$ values

This lets readers know: *Are you reporting a robust finding, or are you living on the edge of statistical reliability?*

If power is high across multiple $k$ values, your finding is solid. If high power only at one specific $k$, be cautious — it might be a fragile, data-dependent result.

---

## Further Reading

- **Cohen, J. (1988).** *Statistical Power Analysis for the Behavioral Sciences*, 2nd ed. Routledge.
  - Classic reference on power analysis. Chapter 1 gives intuition; see index for two-sample tests.

- **Good, P. (2005).** *Permutation, Parametric, and Bootstrap Tests of Hypotheses*, 3rd ed. Springer.
  - Section on power for permutation tests (what you're using for k-NN tests).

- **Klotz, J. H. (1986).** "Nonparametric tests for randomness." In *Encyclopedia of Statistical Sciences*, Vol. 6, pp. 136-143.
  - Technical treatment of power for nonparametric tests based on nearest neighbors.

- **Zhang, Z. (2014).** "Statistical power analysis for effect sizes in social, behavioral, and medical research." *Psychological Test and Assessment Modeling*, 56(2), 117-142.
  - Modern treatment connecting effect sizes, power, and sample size.
