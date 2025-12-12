@def title = "Guide to Normality Testing and Robust Alternatives"
@def published = "12 December 2025"
@def tags = ["statistics"]

# Guide to Normality Testing and Robust Alternatives

## Should You Use the Kolmogorov-Smirnov Test?

**Generally, no.** The Kolmogorov-Smirnov (K-S) test is often **not** the best choice for testing normality, especially when you don't know the population parameters (mean and variance) beforehand.

## Better Alternatives

### 1. **Shapiro-Wilk Test** (Most Recommended)
- **Best for:** Small to medium sample sizes (n < 2000)
- **Power:** Highest statistical power among normality tests
- **Why it's better:** More sensitive at detecting departures from normality than K-S

**Test statistic:**
$$W = \frac{\left(\sum_{i=1}^{n} a_i x_{(i)}\right)^2}{\sum_{i=1}^{n} (x_i - \bar{x})^2}$$

where $x_{(i)}$ are the ordered sample values and $a_i$ are constants.

**Interpretation:** Small p-values (typically p < 0.05) suggest non-normality.

### 2. **Anderson-Darling Test**
- **Best for:** When you want more weight on tail deviations
- **Power:** Better than K-S, gives more weight to extreme values
- **Why use it:** Particularly good at detecting heavy-tailed or skewed distributions

**Test statistic:**
$$A^2 = -n - \sum_{i=1}^{n} \frac{2i-1}{n}\left[\ln F(x_i) + \ln(1-F(x_{n+1-i}))\right]$$

### 3. **Kolmogorov-Smirnov Test** (with Lilliefors correction)
- **When to use:** Large sample sizes, or when comparing to a fully specified distribution
- **Limitation:** Low power compared to Shapiro-Wilk
- **Important:** Use the **Lilliefors correction** when estimating parameters from data

**Test statistic:**
$$D = \sup_x |F_n(x) - F(x)|$$

where $F_n$ is the empirical CDF and $F$ is the theoretical normal CDF.

## Practical Recommendations

### Sample Size Matters

| Sample Size | Recommended Test |
|-------------|------------------|
| n < 50 | Shapiro-Wilk |
| 50 ≤ n < 2000 | Shapiro-Wilk or Anderson-Darling |
| n ≥ 2000 | Anderson-Darling or visual methods |

### Consider Visual Methods Too

Statistical tests alone aren't enough. Always use:

1. **Q-Q plots** (quantile-quantile plots) - most informative
2. **Histograms with normal overlay**
3. **Kernel density plots**

## Understanding Q-Q Plots in Detail

### What is a Q-Q Plot?

A **quantile-quantile (Q-Q) plot** compares the quantiles of your data against the quantiles of a theoretical normal distribution. If your data is normally distributed, the points should fall approximately on a straight diagonal line.

### How It Works

**The process:**
1. Sort your data values from smallest to largest
2. Calculate the theoretical quantiles (z-scores) that a normal distribution would have at those positions
3. Plot: actual data values (y-axis) vs. theoretical normal quantiles (x-axis)

**Mathematical formulation:**

For each data point $x_i$ at position $i$ in the sorted data:
- Plotting position: $p_i = \frac{i - 0.5}{n}$ (where $n$ is sample size)
- Theoretical quantile: $q_i = \Phi^{-1}(p_i)$ (inverse normal CDF)
- Plot the point: $(q_i, x_i)$

### Interpreting Q-Q Plots

#### Perfect Normality
Points fall exactly on the diagonal reference line (rarely happens with real data).

#### Good Enough Normality
Points follow the line closely with minor, random deviations. Small departures, especially in the middle, are usually fine.

#### Common Deviation Patterns

| Pattern | What It Means | Visual Description |
|---------|---------------|-------------------|
| **S-shaped curve** | Heavy tails (leptokurtic) | Points curve above line at both ends |
| **Inverted S-curve** | Light tails (platykurtic) | Points curve below line at both ends |
| **Points above line on right** | Right skew (positive skew) | Upper tail deviates upward |
| **Points below line on left** | Left skew (negative skew) | Lower tail deviates downward |
| **Exponential curve** | Exponential distribution | Systematic upward curve |
| **Discrete steps** | Discrete/grouped data | Horizontal segments instead of smooth curve |

### Why Q-Q Plots are "Most Informative"

1. **Shows WHERE deviations occur**: Unlike a single p-value, you can see if problems are in the tails, center, or throughout

2. **Magnitude of departure**: The vertical distance from the line shows how far your data deviates from normality

3. **Type of non-normality**: The pattern tells you what kind of distribution you might actually have

4. **Sample size insensitivity**: Works well with any sample size, whereas statistical tests become oversensitive with large $n$

5. **Practical assessment**: Helps you judge if departures are severe enough to matter for your analysis

### Example Interpretation

Suppose your Q-Q plot shows:
- Points hug the line in the middle
- Points curve upward at both ends

