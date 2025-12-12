@def title = "Power Analysis for Mann-Whitney U Test"
@def published = "12 December 2025"
@def tags = ["statistics"]

# Power Analysis for Mann-Whitney U Test

## Does This Apply to Mann-Whitney U Test?

**Yes, with modifications.** The principles of power analysis apply broadly to hypothesis tests comparing groups, but the Mann-Whitney U test (also known as the Wilcoxon rank-sum test) has important characteristics for independent samples:

### Key Differences for Mann-Whitney U Test

1. **Efficiency Loss**: Non-parametric tests are slightly less powerful than parametric tests (t-tests) when data is normally distributed
   * **Rule of thumb**: Non-parametric tests require ~5-15% more samples than t-tests for the same power
   * The Mann-Whitney U test needs approximately 5% more samples than an independent samples t-test

2. **Effect Size Measures**:
   * t-tests use **Cohen's d** (difference in means / pooled SD)
   * Mann-Whitney uses **rank-biserial correlation** or **probability of superiority**
   * Approximate conversion: For large samples, Cohen's d ≈ effect size for Mann-Whitney

3. **When to Use Which Test**:
   * **Use t-test** if: Data is approximately normal, or n≥30 per group (Central Limit Theorem)
   * **Use Mann-Whitney** if: Data is heavily skewed, has outliers, or violates normality assumptions

---

## Power Analysis Procedure for Mann-Whitney U Test

### Step 1: Determine Your Design

**Mann-Whitney U test** is for:
* **Independent samples** (two separate groups)
* **Between-subjects designs**
* **Different participants in each group**

*Note: For paired/matched samples, use Wilcoxon signed-rank test instead.*

### Step 2: Calculate Required Sample Size

For **independent** Mann-Whitney test, use the independent t-test formula with adjustment:

$$n = \frac{2(Z_{1-\alpha/2} + Z_{1-\beta})^2}{d^2} \times \text{ARE}$$

Where:
* **ARE** (Asymptotic Relative Efficiency) ≈ 0.955 when data is normal
* This means Mann-Whitney needs about 5% more samples per group than independent t-test
* For α = 0.05, 80% power: $n \approx \frac{15.68}{d^2} \times 1.05 \approx \frac{16.5}{d^2}$ per group

### Step 3: Account for Unequal Group Sizes

If groups have unequal sizes (n₁ and n₂):

$$n_{\text{effective}} = \frac{2n_1 n_2}{n_1 + n_2}$$

This effective sample size is what matters for power calculations.

**Example**: If you have 40 in Group A and 20 in Group B:
* $n_{\text{eff}} = \frac{2 \times 40 \times 20}{40 + 20} = \frac{1600}{60} = 26.7$
* This is equivalent to having ~27 participants per group with equal sizes

### Step 4: Check Minimum Detectable Effect

For Mann-Whitney with n per group and α = 0.05, 80% power:

$$d_{\min} \approx \frac{4.05 \times 1.05}{\sqrt{n}} = \frac{4.25}{\sqrt{n}}$$

**Reference table for Mann-Whitney:**

| n per group | Min Detectable d | Interpretation |
|-------------|------------------|----------------|
| 10 | 1.34 | Very large effects only |
| 20 | 0.95 | Large effects |
| 30 | 0.78 | Large effects |
| 50 | 0.60 | Medium-large effects |
| 100 | 0.43 | Medium effects |
| 200 | 0.30 | Small-medium effects |
| 400 | 0.21 | Small effects |

---

## Complete Testing Procedure

### 1. Check Assumptions

**Before choosing your test:**

```
Normality Check (for each group):
├─ Shapiro-Wilk test (for n < 50)
├─ Visual: QQ-plot
└─ If p < 0.05 or clear skewness → Use Mann-Whitney U

Independence Check:
└─ Ensure groups are truly independent (different participants)
```

### 2. Conduct the Mann-Whitney U Test

**In Julia:**

```julia
using HypothesisTests
using Statistics

# Mann-Whitney U test (called MannWhitneyUTest in Julia)
test_result = MannWhitneyUTest(x, y)

# Get p-value
p_value = pvalue(test_result)

# Get U statistic
u_statistic = test_result.U

# Calculate effect size (rank-biserial correlation)
n1, n2 = length(x), length(y)
r = 1 - (2 * u_statistic) / (n1 * n2)
```

### 3. Calculate Power Post-Hoc

After collecting data, verify you had adequate power:

```julia
using Distributions

# Function to calculate power for Mann-Whitney U test
function mann_whitney_power(n, d, alpha=0.05)
    z_alpha = quantile(Normal(), 1 - alpha/2)
    z_beta = d * sqrt(n/2) - z_alpha
    power = cdf(Normal(), z_beta)
    return power
end

# For observed effect size
n_per_group = 50
observed_d = 0.5
power = mann_whitney_power(n_per_group, observed_d)
println("Power: ", round(power, digits=3))

# Adjust for Mann-Whitney efficiency (multiply n by 0.955)
adjusted_power = mann_whitney_power(n_per_group * 0.955, observed_d)
```

---

## Reporting Your Results

### Minimal Reporting (APA Style)

> A Mann-Whitney U test indicated that Group A (Mdn = 85) had significantly higher scores than Group B (Mdn = 78), U = 245, p = .003, r = .38.

