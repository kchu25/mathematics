@def title = "Tests on Variances: From F-test to Fligner-Killeen"
@def published = "17 February 2026"
@def tags = ["statistics"]

# Tests on Variances: From F-test to Fligner-Killeen

## Why Test Variances?

Many procedures (ANOVA, pooled t-tests) assume **equal variances** across groups. Before running them, you need to check that assumption. The core question is always:

$$H_0: \sigma_1^2 = \sigma_2^2 = \cdots = \sigma_k^2$$

The five tests below form a spectrum — from strong distributional assumptions with high power, to minimal assumptions with robustness against messy data.

| Test | Type | Sensitivity to Non-Normality | Best Use Case |
|------|------|-----|------|
| F-test | Parametric | Very High | Comparing exactly 2 groups confirmed Normal |
| Bartlett's | Parametric | High | Comparing 2+ groups; more powerful but requires Normality |
| Levene's | Semi-Parametric | Moderate | Standard for checking ANOVA assumptions (uses mean deviations) |
| Brown–Forsythe | Semi-Parametric | Low | Skewed data (uses median deviations to ignore outliers) |
| Fligner–Killeen | Non-Parametric | Very Low | Gold standard for messy, non-normal data with heavy outliers |

---

## 1. F-test for Equality of Two Variances

### Intuition

The simplest idea: take the ratio of two sample variances. If both populations have the same true variance, this ratio should hover around 1. The further it deviates from 1, the stronger the evidence that the variances differ.

### Math

Given two independent samples of sizes $n_1$ and $n_2$ drawn from normal populations,

$$F = \frac{s_1^2}{s_2^2}, \qquad F \sim F(n_1 - 1,\; n_2 - 1) \text{ under } H_0$$

where convention places the larger variance in the numerator. Reject $H_0$ if $F > F_{\alpha/2}$ (two-tailed) or $F > F_{\alpha}$ (one-tailed).

### Why it breaks

The F-distribution derivation **requires** both populations to be exactly normal. Even mild skewness or heavy tails inflate the Type I error rate dramatically — you'll "detect" variance differences that don't exist.

**Use when:** You have exactly 2 groups and strong evidence of normality. Otherwise, keep reading.

---

## 2. Bartlett's Test

### Intuition

Bartlett's test generalises the F-test to $k \geq 2$ groups. It pools information across all groups and asks: "Is the variation in the group variances larger than what chance alone would produce?"

### Math

Let each group $i$ have sample size $n_i$ and sample variance $s_i^2$. Define $N = \sum n_i$ and the pooled variance

$$s_p^2 = \frac{\sum_{i=1}^k (n_i - 1)\, s_i^2}{N - k}.$$

The test statistic is

$$\chi^2_B = \frac{(N-k)\ln s_p^2 - \sum_{i=1}^k (n_i-1)\ln s_i^2}{1 + \dfrac{1}{3(k-1)}\!\left(\sum_{i=1}^k \dfrac{1}{n_i-1} - \dfrac{1}{N-k}\right)}$$

Under $H_0$ (all variances equal) and normality, $\chi^2_B \sim \chi^2(k-1)$.

### Reading the statistic

