@def title = "Power Analysis for Non-Parametric Tests"
@def published = "12 December 2025"
@def tags = ["statistics"]

# Power Analysis for Non-Parametric Tests

## Does This Apply to Wilcoxon Signed-Rank Test?

**Yes, with modifications.** The principles from the article apply broadly to hypothesis tests comparing groups, but non-parametric tests like the Wilcoxon signed-rank test have important differences:

### Key Differences for Non-Parametric Tests

1. **Efficiency Loss**: Non-parametric tests are slightly less powerful than parametric tests (t-tests) when data is normally distributed
   - **Rule of thumb**: Non-parametric tests require ~5-15% more samples than t-tests for the same power
   - The article mentions this: "Non-parametric tests: Slightly less powerful than t-tests when data is normal; may need ~5-15% more samples"

2. **Effect Size Measures**: 
   - t-tests use **Cohen's d** (difference in means / pooled SD)
   - Wilcoxon uses **rank-biserial correlation** or **probability of superiority**
   - Approximate conversion: For large samples, Cohen's d ≈ effect size for Wilcoxon

3. **When to Use Which Test**:
   - **Use t-test** if: Data is approximately normal, or $n \geq 30$ per group (Central Limit Theorem)
   - **Use Wilcoxon** if: Data is heavily skewed, has outliers, or violates normality assumptions

---

## Power Analysis Procedure for Wilcoxon Signed-Rank Test

### Step 1: Determine Your Design

**Wilcoxon signed-rank test** is for:
- **Paired samples** (same subjects, two conditions)
- **Before-after measurements**
- **Matched pairs**

*Note: For independent groups, use Mann-Whitney U test instead.*

### Step 2: Calculate Required Sample Size

For **paired** Wilcoxon test, use the paired t-test formula with adjustment:

$$n = \frac{(Z_{1-\alpha/2} + Z_{1-\beta})^2}{d^2} \times \text{ARE}$$

Where:
- **ARE** (Asymptotic Relative Efficiency) ≈ 0.955 when data is normal
- This means Wilcoxon needs about 5% more samples than paired t-test
- For α = 0.05, 80% power: $n \approx \frac{7.84}{d^2} \times 1.05 \approx \frac{8.2}{d^2}$

### Step 3: Calculate Effective Sample Size for Paired Design

The article mentions paired tests are "much more powerful" due to correlation:

$$n_{\text{eff,paired}} = n \times (1 - \rho)$$

Where $\rho$ is the correlation between paired measurements.

**Example**: If you have 30 paired measurements with correlation $\rho = 0.6$:
- Effective unpaired sample size: $n_{\text{eff}} = 30 \times (1 - 0.6) = 12$
- But for paired test, you only need 30 pairs (not 30 per group!)

### Step 4: Check Minimum Detectable Effect

For Wilcoxon with $n$ pairs and α = 0.05, 80% power:

$$d_{\min} \approx \frac{2.8 \times 1.05}{\sqrt{n}} = \frac{2.94}{\sqrt{n}}$$

**Reference table for Wilcoxon:**

| Pairs (n) | Min Detectable d | Interpretation |
|-----------|------------------|----------------|
| 10        | 0.93             | Very large effects only |
| 20        | 0.66             | Large effects |
| 30        | 0.54             | Medium-large effects |
| 50        | 0.42             | Medium effects |
| 100       | 0.29             | Small-medium effects |
| 200       | 0.21             | Small effects |

---

## Complete Testing Procedure

### 1. Check Assumptions

**Before choosing your test:**

```
Normality Check:
├─ Shapiro-Wilk test (for n < 50)
├─ Visual: QQ-plot
└─ If p < 0.05 or clear skewness → Use Wilcoxon
```

### 2. Conduct the Wilcoxon Test

**In R:**
```r
# For paired samples
wilcox.test(x, y, paired = TRUE, 
            alternative = "two.sided")

# With effect size
library(effectsize)
rank_biserial(x, y, paired = TRUE)
```

**In Python:**
```python
from scipy.stats import wilcoxon
from scipy.stats import ranksums  # for independent

# Paired test
statistic, p_value = wilcoxon(x, y)

# Effect size (rank-biserial)
from pingouin import wilcoxon
result = wilcoxon(x, y)
print(result['RBC'])  # rank-biserial correlation
```

### 3. Calculate Power Post-Hoc

After collecting data, verify you had adequate power:

```r
# Install: install.packages("pwr")
library(pwr)

# For paired samples (approximate)
pwr.t.test(n = 30,           # number of pairs
           d = 0.5,          # observed effect size
           sig.level = 0.05,
           type = "paired",
           alternative = "two.sided")

# Adjust by multiplying n by 0.955 for Wilcoxon
```

---

## Reporting Your Results

### Minimal Reporting (APA Style)

