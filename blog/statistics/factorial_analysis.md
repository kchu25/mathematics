@def title = "Method 2: Factorial Design Analysis - Step by Step"
@def published = "8 January 2026"
@def tags = ["statistics"]

# Method 2: Factorial Design Analysis - Step by Step

## What You're Testing

You've got three groups: x1 alone, x2 alone, and both together. You're asking: **"Is there something special happening when x1 and x2 are combined, beyond just their individual effects?"**

The interaction term (β₃) captures this "extra" effect.

---

## Step 1: Set Up Your Data

First, code your groups with binary indicators:

| Group | x1 | x2 |
|-------|----|----|
| x1 only | 1 | 0 |
| x2 only | 0 | 1 |
| Both | 1 | 1 |

Each row in your dataset should have:
- Your outcome measurement (Y)
- An x1 indicator (1 if x1 is present, 0 if not)
- An x2 indicator (1 if x2 is present, 0 if not)

---

## Step 2: Create the Interaction Term

Calculate x1 × x2 for each observation:
- x1 only group: 1 × 0 = 0
- x2 only group: 0 × 1 = 0
- Both group: 1 × 1 = 1

This interaction term is zero unless both factors are present.

---

## Step 3: Fit the Linear Model

Run a linear regression with this model:

$Y = \beta_0 + \beta_1 x_1 + \beta_2 x_2 + \beta_3 (x_1 \times x_2) + \epsilon$

Where:
- $\beta_0$ = baseline (intercept)
- $\beta_1$ = effect of x1 alone
- $\beta_2$ = effect of x2 alone
- $\beta_3$ = **interaction effect** (the extra bit when both are present)
- $\epsilon$ = random error

---

## Step 4: Understand What Each β Means

**When $x_1 = 1, x_2 = 0$ (x1 only):**  
$Y = \beta_0 + \beta_1$

**When $x_1 = 0, x_2 = 1$ (x2 only):**  
$Y = \beta_0 + \beta_2$

**When $x_1 = 1, x_2 = 1$ (both together):**  
$Y = \beta_0 + \beta_1 + \beta_2 + \beta_3$

That **$\beta_3$** is the key. It's the deviation from simple addition:
- If $\beta_3 = 0$ → perfectly additive (no interaction)
- If $\beta_3 > 0$ → synergy (1+1=3)
- If $\beta_3 < 0$ → antagonism (1+1=1.5)

---

## Step 5: Look at the t-test for β₃

Your regression output will include a t-test for each coefficient. Find the row for the interaction term $(x_1 \times x_2)$.

Check the p-value:
- **p < 0.05**: The interaction is statistically significant
- **p ≥ 0.05**: No evidence of interaction (effects are additive)

The t-statistic tells you how many standard errors $\beta_3$ is from zero. Larger absolute values = stronger evidence of interaction.

---

## Step 6: Interpret the Results

**If p < 0.05 (significant interaction):**

Look at the **sign of $\beta_3$**:
- **Positive $\beta_3$**: Synergistic effect
  - The combined effect is greater than the sum of individual effects
  - Example: $\beta_3 = +3$ means the combo gives 3 units MORE than just adding x1 and x2
  
- **Negative $\beta_3$**: Antagonistic effect
  - The combined effect is less than the sum of individual effects
  - Example: $\beta_3 = -2$ means the combo gives 2 units LESS than expected

**If p ≥ 0.05 (no significant interaction):**
- The effects are approximately additive
- $Y_{both} \approx Y_{x1} + Y_{x2}$ (within random noise)

---

## Step 7: Check Your Model Assumptions

Before trusting your results, verify:

**1. Linearity**: The relationship between predictors and outcome is linear
- Plot residuals vs. fitted values (should look random)

**2. Independence**: Observations are independent of each other
- No repeated measurements without accounting for them

**3. Normality**: Residuals are approximately normally distributed
- QQ plot should be roughly straight
- More important with small samples

**4. Equal variance** (homoscedasticity): Error variance is constant
- Residuals shouldn't fan out in plots

If assumptions are violated, consider transforming your outcome variable or using robust regression methods.

---

## Step 8: Report Your Findings

Include these elements:

**Effect size**: Report β₃ with confidence intervals
- Example: "β₃ = 4.2 (95% CI: 1.8, 6.6)"

**Statistical significance**: Report the p-value
- Example: "t(87) = 3.45, p = 0.001"

**Direction**: State whether it's synergistic or antagonistic
- Example: "The interaction was significantly synergistic"