### Full Reporting (Research Paper)

> **Statistical Analysis:** A Mann-Whitney U test was used to compare scores between groups due to non-normal distribution in Group B (Shapiro-Wilk test, p = .02). Based on a priori power analysis, 50 participants per group provided 80% power to detect a medium effect size (d ≥ 0.60) at α = 0.05 (two-tailed).
>
> **Results:** The Mann-Whitney U test revealed a statistically significant difference between groups, U = 245, p = .003. The effect size (rank-biserial correlation r = .38) indicated a medium effect. Group A scores (Mdn = 85, IQR = 79-92) were higher than Group B scores (Mdn = 78, IQR = 71-84).

### What to Include

**Essential elements:**

1. **Test used**: "Mann-Whitney U test" or "Wilcoxon rank-sum test"
2. **Justification**: Why not t-test? ("non-normal distribution", "presence of outliers")
3. **Sample sizes**: n₁ and n₂ (number per group)
4. **Test statistic**: U-statistic or W-statistic
5. **P-value**: Exact value when p > .001
6. **Effect size**: Rank-biserial correlation (r) or probability of superiority
7. **Descriptive statistics**: Medians and IQR (not means/SD)

**For power analysis reporting:**

```
A priori: "Power analysis indicated that n = 50 per group 
          provided 80% power to detect d ≥ 0.60."

Post-hoc: "With n₁ = 45 and n₂ = 40, the study achieved 80% 
          power to detect effects of d ≥ 0.65."

Underpowered: "The sample (n = 20 per group) provided 80% 
              power only for large effects (d ≥ 0.95). 
              Smaller effects may have been missed."
```

---

## Practical Example: Complete Workflow

### Scenario

Testing whether a new teaching method improves test scores compared to traditional method (independent groups design, n₁ = 50, n₂ = 50).

### Step-by-Step:

**1. Check if you have adequate power:**

```
n = 50 per group
d_min = 4.25/√50 = 4.25/7.07 ≈ 0.60

Expected effect from literature: d = 0.7
0.7 > 0.60 → ✓ Adequate power!
```

**2. Check normality:**

```
Group 1 Shapiro-Wilk: p = 0.32 → Normal
Group 2 Shapiro-Wilk: p = 0.01 → Not normal
Decision: Use Mann-Whitney (one group violates normality)
```

**3. Run the test:**

```julia
using HypothesisTests

# Mann-Whitney U test
result = MannWhitneyUTest(new_method, traditional)

# Get results
println("U statistic: ", result.U)
println("p-value: ", pvalue(result))

# Calculate effect size (rank-biserial correlation)
n1, n2 = length(new_method), length(traditional)
r = 1 - (2 * result.U) / (n1 * n2)
println("Effect size (r): ", round(r, digits=2))
```

**4. Report:**

> "A Mann-Whitney U test (chosen due to non-normal distribution in the control group, p = .01) indicated that students in the new teaching method group scored significantly higher than those in the traditional method group (U = 1,725, p < .001, r = .42). New method scores (Mdn = 84, IQR = 78-89) exceeded traditional method scores (Mdn = 76, IQR = 70-82). With n = 50 per group, this study achieved 80% power to detect effects of d ≥ 0.60, confirming adequate statistical power for the observed medium-to-large effect."

---

## Quick Decision Tree

```
Start: Do you have paired or independent data?
    ↓
INDEPENDENT → Check normality (both groups)
    ↓
Both Normal? → YES: Use independent samples t-test
             → NO: Use Mann-Whitney U test
    ↓
Calculate: d_min = 4.25/√n
    ↓
Compare: Expected d > d_min?
    ↓
YES → ✓ Adequate power, proceed
NO  → ✗ Need more participants per group
```

---

## Common Mistakes to Avoid

❌ **WRONG:** "I'll use the same sample size formula as paired tests"
* Mann-Whitney (independent) needs nearly twice as many total participants as paired tests

✓ **CORRECT:** Use independent samples formula with appropriate adjustment

❌ **WRONG:** "I'll report means and standard deviations"
* Non-parametric tests compare medians, not means

✓ **CORRECT:** Report medians and interquartile ranges (IQR)

❌ **WRONG:** "I'll use Cohen's d directly from my data"
* Use rank-biserial correlation for Mann-Whitney

✓ **CORRECT:** Calculate and report rank-biserial r

❌ **WRONG:** "Mann-Whitney and Wilcoxon are different tests"
* Mann-Whitney U test and Wilcoxon rank-sum test are the same test (different names)
* Don't confuse with Wilcoxon signed-rank test (for paired data)

✓ **CORRECT:** Mann-Whitney U = Wilcoxon rank-sum (both for independent samples)

---

## Additional Resources

**Effect Size Conversions** (approximate):
* Cohen's d = 0.2 ≈ r = 0.10 (small)
* Cohen's d = 0.5 ≈ r = 0.24 (medium)
* Cohen's d = 0.8 ≈ r = 0.37 (large)

**When sample size < 30 per group:**
* Consider exact p-values instead of normal approximation
* Use U-statistic directly
* Be more cautious about power—non-parametric tests lose more efficiency with small samples

**Handling ties in ranks:**
* Mann-Whitney U test automatically accounts for tied values
* Most software implementations apply tie corrections by default
* Large numbers of ties may reduce test power slightly