> A Wilcoxon signed-rank test indicated that the post-treatment scores (Mdn = 85) were significantly higher than pre-treatment scores (Mdn = 78), Z = 3.24, p = .001, r = .42.

### Full Reporting (Research Paper)

> **Statistical Analysis:** A Wilcoxon signed-rank test was used to compare pre- and post-treatment scores due to non-normal distribution (Shapiro-Wilk test, p < .05). Based on a priori power analysis, 50 paired observations provided 80% power to detect a medium effect size (d ≥ 0.42) at α = 0.05 (two-tailed).
>
> **Results:** The Wilcoxon signed-rank test revealed a statistically significant increase in scores following treatment, Z = 3.24, p = .001. The effect size (rank-biserial correlation r = .42) indicated a medium-to-large effect. Post-treatment scores (Mdn = 85, IQR = 79-92) were higher than pre-treatment scores (Mdn = 78, IQR = 71-84).

### What to Include

**Essential elements:**
1. **Test used**: "Wilcoxon signed-rank test"
2. **Justification**: Why not t-test? ("non-normal distribution")
3. **Sample size**: Number of pairs
4. **Test statistic**: Z-value (for large n) or W-statistic
5. **P-value**: Exact value when p > .001
6. **Effect size**: Rank-biserial correlation (r) or probability of superiority
7. **Descriptive statistics**: Medians and IQR (not means/SD)

**For power analysis reporting:**
```
A priori: "Power analysis indicated that n = 50 pairs 
          provided 80% power to detect d ≥ 0.42."

Post-hoc: "With n = 35 pairs, the study achieved 80% 
          power to detect effects of d ≥ 0.50."

Underpowered: "The sample (n = 20 pairs) provided 80% 
              power only for large effects (d ≥ 0.66). 
              Smaller effects may have been missed."
```

---

## Practical Example: Complete Workflow

### Scenario
Testing whether a new meditation app reduces stress scores (paired design, n = 40 participants).

### Step-by-Step:

**1. Check if you have adequate power:**
```
n = 40 pairs
d_min = 2.94/√40 = 2.94/6.32 ≈ 0.47

Expected effect from literature: d = 0.6
0.6 > 0.47 → ✓ Adequate power!
```

**2. Check normality:**
```
Shapiro-Wilk test: p = 0.02 → Not normal
QQ-plot shows heavy tails
Decision: Use Wilcoxon (not paired t-test)
```

**3. Run the test:**
```r
# Calculate differences
diff <- post_stress - pre_stress

# Wilcoxon test
result <- wilcox.test(post_stress, pre_stress, 
                      paired = TRUE)

# Effect size
library(effectsize)
r <- rank_biserial(post_stress, pre_stress, 
                   paired = TRUE)
```

**4. Report:**
> "A Wilcoxon signed-rank test (chosen due to non-normal distribution, p = .02) indicated significant stress reduction following the 8-week meditation program (Z = -3.89, p < .001, r = -.45). Pre-intervention stress scores (Mdn = 72, IQR = 65-78) decreased to post-intervention scores (Mdn = 58, IQR = 52-64). With n = 40 pairs, this study achieved 80% power to detect effects of d ≥ 0.47, confirming adequate statistical power for the observed medium-to-large effect."

---

## Quick Decision Tree

```
Start: Do you have paired or independent data?
    ↓
PAIRED → Check normality
    ↓
Normal? → YES: Use paired t-test
        → NO: Use Wilcoxon signed-rank test
    ↓
Calculate: d_min = 2.94/√n
    ↓
Compare: Expected d > d_min?
    ↓
YES → ✓ Adequate power, proceed
NO  → ✗ Need more participants
```

---

## Common Mistakes to Avoid

❌ **WRONG:** "I'll use the exact same sample size formula as t-tests"
- Wilcoxon needs ~5-15% more samples

✓ **CORRECT:** Multiply required n by 1.05-1.15 adjustment factor

❌ **WRONG:** "I'll report means and standard deviations"
- Non-parametric tests compare medians, not means

✓ **CORRECT:** Report medians and interquartile ranges (IQR)

❌ **WRONG:** "I'll use Cohen's d directly from my data"
- Use rank-biserial correlation for Wilcoxon

✓ **CORRECT:** Calculate and report rank-biserial r

---

## Additional Resources

**Effect Size Conversions** (approximate):
- Cohen's d = 0.2 ≈ r = 0.10 (small)
- Cohen's d = 0.5 ≈ r = 0.24 (medium)  
- Cohen's d = 0.8 ≈ r = 0.37 (large)

**When sample size < 30:**
- Consider exact p-values instead of Z-approximation
- Use W-statistic (sum of positive ranks) instead of Z
- Be more cautious about power—non-parametric tests lose more efficiency with small samples