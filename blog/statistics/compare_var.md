@def title = "Comparing the Spread of Two Subpopulations (Context-Aware)"
@def published = "12 February 2026"
@def tags = ["statistics"]

# Comparing the "Spread" of Two Subpopulations (Context-Aware)

## The Real Question

You don't just want to know if two subpopulations are at the same location—you want to know if they have **similar variability** relative to their parent populations.

**Example:** 
- Population A has a subgroup with SD = 15
- Population B has a subgroup with SD = 20

But Population A itself has SD = 100, while Population B has SD = 50.

**Which subpopulation is more "spread out" in context?**

---

## The Setup

After standardizing to Z-scores, you now have:
- **Subpopulation A:** Z-scores with variance $s_A^2$
- **Subpopulation B:** Z-scores with variance $s_B^2$

**Key insight:** If both subpopulations are "random samples" from their parent populations, their Z-score variances should both be ≈ 1.0 (since the parent populations have variance = 1 after standardization).

**Deviations from 1.0** tell you something interesting:
- $s_Z^2 < 1$: The subpopulation is more homogeneous than the parent (clustered)
- $s_Z^2 > 1$: The subpopulation is more heterogeneous than the parent (dispersed)
- $s_Z^2 \approx 1$: The subpopulation mirrors the parent's spread

---

## Rigorous Test 1: F-test for Equality of Variances

**Null Hypothesis:** $\sigma_A^2 = \sigma_B^2$ (the two subpopulations have equal spread)

**Test Statistic:**

$$F = \frac{s_A^2}{s_B^2}$$

where $s_A^2$ and $s_B^2$ are the sample variances of the Z-scores.

**Degrees of freedom:** $df_1 = n_A - 1$, $df_2 = n_B - 1$

**Decision:**
- If $F$ is close to 1, the spreads are similar
- If $F$ is significantly different from 1, they differ

### Limitations
- **Sensitive to non-normality**: The F-test assumes normal distributions
- **Two-tailed testing is tricky**: You need to check both tails

---

## Rigorous Test 2: Levene's Test (More Robust)

Levene's test is the **robust alternative** when you're not sure about normality.

**How it works:**
1. Calculate the absolute deviations from each group's median: $|z_i - \text{median}(z)|$
2. Run a standard **t-test** on these deviations

**Null Hypothesis:** The spreads are equal

**Why it's better:** 
- Less sensitive to outliers
- Doesn't assume perfect normality
- Standard output from most statistical packages

---

## Rigorous Test 3: Variance Ratio Confidence Interval

Instead of just testing "are they different," you can estimate **how much** they differ.

**Confidence Interval for the Ratio:**

$$\left(\frac{s_A^2}{s_B^2} \cdot \frac{1}{F_{\alpha/2}}, \quad \frac{s_A^2}{s_B^2} \cdot F_{\alpha/2}\right)$$

where $F_{\alpha/2}$ is the critical value from the F-distribution.

**Interpretation:**
- If the CI includes 1.0: No evidence of different spreads
- If the CI is [0.9, 1.1]: Strong evidence they're practically the same
- If the CI is [0.5, 2.0]: Inconclusive (wide interval, need more data)

---

## The Coefficient of Variation (Relative Spread)

For a single subpopulation, you can compute:

$$CV_Z = \frac{s_Z}{|\bar{z}|}$$

This measures "spread relative to position." But **be careful**:
- Only meaningful if $\bar{z} \neq 0$
- Sensitive to small denominators

Generally, **just compare the variances directly** rather than CV for Z-scores.

---

## Visual Comparison: The "Spread Plot"

Create overlaid density plots or boxplots:

```
       Population A Context          Population B Context
       (SD = 1.0)                    (SD = 1.0)
           |                              |
    Subpop A (s = 0.8)            Subpop B (s = 1.2)
    [narrower distribution]       [wider distribution]
```

**What to show:**
1. Background shading at $\pm 1$ SD (the "normal" spread)
2. Overlay each subpopulation's distribution
3. Annotate with $s_A$ and $s_B$ values

If both distributions have similar "width" on the standardized axis, they have similar context-adjusted spread.

---

## Practical Workflow

### Step 1: Compute Z-score Variances
```
Group A: s²_A = var(z_scores_A) = 0.64  →  SD = 0.8
Group B: s²_B = var(z_scores_B) = 1.44  →  SD = 1.2
```

### Step 2: Run Levene's Test
```
Levene's test: F(1, 63) = 8.2, p = 0.006
```
**Conclusion:** The spreads are significantly different.

### Step 3: Compute Variance Ratio
```
Ratio = s²_A / s²_B = 0.64 / 1.44 = 0.44
95% CI for ratio: [0.25, 0.79]
```
**Interpretation:** Group A is consistently 2-4× less spread out than Group B.

### Step 4: Report Effect Size
```
Cohen's d for variance difference ≈ 0.8 (medium-large effect)
```

---

## Example Write-Up

> "To compare the context-adjusted variability of both subpopulations, we computed the variance of their standardized scores. Subpopulation A exhibited significantly lower spread ($s_A = 0.8$) than Subpopulation B ($s_B = 1.2$), Levene's test $F(1, 63) = 8.2, p = 0.006$. The variance ratio (95% CI: [0.25, 0.79]) suggests that Subpopulation A is 2–4 times more homogeneous relative to its parent population than Subpopulation B is to its own. This indicates different selective mechanisms or constraints operating in each context."

---

## Testing Against a "Null" Spread (Variance = 1)

Sometimes you want to know: **"Is this subpopulation more/less spread out than the parent population?"**

**One-sample test:**

$$\chi^2 = \frac{(n-1) s_Z^2}{\sigma_0^2}$$

where $\sigma_0^2 = 1$ (the expected variance if the subpopulation is just a random sample).

**Example:**
- You find $s_Z^2 = 0.64$ with $n = 30$
- $\chi^2 = 29 \times 0.64 / 1 = 18.56$
- Compare to $\chi^2(29)$ distribution
- If $p < 0.05$: The subpopulation is significantly **less variable** than the parent

---

## Quick Decision Tree

```
What do you want to know?

├─ Are the two subpopulations equally spread?
│  └─ Use: Levene's test or F-test
│
├─ How different are their spreads?
│  └─ Use: Variance ratio with 95% CI
│
└─ Is a single subpopulation more/less spread than its parent?
   └─ Use: Chi-squared test (variance = 1?)
```

---

## Key Takeaways

1. **Compare variances of Z-scores**, not raw variances
2. **Use Levene's test** for robustness (or F-test if you trust normality)
3. **Report variance ratios and CIs**, not just p-values
4. **Visualize** with overlaid density plots on the standardized scale
5. A variance $\neq 1$ tells you the subpopulation is **selected differently** than the parent

The variance of Z-scores is the purest measure of "context-adjusted spread."