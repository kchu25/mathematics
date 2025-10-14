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

## Workflow: Checking If Your Unequal Samples Are Adequate

### Scenario: You Already Have Data

**You have:** Group 1 with n₁ samples, Group 2 with n₂ samples

**Question:** "Do I have enough samples to detect a meaningful difference?"

> **⚠️ Common Confusion: Planning vs. Checking**
> 
> **Two different scenarios:**
> 
> **SCENARIO A - PLANNING (before data collection):**
> - Question: "How many samples do I need to collect?"
> - Use formula: $n = 15.68/d^2$ to calculate required sample size
> - You decide δ and σ beforehand → get d → calculate n
> 
> **SCENARIO B - CHECKING (after data collection):**
> - Question: "I already have n₁ and n₂ samples - are they enough?"
> - Calculate: $d_{\min} = 2.8/\sqrt{n_{\text{eff}}}$ to find what you CAN detect
> - Compare d_min to your expected effect size d
> - **This is the workflow below!**
> 
> **Key difference:** 
> - Planning: δ, σ → d → **n** (you're solving for sample size)
> - Checking: n₁, n₂ → n_eff → **d_min** (you're solving for detectable effect)

### Step-by-Step Workflow

#### **Step 1: Calculate Effective Sample Size**

$$n_{\text{eff}} = \frac{2n_1 n_2}{n_1 + n_2}$$

**Example:** n₁ = 45, n₂ = 30
$$n_{\text{eff}} = \frac{2 \times 45 \times 30}{45 + 30} = \frac{2700}{75} = 36$$

---

#### **Step 2: Calculate Pooled Standard Deviation**

From your actual data, calculate the standard deviations s₁ and s₂ for each group, then:

$$\sigma_{\text{pooled}} = \sqrt{\frac{(n_1-1)s_1^2 + (n_2-1)s_2^2}{n_1 + n_2 - 2}}$$

**Example:** n₁=45, s₁=8.5; n₂=30, s₂=9.2
$$\sigma_{\text{pooled}} = \sqrt{\frac{44 \times 72.25 + 29 \times 84.64}{73}} = \sqrt{\frac{5633.56}{73}} \approx 8.78$$

---

#### **Step 3: Decide Your Minimum Meaningful Difference (δ)**

**This is subjective and domain-specific!**

Ask yourself: "What's the smallest difference that would matter in practice?"

**Example:** Comparing test scores
- If a 2-point difference doesn't matter clinically/practically → δ = 2 is too small
- If a 5-point difference is meaningful → set δ = 5

---

#### **Step 4: Calculate Expected Cohen's d**

$$d = \frac{\delta}{\sigma_{\text{pooled}}}$$

**Example:** δ = 5, σ = 8.78
$$d = \frac{5}{8.78} \approx 0.57$$

This is a medium-to-large effect.

---

#### **Step 5: Calculate Minimum Detectable Cohen's d**

This tells you what your current sample can actually detect:

$d_{\min} = \frac{Z_{1-\alpha/2} + Z_{1-\beta}}{\sqrt{n_{\text{eff}}/2}}$

For α=0.05, 80% power, this simplifies to:

$d_{\min} = \frac{2.8}{\sqrt{n_{\text{eff}}}}$

> **📊 Where Does the 2.8 Come From?**
> 
> The coefficient 2.8 comes from:
> $Z_{1-\alpha/2} + Z_{1-\beta} = 1.96 + 0.84 = 2.80$
> 
> **Assumptions built into this coefficient:**
> - **α = 0.05** (two-sided test, 5% significance level)
> - **Power = 80%** (β = 0.20, so 20% chance of Type II error)
> 
> **For different assumptions, use different coefficients:**
> 
> | α (two-sided) | Power | $Z_{1-\alpha/2}$ | $Z_{1-\beta}$ | Coefficient |
> |---------------|-------|------------------|---------------|-------------|
> | 0.10 | 70% | 1.645 | 0.524 | **2.17** |
> | 0.05 | 70% | 1.960 | 0.524 | **2.48** |
> | **0.05** | **80%** | **1.960** | **0.842** | **2.80** |
> | 0.05 | 90% | 1.960 | 1.282 | **3.24** |
> | 0.05 | 95% | 1.960 | 1.645 | **3.61** |
> | 0.01 | 80% | 2.576 | 0.842 | **3.42** |
> | 0.01 | 90% | 2.576 | 1.282 | **3.86** |
> 
> **Example:** For 90% power with α=0.05:
> $d_{\min} = \frac{3.24}{\sqrt{n_{\text{eff}}}}$
> 
> **Key insight:** Higher power or lower α → larger coefficient → larger d_min → need more samples to achieve that power/significance level.

**Example:** n_eff = 36, using standard assumptions (α=0.05, 80% power)
$d_{\min} = \frac{2.8}{\sqrt{36}} = \frac{2.8}{6} \approx 0.47$

---

#### **Reference Table: Minimum Detectable d for Common Sample Sizes**

Using standard assumptions (α=0.05, 80% power, two-sided):

| n_eff | d_min | Interpretation |
|-------|-------|----------------|
| 10 | 0.89 | Very large effects only |
| 15 | 0.72 | Large effects only |
| 20 | 0.63 | Large effects |
| 25 | 0.56 | Medium-large effects |
| 30 | 0.51 | Medium effects |
| 40 | 0.44 | Medium effects |
| 50 | 0.40 | Small-medium effects |
| 64 | 0.35 | Small-medium effects |
| 80 | 0.31 | Small-medium effects |
| 100 | 0.28 | Small effects |
| 150 | 0.23 | Small effects |
| 200 | 0.20 | Small effects |
| 300 | 0.16 | Very small effects |
| 400 | 0.14 | Very small effects |

**How to use this table:**
1. Calculate your n_eff from your actual sample sizes
2. Find the closest n_eff in the table
3. Check if d_min is reasonable for your expected effect
4. If your expected d > d_min → you're good!
5. If your expected d < d_min → underpowered

---

#### **Step 6: Compare and Decide**

**Decision Rule:**

| Comparison | Interpretation | Action |
|------------|----------------|--------|
| **d (expected) > d_min** | ✓ You have enough power | Proceed with analysis |
| **d (expected) ≈ d_min** | ⚠️ Borderline power (~80%) | Proceed but note limitations |
| **d (expected) < d_min** | ✗ Underpowered | Results unreliable; need more data |

**In our example:**
- Expected d = 0.57
- Minimum detectable d_min = 0.47
- **0.57 > 0.47** → ✓ **You have adequate power!**

> **✍️ How to Report Your Power Analysis**
> 
> **After confirming adequate power, you should report this in your analysis. Here are standard ways to phrase it:**
> 
> **Option 1 - Simple statement:**
> "The sample means were compared using an independent samples t-test with 80% power to detect a medium effect size (d = 0.5) at α = 0.05."
> 
> **Option 2 - More detailed:**
> "Given sample sizes of n₁ = 45 and n₂ = 30 (effective n = 36), this study had 80% power to detect an effect size of d ≥ 0.47 at the α = 0.05 significance level."
> 
> **Option 3 - Full reporting (recommended for publications):**
> "A post-hoc power analysis indicated that with n₁ = 45 and n₂ = 30, the study achieved 80% power (1-β = 0.80) to detect a minimum effect size of Cohen's d = 0.47 at α = 0.05 (two-tailed). The expected effect size based on prior literature was d = 0.57, confirming adequate statistical power."
> 
> **What NOT to say:**
> ❌ "The test was significant with p < 0.05" (this doesn't mention power)
> ❌ "We had enough samples" (too vague)
> ❌ "Post-hoc power was 95%" after seeing significant results (this is circular reasoning - see note below)
> 
> **Important considerations:**
> 
> **1. A priori vs. post-hoc power:**
> - **A priori (before seeing results)**: Calculate power based on expected effect → GOOD
> - **Post-hoc using observed effect**: Calculate power after seeing your result → PROBLEMATIC (gives inflated sense of confidence)
> - **Post-hoc using n_eff and d_min**: Calculate what you COULD detect with your samples → GOOD (this is what we did above)
> 
> **2. If you find "not significant":**
> - Report power analysis to show whether you had adequate power to detect effects
> - Example: "No significant difference was found (p = 0.12). However, with n_eff = 36, the study only had 80% power to detect effects of d ≥ 0.47. Smaller effects may exist but were not detectable with this sample size."
> 
> **3. If you're underpowered:**
> - Be honest about limitations
> - Example: "The sample sizes (n₁ = 15, n₂ = 12, n_eff = 13) provided only 80% power to detect very large effects (d ≥ 0.77). Results should be interpreted cautiously as smaller but meaningful effects may have been missed."

---

### Alternative: If You Don't Know δ Yet

**If you haven't decided what difference matters**, reverse the calculation:

#### Calculate what difference your sample CAN detect:

$$\delta_{\min} = d_{\min} \times \sigma_{\text{pooled}} = \frac{2.8}{\sqrt{n_{\text{eff}}}} \times \sigma_{\text{pooled}}$$

**Example:** n_eff = 36, σ = 8.78
$$\delta_{\min} = 0.47 \times 8.78 \approx 4.1$$

**Interpretation:** "With my current sample sizes, I can detect differences of 4.1 points or larger with 80% power."

**Then ask:** "Is a 4.1-point difference meaningful in my field?"
- If YES → proceed
- If NO (you need to detect smaller differences) → need more samples

---

### Quick Reference Workflow Chart

```
START: You have n₁ and n₂ samples
    ↓
[1] Calculate n_eff = 2n₁n₂/(n₁+n₂)
    ↓
[2] Calculate σ_pooled from your data
    ↓
[3] Decide: What's the minimum meaningful difference (δ)?
    ↓
[4] Calculate expected d = δ/σ_pooled
    ↓
[5] Calculate d_min = 2.8/√n_eff
    ↓
[6] Compare: Is d > d_min?
    ↓
YES: ✓ Adequate power → Run your t-test
NO: ✗ Underpowered → Collect more data or reconsider
```

---

### Common Mistakes to Avoid

❌ **WRONG:** "I'll calculate n_eff and use the formula n = 15.68/d²"
- That formula is for **planning** sample size, not checking existing samples

✓ **CORRECT:** Use d_min = 2.8/√n_eff to find what you **can** detect, then compare to what you **want** to detect

❌ **WRONG:** Setting δ based on what you observe in your data
- This is circular reasoning and invalidates your test

✓ **CORRECT:** Set δ based on prior knowledge, clinical significance, or practical importance **before** looking at your results

---

## The Formula (For Planning Sample Size)

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

$$d_{\min} \approx \frac{2.8}{\sqrt{n_{\text{eff}}}}$$

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