**Context**: Relate it back to the biological/practical meaning
- Example: "The combined treatment produced 4.2 units more improvement than expected from individual treatments alone"

---

## Important Notes

**If x1 and x2 are continuous variables** (not separate experimental groups):
- This approach is still correct and standard
- Consider **centering** your variables (subtract the mean) before creating the interaction term to reduce multicollinearity
- Be aware that interaction effects typically require **4× the sample size** to detect compared to main effects of the same magnitude

**Sample size matters**:
- Detecting interactions requires more data than detecting main effects
- With N < 50, you can only reliably detect very large interactions (d ≥ 0.8)
- For medium interactions (d ≈ 0.5), aim for N ≥ 100-150

**Multicollinearity check**:
- If x1 and x2 are correlated, standard errors may inflate
- Check variance inflation factors (VIF < 10 is generally okay)
- Centering variables helps reduce this issue

---

## Quick Troubleshooting

**"My interaction term isn't significant, but I expected one"**
- Check your sample size - you might be underpowered
- Verify your data coding is correct
- Consider if your expected effect size is realistic

**"All my coefficients are correlated"**
- Try centering your x1 and x2 variables
- This often improves interpretation without changing the interaction test

**"My residuals look weird"**
- Check for outliers
- Consider transforming Y (log, sqrt)
- Try robust regression methods

---

## Julia Code Example

Here's how to run this analysis in Julia:

```julia
using GLM, DataFrames, Statistics

# Create your data
# Assume you have measurements y for each group
data = DataFrame(
    y = [y_x1_only..., y_x2_only..., y_both...],  # your measurements
    x1 = [ones(n1)..., zeros(n2)..., ones(n_both)...],
    x2 = [zeros(n1)..., ones(n2)..., ones(n_both)...]
)

# Fit the linear model with interaction
model = lm(@formula(y ~ x1 + x2 + x1&x2), data)

# Display results
println(model)

# Extract the interaction coefficient and p-value
β3 = coef(model)[4]  # interaction term coefficient
p_value = coeftable(model).cols[4][4]  # p-value for interaction

println("\nInteraction effect (β₃): ", round(β3, digits=3))
println("P-value: ", round(p_value, digits=4))

# Interpret the result
if p_value < 0.05
    if β3 > 0
        println("Significant SYNERGISTIC interaction detected!")
    else
        println("Significant ANTAGONISTIC interaction detected!")
    end
else
    println("No significant interaction - effects are additive")
end

# Get confidence intervals
ci = confint(model)
println("\n95% CI for β₃: [", round(ci[4,1], digits=3), ", ", 
        round(ci[4,2], digits=3), "]")
```

**Example with concrete data:**

```julia
using GLM, DataFrames

# Example: Drug interaction study
# x1 only group (n=10): drug A alone
y_x1 = [12.3, 14.1, 13.5, 15.2, 12.8, 14.6, 13.9, 15.1, 13.2, 14.3]

# x2 only group (n=10): drug B alone  
y_x2 = [11.8, 13.2, 12.5, 13.9, 12.1, 13.7, 12.9, 13.5, 12.3, 13.1]

# Both group (n=10): drugs A + B together
y_both = [28.5, 31.2, 29.8, 32.1, 28.9, 30.5, 29.3, 31.8, 29.1, 30.2]

# Combine into dataframe
data = DataFrame(
    y = vcat(y_x1, y_x2, y_both),
    x1 = vcat(ones(10), zeros(10), ones(10)),
    x2 = vcat(zeros(10), ones(10), ones(10))
)

# Fit model
model = lm(@formula(y ~ x1 + x2 + x1&x2), data)
println(model)

# The output will show:
# β₀ ≈ 11.8 (baseline)
# β₁ ≈ 1.6 (effect of drug A alone)
# β₂ ≈ 1.3 (effect of drug B alone)  
# β₃ ≈ 16.2 (interaction - the extra effect!)
```

**What you're testing:**
- Null hypothesis: $\beta_3 = 0$ (no interaction, perfectly additive)
- Alternative: $\beta_3 \neq 0$ (synergy or antagonism)

In this example, you'd expect $Y_{both} = 11.8 + 1.6 + 1.3 = 14.7$ if additive, but you actually observe ~30. That extra ~16 units is your synergistic interaction!

---

## The Bottom Line

This method tests the same thing as Method 1 (direct comparison), just dressed up in regression language. The benefit? You get standard errors, confidence intervals, and p-values automatically. Plus, you can easily add covariates if needed later.