**Diagnosis:** Your data has heavier tails than a normal distribution (more extreme values than expected). This might indicate:
- Outliers or contamination
- A t-distribution-like shape
- The need for robust statistical methods

### When to Be Concerned

**Minor concerns (usually OK):**
- Slight waviness around the line
- 1-2 points slightly off at the extremes
- Small sample size with some scatter

**Major concerns (investigate further):**
- Systematic S-curve or inverted S-curve
- Large gaps between points and line
- Multiple outliers far from the line
- Clear exponential or other non-linear pattern

### Practical Tips

- **Always plot it**: Even if statistical tests say "normal," look at the Q-Q plot
- **Focus on the middle**: Extreme quantiles are naturally more variable
- **Add confidence bands**: Some software adds confidence intervals around the reference line
- **Compare to reference distributions**: Create Q-Q plots for known normal data to calibrate your eye

### Why Visual Methods Matter

- Statistical tests can be **oversensitive** with large samples (detecting trivial departures)
- They can be **underpowered** with small samples
- Many statistical procedures are **robust** to mild violations of normality

## Decision Framework

```
Do you need to test normality?
│
├─ Small sample (n < 50)
│  └─ Use: Shapiro-Wilk + Q-Q plot
│
├─ Medium sample (50 ≤ n < 2000)
│  └─ Use: Shapiro-Wilk or Anderson-Darling + Q-Q plot
│
└─ Large sample (n ≥ 2000)
   └─ Use: Visual methods primarily (Q-Q plot)
      └─ If test needed: Anderson-Darling
```

## The "Test-Then-Decide" Approach: A Controversial Practice

### Is Shapiro-Wilk → t-test Standard Practice?

**Short answer:** It's common, but **statisticians generally advise against it**.

### Why It's Problematic

#### 1. **The Multiple Testing Problem**
When you test normality first (α = 0.05), then perform a t-test (α = 0.05), your actual Type I error rate is **not** 5% anymore. You're making two decisions, which compounds error rates.

#### 2. **Mismatched Statistical Power**
- With **small samples**: Normality tests have low power (often fail to detect non-normality when it exists) → you use t-test even when assumptions are violated
- With **large samples**: Normality tests have high power (detect trivial departures) → you abandon t-test even when it would work fine due to robustness

This is backwards from what you need!

#### 3. **The Robustness Paradox**

The t-test is remarkably robust to violations of normality, especially when:
- Sample sizes are moderate to large (n ≥ 30 per group)
- Sample sizes are roughly equal between groups
- Distributions are not extremely skewed

So you might abandon a perfectly valid t-test based on a "failed" normality test.

### What Statisticians Recommend Instead

#### **Option 1: Assume Nothing, Use Robust Methods** (Safest)
Skip normality testing entirely and use methods that work regardless:
- **Welch's t-test** (doesn't assume equal variances)
- **Mann-Whitney U test** (non-parametric alternative) - see detailed explanation below
- **Permutation tests** (exact, no distributional assumptions)
- **Bootstrap methods** (resampling-based inference)

#### **Option 2: Check Assumptions, Don't Test Them** (Most Common)
- Use **Q-Q plots** and **histograms** to visually assess normality
- Look for major violations (extreme skewness, heavy outliers)
- Make an **informed judgment** rather than a mechanical p-value cutoff
- Proceed with t-test if violations are minor

#### **Option 3: Let Theory Guide You** (Best Practice)
- Consider what you know about the data generation process
- Use domain knowledge about typical distributions
- Choose methods based on study design, not mechanical testing

### A Better Decision Framework

```
Before analysis:
│
├─ Check sample size
│  ├─ n < 15 per group → Consider non-parametric tests
│  └─ n ≥ 30 per group → t-test is robust
│
├─ Visualize data (Q-Q plots, boxplots)
│  ├─ Extreme skewness or outliers? → Transform or use robust methods
│  └─ Roughly symmetric? → t-test is fine
│
└─ Check equal variances (more important than normality!)
   ├─ Very unequal? → Use Welch's t-test
   └─ Similar? → Standard t-test is fine
```

### What to Do Instead

**Good workflow:**
1. **Visualize first** - make Q-Q plots, histograms, boxplots
2. **Think about context** - what do you know about this type of data?
3. **Assess severity** - are violations severe or trivial?
4. **Choose appropriate test**:
   - Minor issues + n ≥ 30: Regular t-test
   - Moderate skewness: Welch's t-test or transform data
   - Severe violations: Non-parametric test or bootstrap
5. **Consider reporting both** - parametric and non-parametric results

## Understanding the Mann-Whitney U Test

### What Does It Actually Test?

You're right - that formal definition is terrible! Here's what the Mann-Whitney U test really does:

**Simple version:** It tests whether one group tends to have **larger values** than the other group.