- **Numerator:** Compares the log of the pooled variance to the average of the individual log-variances. If variances are equal, these coincide (by Jensen's inequality, $\ln$ of the mean $\geq$ mean of $\ln$, with equality iff all values are the same).
- **Denominator:** A small-sample correction factor that pushes the distribution closer to $\chi^2$.

### Why it breaks

Like the F-test, Bartlett's test is built on the normality assumption. It is **even more** sensitive to kurtosis than the F-test: heavy-tailed data inflates $\chi^2_B$ and produces false positives.

**Use when:** You have $k \geq 2$ groups that you are confident are normally distributed. It is more powerful than Levene's in that case.

---

## 3. Levene's Test

### Intuition

Here's the clever trick: instead of working with variances directly, **transform** each observation into its absolute deviation from the group mean, then run a standard one-way ANOVA on those deviations. If the original groups had equal variances, the deviations should have equal means across groups.

### Math

For observation $y_{ij}$ in group $i$, compute

$$z_{ij} = |y_{ij} - \bar{y}_i|$$

where $\bar{y}_i$ is the mean of group $i$. Then apply a one-way ANOVA F-statistic to the $z_{ij}$:

$$W = \frac{(N-k)}{(k-1)} \cdot \frac{\sum_{i=1}^k n_i (\bar{z}_{i\cdot} - \bar{z}_{\cdot\cdot})^2}{\sum_{i=1}^k \sum_{j=1}^{n_i} (z_{ij} - \bar{z}_{i\cdot})^2}$$

Under $H_0$, $W \;\dot\sim\; F(k-1,\; N-k)$.

### Why it works

By replacing raw values with deviations from the center, you convert a **variance comparison** into a **mean comparison** — and ANOVA is well-understood for comparing means. The absolute value makes the test moderately robust to non-normality, though it still uses the **mean** as the center, which is itself affected by outliers.

**Use when:** Checking the homogeneity-of-variance assumption before ANOVA. This is the default in most software packages.

---

## 4. Brown–Forsythe Test

### Intuition

Brown–Forsythe is Levene's test with one modification: replace the group **mean** with the group **median**. Since the median is resistant to outliers, this makes the test much more robust to skewed or heavy-tailed data.

### Math

Define

$$z_{ij} = |y_{ij} - \tilde{y}_i|$$

where $\tilde{y}_i$ is the **median** of group $i$. Then compute the same $W$ statistic as Levene's test on the $z_{ij}$ values, again compared to $F(k-1, N-k)$.

### Why the median matters

Consider a group with an extreme outlier. Using the mean, the outlier pulls the center toward itself, and every other observation gets a larger deviation — **inflating** the apparent variance. The median barely moves, so the outlier's deviation is large but doesn't distort everyone else's. This makes Brown–Forsythe more honest about the bulk of the data.

**Use when:** Your data is skewed, has outliers, or you don't trust normality but still want a familiar F-based framework.

---

## 5. Fligner–Killeen Test

### Intuition

Go fully non-parametric. Instead of working with the raw deviations, **rank** them. This throws away magnitude information (a deviation of 100 is treated the same as a deviation of 10, as long as they have the same rank) and makes the test immune to outliers and distributional shape.

### Math

1. Compute absolute deviations from each group's median: $z_{ij} = |y_{ij} - \tilde{y}_i|$.
2. Rank all $N$ values of $z_{ij}$ together to get ranks $r_{ij}$.
3. Convert ranks to **normal scores**: $a_{ij} = \Phi^{-1}\!\left(\frac{1 + r_{ij}/(N+1)}{2}\right)$, where $\Phi^{-1}$ is the standard normal quantile function.
4. Compute

$$\chi^2_{FK} = \frac{\sum_{i=1}^k n_i (\bar{a}_{i\cdot} - \bar{a}_{\cdot\cdot})^2}{\sum_{i=1}^k \sum_{j=1}^{n_i}(a_{ij} - \bar{a}_{\cdot\cdot})^2 / (N-1)}$$

Under $H_0$, $\chi^2_{FK} \sim \chi^2(k-1)$.

### Why it's the most robust

- **Median** centering removes outlier influence on the center.
- **Ranking** removes outlier influence on the scale.
- **Normal scores** (van der Waerden scores) give slightly more power than raw ranks while preserving robustness.

The cost: you lose some statistical power when the data really **is** normal, since you're discarding information about magnitudes.

**Use when:** Data is clearly non-normal, has heavy tails, or you have no idea what the distribution looks like. This is the safest choice.

---

## The Progression

All five tests answer the same question, but they increasingly **protect** against non-normality by giving up parametric power:

$$\underbrace{F\text{-test} \to \text{Bartlett's}}_{\text{parametric: use variances directly}} \to \underbrace{\text{Levene's} \to \text{Brown–Forsythe}}_{\text{use deviations from center}} \to \underbrace{\text{Fligner–Killeen}}_{\text{rank the deviations}}$$

Each arrow represents a trade-off: you lose a bit of power under perfect normality, but gain substantial protection against the real world.

---

## Quick Decision Tree

```
Is your data normally distributed?
│
├─ Yes, confident
│   ├─ Exactly 2 groups? → F-test
│   └─ 3+ groups?       → Bartlett's test
│
├─ Approximately / unsure
│   ├─ Symmetric, mild outliers? → Levene's test
│   └─ Skewed or moderate outliers? → Brown–Forsythe test
│
└─ No / heavy outliers / unknown distribution
    └─ Fligner–Killeen test
```

---

## Key Takeaways

1. **Don't default to the F-test** — it's fragile outside of perfect normality.
2. **Levene's test** (mean-based) is the standard default for ANOVA assumption checking.
3. **Brown–Forsythe** (median-based) is strictly better when data is skewed — prefer it in practice.
4. **Fligner–Killeen** is the safest option for ugly data, at a small cost in power.
5. **Bartlett's** is only worth using when you are certain the data is normal and want maximum power across $k$ groups.
6. Always **pair** a formal test with visual inspection (boxplots, density plots) — no single test tells the whole story.
