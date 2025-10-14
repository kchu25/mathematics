@def title = "Sample Size for Comparing Population Means"
@def published = "14 October 2025"
@def tags = ["statistics"]
# Sample Size for Comparing Population Means

## Key Question
**How many samples do I need per group to meaningfully compare population means?**

**Answer:** No fixed minimum—it depends on:
- Effect size you want to detect
- Within-group variability
- Significance level (α)
- Desired statistical power (1−β)

> **📋 What Statistical Tests Does This Apply To?**
> 
> **YES - These calculations apply to:**
> - **Independent samples t-test** (two groups, different subjects) ✓
> - **Paired t-test** (two measurements, same subjects) - but more powerful (see note below)
> - **One-way ANOVA** (multiple groups) - with modifications for multiple comparisons
> - **Z-test** (when population variance is known)
> - **Mann-Whitney U test / Wilcoxon** (non-parametric alternatives) - approximately
> 
> **The formulas in this document are specifically derived for:**
> - Two independent groups
> - Normal distributions (or large enough n for Central Limit Theorem)
> - Equal variances (pooled variance assumption)
> - Two-sided tests
> 
> **Important notes:**
> - **t-test vs z-test**: At n≥30, t-distribution ≈ normal distribution, so formulas using Z-values are accurate approximations
> - **Paired t-test**: Much more powerful! Effective sample size increases by factor of $1/(1-\rho)$ where $\rho$ is correlation between paired measurements
> - **Non-parametric tests**: Slightly less powerful than t-tests when data is normal; may need ~5-15% more samples
> - **ANOVA with >2 groups**: Use similar principles but adjust for multiple comparisons (use Cohen's f instead of d)
> 
> **Bottom line**: These power calculations are most accurate for independent samples t-tests, but the general principles (more samples = detect smaller effects) apply to all comparison tests.

---

## The Formula

For **two independent groups** with equal sample sizes (two-sided t-test):

$$n = 2\left(\frac{Z_{1-\alpha/2} + Z_{1-\beta}}{\delta/\sigma}\right)^2$$

Where:
- $n$ = sample size per group
- $Z_{1-\alpha/2}$ = critical value for significance (1.96 for α=0.05)
- $Z_{1-\beta}$ = critical value for power (0.84 for 80% power)
- $\delta$ = minimum detectable difference between means
- $\sigma$ = pooled standard deviation

> **📏 What is δ (delta) - Minimum Detectable Difference?**
> 
> **δ** is the smallest difference between group means that you care about detecting.
> 
> **Example**: You're comparing blood pressure between two treatments.
> - Group 1 mean: 120 mmHg
> - Group 2 mean: 115 mmHg
> - Difference: δ = 5 mmHg
> 
> **You decide**: "I only care if the difference is at least 5 mmHg. Smaller differences aren't clinically meaningful to me."
> 
> So δ = 5 mmHg is your **minimum detectable difference**. Your sample size calculation ensures you have enough power to detect differences of this size (or larger).
> 
> **Key point**: You choose δ based on what matters in your field, not statistics!

> **📐 What is σ (sigma) - Pooled Standard Deviation?**
> 
> **σ** measures how spread out (variable) your data is within each group.
> 
> **For equal sample sizes** (n₁ = n₂ = n):
> $$\sigma = \sqrt{\frac{s_1^2 + s_2^2}{2}}$$
> 
> **For unequal sample sizes**:
> $$\sigma = \sqrt{\frac{(n_1-1)s_1^2 + (n_2-1)s_2^2}{n_1 + n_2 - 2}}$$
> 
> where $s_1$ and $s_2$ are the standard deviations of each group.
> 
> **Example**: 
> - Group 1: SD = 10 mmHg, n₁ = 30
> - Group 2: SD = 12 mmHg, n₂ = 50
> - Pooled: $\sigma = \sqrt{\frac{29 \times 100 + 49 \times 144}{78}} = \sqrt{\frac{9956}{78}} \approx 11.3$ mmHg
> 
> **Why "pooled"?** We're assuming both groups have similar variability, so we combine them into one estimate.

> **📊 What is Statistical Power?**
> 
> **Power (1−β)** is the probability of detecting a real effect when it actually exists. In other words, it's your chance of avoiding a "false negative."
> 
> **The Four Possible Outcomes:**
> - **True Positive** (Power): Real effect exists → Test detects it ✓
> - **False Negative** (Type II error, β): Real effect exists → Test misses it ✗
> - **True Negative**: No effect exists → Test finds nothing ✓
> - **False Positive** (Type I error, α): No effect exists → Test claims one ✗
> 
> **Power = 80%** means: "If there really is a difference of size δ between the groups, my test has an 80% chance of finding it statistically significant."
> 
> **Why 80%?** Convention. Higher power (90%, 95%) is better but requires larger samples. Below 80% is considered underpowered—you're likely to miss real effects.
> 
> **Analogy**: Think of power as the sensitivity of a metal detector. High power (big sample) = detects small coins. Low power (small sample) = only detects large objects.

### Simplified Formula (α=0.05, 80% power)

$$n \approx 15.68\left(\frac{\sigma}{\delta}\right)^2$$

Or equivalently, using Cohen's $d = \delta/\sigma$:

$$n \approx \frac{15.68}{d^2}$$

> **🎯 What is Cohen's d - Standardized Effect Size?**
> 
> **Cohen's d** is just the difference between means (δ) divided by the standard deviation (σ):
> $$d = \frac{\delta}{\sigma} = \frac{\text{difference in means}}{\text{standard deviation}}$$
> 
> **Why use it?** It's a "standardized" measure—tells you how big the effect is relative to the variability.
> 
> **Example**:
> - Blood pressure: δ = 5 mmHg, σ = 10 mmHg → d = 0.5 (medium effect)
> - Height: δ = 5 cm, σ = 10 cm → d = 0.5 (also medium effect!)
> 
> Even though 5 mmHg ≠ 5 cm, both have d = 0.5, so they're equally "detectable" statistically.
> 
> **Cohen's Guidelines:**
> - **Small effect**: d = 0.2 (difference is 20% of one SD)
> - **Medium effect**: d = 0.5 (difference is 50% of one SD)
> - **Large effect**: d = 0.8 (difference is 80% of one SD)
> 
> **Intuition**: 
> - d = 1.0 means the two groups' means are separated by one full standard deviation
> - Larger d = easier to detect = need fewer samples
> - Smaller d = harder to detect = need more samples

---

## Worked Examples (α=0.05, 80% power)

| Effect Size | Cohen's d | $\delta/\sigma$ | n per group |
|-------------|-----------|-----------------|-------------|
| **Large** | 0.8 | δ = 0.8σ | **~25** |
| **Medium** | 0.5 | δ = 0.5σ | **~63** |
| **Small** | 0.2 | δ = 0.2σ | **~392** |

### Interpretation
- **Large effect** (difference equals 80% of SD): Need ~25 samples per group
- **Medium effect** (difference is 50% of SD): Need ~63 samples per group  
- **Small effect** (difference is 20% of SD): Need ~392 samples per group

---

## Important Considerations

### 1. **Paired vs. Independent Designs**
- **Paired (within-subject)**: Much more powerful; required $n$ reduces by factor $(1-\rho)$ where $\rho$ is the correlation between paired measurements
- Use when measuring the same subjects under different conditions

### 2. **Multiple Groups (ANOVA)**
- Formula changes for >2 groups
- Use Cohen's $f$ instead of Cohen's $d$
- Requires ANOVA power calculator

### 3. **Multiple Comparisons**
- Adjust α (e.g., Bonferroni correction)
- Adjustment increases required sample size
- Example: 5 comparisons → α = 0.05/5 = 0.01 per test

### 4. **Unequal Group Sizes or Variances**
- Formula adjustments needed
- Power decreases with imbalanced groups
- Use effective sample size (harmonic mean)
- See detailed section below

### 5. **Unknown Variance**
- Estimate $\sigma$ from pilot data or literature
- Or specify effect size directly as Cohen's $d$

---

## If Your Sample Sizes Are Already Fixed

### Question: "I have n₁ and n₂ samples - is this enough?"

When sample sizes are fixed, you need to know: **"What size difference can my test reliably detect?"**

#### Step 1: Calculate Minimum Detectable Effect Size

$$d_{\min} = \frac{Z_{1-\alpha/2} + Z_{1-\beta}}{\sqrt{n_{\text{eff}}/2}}$$

where $n_{\text{eff}} = \frac{2n_1 n_2}{n_1 + n_2}$ (harmonic mean)

For α=0.05, 80% power:

$d_{\min} \approx \frac{2.8}{\sqrt{n_{\text{eff}}}}$

**What this means:** 
- $d_{\min}$ is the **minimum detectable effect size** with 80% power
- Any effect size **larger** than $d_{\min}$ will be **even easier to detect** (higher power)
- Any effect size **smaller** than $d_{\min}$ will have **lower power** (more likely to miss it)

**In practice:**
- If your expected effect is $d = 0.6$ and $d_{\min} = 0.4$, you're good! (Your effect is larger → easier to detect)
- If your expected effect is $d = 0.3$ and $d_{\min} = 0.4$, you're underpowered (Your effect is smaller → likely to miss it)

#### Step 2: Compare to Your Expected Effect

| Your Sample Sizes | $n_{\text{eff}}$ | Min Detectable d | Can Detect |
|-------------------|------------------|------------------|------------|
| n₁=30, n₂=30 | 30 | 0.51 | Medium+ effects |
| n₁=50, n₂=50 | 50 | 0.40 | Small-medium effects |
| n₁=100, n₂=50 | 67 | 0.34 | Small-medium effects |
| n₁=100, n₂=20 | 33 | 0.49 | Medium effects only |
| n₁=200, n₂=200 | 200 | 0.20 | Small effects |

### What This Actually Means

**The problem**: Even if a real difference exists, your test might say "not significant" if you don't have enough samples. This is called being "underpowered."

**Think of it this way**: 
- You're comparing two groups
- There IS a real difference between them
- But your sample is too small, so the test says "no significant difference"
- You conclude nothing is happening, but you're WRONG

**How many samples do you need to avoid this?**

It depends on **how big the real difference is** (relative to variability):

| Your Sample Size | Real difference you'll reliably detect |
|------------------|----------------------------------------|
| **10-15 per group** | Only HUGE differences (group means differ by ≥80% of one SD) |
| **30 per group** | Large differences (group means differ by ≥50% of one SD) |
| **50 per group** | Medium differences (group means differ by ≥40% of one SD) |
| **100+ per group** | Small differences (group means differ by ≥30% of one SD) |

**Example to make this concrete:**

Say you're comparing test scores (SD = 10 points):
- **10 per group**: You'll only detect if groups differ by ≥8 points
- **30 per group**: You'll detect if groups differ by ≥5 points
- **50 per group**: You'll detect if groups differ by ≥4 points
- **100 per group**: You'll detect if groups differ by ≥3 points

**If the real difference is smaller than what your sample can detect, the test will likely say "not significant" even though a difference EXISTS.**

⚠️ **Below 10 per group**: You can only detect enormous differences. Small/medium differences will be missed, and your test is basically useless.

### Quick Check: "Is My Test Meaningful?"

**The real question is: "Will my test be able to detect the difference if it exists?"**

**YES - Your test is meaningful if:**
- You expect a LARGE difference (groups differ by ≥50% of SD) AND you have ≥30 per group
- You expect a MEDIUM difference (groups differ by ≥40% of SD) AND you have ≥50 per group  
- You expect a SMALL difference (groups differ by ≥30% of SD) AND you have ≥100 per group

**RISKY - You might miss real differences:**
- Your sample sizes are between these thresholds
- Consider your results exploratory, not definitive

**NO - Your test is likely useless:**
- You have <10 per group (unless you expect enormous differences)
- You expect a small difference but have <100 per group

**Bottom line**: More samples = able to detect smaller differences. Fewer samples = can only detect big, obvious differences.

---

## Unequal Sample Sizes (Details)

### Understanding Effective Sample Size

When group sizes differ, statistical power depends on the **harmonic mean**:

$$n_{\text{eff}} = \frac{2n_1 n_2}{n_1 + n_2}$$

This represents the "equivalent balanced sample size" in terms of statistical power.

### Important Implications

1. **Power loss with imbalance**: The effective sample size is always ≤ the average of $n_1$ and $n_2$
   
2. **Extreme example**: 
   - Balanced: $n_1 = n_2 = 50$ → $n_{\text{eff}} = 50$
   - Imbalanced: $n_1 = 80, n_2 = 20$ → $n_{\text{eff}} = 32$ (36% power loss!)
   - Imbalanced: $n_1 = 90, n_2 = 10$ → $n_{\text{eff}} = 18$ (64% power loss!)

3. **Rule**: To maximize power with fixed total N, use **equal group sizes**

### Practical Example

Suppose you want 80% power to detect $d = 0.5$ (medium effect) with α=0.05:

- **Balanced design**: $n_1 = n_2 = 63$ (Total N = 126)
- **Imbalanced 2:1**: $n_1 = 84, n_2 = 42$ (Total N = 126, but $n_{\text{eff}} = 56$, underpowered!)
- **To achieve same power with 2:1 ratio**: Need $n_1 = 95, n_2 = 47$ (Total N = 142)

### Computing Required Unequal Sample Sizes

If you must use a specific ratio $r = n_1/n_2$, and you need effective sample size $n_{\text{eff}}$:

$$n_2 = n_{\text{eff}} \cdot \frac{r+1}{2r}$$

$$n_1 = r \cdot n_2$$

**Example with 3:1 ratio and medium effect:**
- Need $n_{\text{eff}} \approx 63$ for 80% power
- With $r = 3$: $n_2 = 63 \times \frac{4}{6} = 42$, $n_1 = 126$
- Total N = 168 vs. 126 for balanced design (33% more participants needed)

---

## Rule of Thumb

**If you have no prior information:**
- Aim for **minimum 30 per group** to rely on Central Limit Theorem
- ⚠️ Warning: 30 may be too small to detect small-to-medium effects

**Should you exclude samples with <30 data points?**

**It depends on your goal:**

**YES, exclude if:**
- You're doing a **rigorous confirmatory study** where you need reliable statistical inference
- You need to detect **medium or smaller effects** (30 is borderline at best)
- You have **multiple groups** to compare and can't afford underpowered comparisons
- The cost of false negatives (missing real effects) is high

**NO, keep if:**
- You're doing **exploratory analysis** and just want to see patterns
- You expect **very large effects** (d > 0.8) - small samples can still detect these
- It's a **pilot study** or preliminary investigation
- You're willing to interpret results cautiously and follow up with larger samples
- Excluding the group would create bias (e.g., always excluding rare conditions)

**Better approach than hard cutoffs:**
1. Calculate $n_{\text{eff}}$ and $d_{\min}$ for each comparison
2. Keep groups where $d_{\min}$ is reasonable for your expected effect size
3. Report power calculations alongside results
4. Be transparent about which comparisons are underpowered

**Example:** If you have groups with n=25, n=35, n=50, n=100:
- Don't automatically exclude n=25
- Instead: Note that n=25 can only detect d≥0.56, n=35 can detect d≥0.47, etc.
- Decide based on whether these thresholds match your expected effects

---

## What You Need to Calculate Sample Size

Provide **either**:

**Option A:** Raw parameters
- Estimated standard deviation ($\sigma$)
- Minimum detectable difference ($\delta$)

**Option B:** Effect size
- Cohen's $d = \delta/\sigma$ you want to detect
  - Small: $d = 0.2$
  - Medium: $d = 0.5$
  - Large: $d = 0.8$

**Plus:**
- Desired significance level (α, typically 0.05)
- Desired power (1−β, typically 0.80 or 0.90)
- Study design (paired vs. independent)
- Number of groups/comparisons

---

## Quick Reference Table

| Desired Power | α = 0.05 | Cohen's d = 0.5 | n per group |
|---------------|----------|-----------------|-------------|
| 70% | Two-sided | Medium effect | ~52 |
| **80%** | **Two-sided** | **Medium effect** | **~63** |
| 90% | Two-sided | Medium effect | ~85 |
| 95% | Two-sided | Medium effect | ~105 |