**More precisely:** It tests whether the two groups come from populations with the same **distribution** (location, spread, and shape).

### The Intuitive Explanation

Imagine you have two groups:
- **Group X:** Reaction times for Drug A
- **Group Y:** Reaction times for Drug B

The Mann-Whitney U test asks: **"If I randomly pick one person from Group X and one from Group Y, what's the probability that X > Y?"**

- If the groups are identical, this probability should be 50% (like flipping a fair coin)
- If Group X tends to be larger, the probability will be > 50%
- If Group Y tends to be larger, the probability will be < 50%

**Null hypothesis:** $P(X > Y) = 0.5$ (no difference between groups)

**Alternative hypothesis:** $P(X > Y) \neq 0.5$ (one group tends to be larger)

**Important note on symmetry:** The test is completely symmetric - it doesn't matter which group you call X or Y. The null hypothesis $P(X > Y) = 0.5$ is equivalent to $P(X > Y) = P(Y > X)$, and both probabilities equal 0.5 when groups are identical. The alternative hypothesis $P(X > Y) \neq 0.5$ captures both possibilities: either $P(X > Y) > 0.5$ (X tends larger) or $P(X > Y) < 0.5$ (Y tends larger). You'll get the same p-value regardless of group labeling.

### How It Actually Works

The test uses **ranks** instead of raw values:

1. **Combine all data** from both groups
2. **Rank everything** from smallest (rank 1) to largest (rank n)
3. **Sum the ranks** for each group
4. **Compare the sums** - if one group has much higher ranks, it tends to have larger values

**Example:**

Group A: 12, 15, 18  
Group B: 10, 14, 20

Combined and ranked:
- 10 (B, rank 1)
- 12 (A, rank 2)
- 14 (B, rank 3)
- 15 (A, rank 4)
- 18 (A, rank 5)
- 20 (B, rank 6)

Sum of ranks:
- Group A: 2 + 4 + 5 = 11
- Group B: 1 + 3 + 6 = 10

Are these sums different enough to conclude the groups differ? The Mann-Whitney U statistic tells us.

### What It's Really Testing

**Most common interpretation:** Tests if the **medians** differ between groups

**More accurate interpretation:** Tests if the entire **distributions** differ in location

**Technical truth:** Tests if values from one group are **stochastically larger** than the other

### When Interpretations Matter

#### If distributions have the same shape (just shifted):
✓ You can say: "Group A has a higher median than Group B"  
✓ You can interpret it as a location shift

#### If distributions have different shapes:
⚠️ Be careful saying "medians differ"  
✓ Better: "Group A tends to have larger values than Group B"  
✓ Or: "The distributions differ"

### Why Use Mann-Whitney Instead of t-test?

| Situation | Better Choice | Reason |
|-----------|---------------|---------|
| Ordinal data (rankings, Likert scales) | Mann-Whitney | t-test requires interval/ratio data |
| Severe skewness | Mann-Whitney | Not sensitive to outliers |
| Small sample + non-normal | Mann-Whitney | Fewer assumptions |
| Outliers present | Mann-Whitney | Uses ranks, not raw values |
| Normal-ish data, n ≥ 30 | t-test | More statistical power |

### The U Statistic Formula

For Group X with size $n_x$ and Group Y with size $n_y$:

$U_x = n_x n_y + \frac{n_x(n_x + 1)}{2} - R_x$

where $R_x$ is the sum of ranks for Group X.

The test uses the **smaller** of $U_x$ and $U_y$.

### Common Misconceptions

❌ **"It tests if medians are different"** → Only true if distributions have the same shape  
❌ **"It's just a t-test on ranks"** → Close, but not exactly  
❌ **"It's always better than a t-test"** → No, t-test is more powerful when assumptions are met  
✓ **"It tests if one group stochastically dominates the other"** → Most accurate

### Practical Takeaway

Use Mann-Whitney when you want to test **"Does Group A tend to have larger/smaller values than Group B?"** without assuming anything about the shape of the distributions. It's your robust, assumption-free alternative to the t-test.

### The Academic Reality

Despite statisticians' recommendations, many fields still use the "test normality first" approach because:
- It's taught in introductory courses
- Reviewers sometimes expect it
- It provides a "defensive" paper trail

If you're in a field where this is expected, consider:
- Reporting visual assessments alongside statistical tests
- Using α = 0.01 for normality tests (more conservative)
- Noting the limitations in your methods section
- Presenting both parametric and non-parametric results

## The Bottom Line

**Avoid the standard Kolmogorov-Smirnov test for normality testing.** If you must use it, use the Lilliefors-corrected version. The Shapiro-Wilk test is generally the gold standard for most applications, combined with visual inspection through Q-Q plots.

**However**, the best practice is typically to **skip formal normality testing** and instead: (1) visualize your data, (2) consider the robustness of your chosen test, and (3) use methods that don't require strict normality assumptions when in doubt.