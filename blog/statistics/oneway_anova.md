@def title = "One-Way ANOVA: Mathematical Framework & Workflow"
@def published = "14 October 2025"
@def tags = ["statistics"]

# One-Way ANOVA: Mathematical Framework & Workflow

## Setup and Notation

We have **k groups** (treatments) with potentially different sample sizes:
- Group 1: $n_1$ observations
- Group 2: $n_2$ observations
- ...
- Group k: $n_k$ observations

**Total sample size:** $N = n_1 + n_2 + ... + n_k$

**Data structure:**
- $y_{ij}$ = the $j$-th observation in the $i$-th group
- $i = 1, 2, ..., k$ (group index)
- $j = 1, 2, ..., n_i$ (observation index within group $i$)

## The ANOVA Model

$$y_{ij} = \mu_i + \epsilon_{ij}$$

where:
- $\mu_i$ = true mean of group $i$
- $\epsilon_{ij} \sim N(0, \sigma^2)$ = random error (assumed independent, normally distributed with common variance)

## Hypotheses

**Null hypothesis:** $H_0: \mu_1 = \mu_2 = ... = \mu_k$ (all group means are equal)

**Alternative hypothesis:** $H_a:$ at least one $\mu_i$ differs from the others

## Step-by-Step Calculation Workflow

### Step 1: Calculate Group Means

For each group $i$:

$$\bar{y}_i = \frac{1}{n_i}\sum_{j=1}^{n_i} y_{ij}$$

### Step 2: Calculate Grand Mean

$$\bar{y}_{..} = \frac{1}{N}\sum_{i=1}^{k}\sum_{j=1}^{n_i} y_{ij} = \frac{1}{N}\sum_{i=1}^{k} n_i\bar{y}_i$$

### Step 3: Calculate Sum of Squares

**Total Sum of Squares (SST):** Measures total variation in the data

$$SST = \sum_{i=1}^{k}\sum_{j=1}^{n_i} (y_{ij} - \bar{y}_{..})^2$$

**Between-Groups Sum of Squares (SSB):** Measures variation *between* group means

$$SSB = \sum_{i=1}^{k} n_i(\bar{y}_i - \bar{y}_{..})^2$$

**Within-Groups Sum of Squares (SSW):** Measures variation *within* groups

$$SSW = \sum_{i=1}^{k}\sum_{j=1}^{n_i} (y_{ij} - \bar{y}_i)^2$$

**Key relationship:**
$$SST = SSB + SSW$$

### Step 4: Calculate Degrees of Freedom

- Between groups: $df_B = k - 1$
- Within groups: $df_W = N - k$
- Total: $df_T = N - 1$

Note: $df_T = df_B + df_W$

### Step 5: Calculate Mean Squares

**Mean Square Between (MSB):**

$$MSB = \frac{SSB}{df_B} = \frac{SSB}{k-1}$$

**Mean Square Within (MSW):**

$$MSW = \frac{SSW}{df_W} = \frac{SSW}{N-k}$$

Note: MSW is the **pooled variance estimate** across all groups.

### Step 6: Calculate F-Statistic

$$F = \frac{MSB}{MSW}$$

Under $H_0$, this follows an F-distribution with $(k-1, N-k)$ degrees of freedom.

**Intuition:** 
- If $H_0$ is true, both MSB and MSW estimate $\sigma^2$, so F ≈ 1
- If $H_0$ is false, MSB will be larger than MSW, so F > 1

### Step 7: Determine P-value

$$p\text{-value} = P(F_{k-1, N-k} \geq F_{observed})$$

**Decision rule:**
- If p-value < $\alpha$ (typically 0.05): Reject $H_0$ → At least one group mean differs
- If p-value ≥ $\alpha$: Fail to reject $H_0$ → No significant difference detected

## ANOVA Table Summary

| Source | Sum of Squares | df | Mean Square | F-statistic | p-value |
|--------|----------------|-------|-------------|-------------|---------|
| Between Groups | SSB | $k-1$ | $MSB = \frac{SSB}{k-1}$ | $F = \frac{MSB}{MSW}$ | $P(F_{k-1,N-k} \geq F)$ |
| Within Groups | SSW | $N-k$ | $MSW = \frac{SSW}{N-k}$ | | |
| Total | SST | $N-1$ | | | |

## Worked Example

**Data:** Three teaching methods with different sample sizes

- Method A: [85, 88, 90, 87] → $n_1 = 4$
- Method B: [78, 82, 80] → $n_2 = 3$
- Method C: [92, 95, 93, 91, 94] → $n_3 = 5$

Total: $N = 12$, $k = 3$

### Calculations:

**Group means:**
- $\bar{y}_1 = \frac{85+88+90+87}{4} = 87.5$
- $\bar{y}_2 = \frac{78+82+80}{3} = 80$
- $\bar{y}_3 = \frac{92+95+93+91+94}{5} = 93$

**Grand mean:**
$$\bar{y}_{..} = \frac{4(87.5) + 3(80) + 5(93)}{12} = \frac{350 + 240 + 465}{12} = \frac{1055}{12} = 87.92$$

**SSB:**
$$SSB = 4(87.5-87.92)^2 + 3(80-87.92)^2 + 5(93-87.92)^2$$
$$= 4(0.1764) + 3(62.7264) + 5(25.8064) = 0.71 + 188.18 + 129.03 = 317.92$$

**SSW:** (calculation for each observation)
$$SSW = [(85-87.5)^2 + (88-87.5)^2 + ... + (94-93)^2] = 70.5$$

**SST:**
$$SST = SSB + SSW = 317.92 + 70.5 = 388.42$$

**Degrees of freedom:**
- $df_B = 3 - 1 = 2$
- $df_W = 12 - 3 = 9$

**Mean squares:**
- $MSB = \frac{317.92}{2} = 158.96$
- $MSW = \frac{70.5}{9} = 7.83$

**F-statistic:**
$$F = \frac{158.96}{7.83} = 20.30$$

**Conclusion:** With $F(2,9) = 20.30$, we get $p < 0.001$, strongly rejecting $H_0$. At least one teaching method produces significantly different results.

## Assumptions to Check

1. **Independence:** Observations are independent within and across groups
2. **Normality:** Data within each group are approximately normally distributed (can check with Q-Q plots or Shapiro-Wilk test)
3. **Homogeneity of variance:** All groups have equal variance (check with Levene's test or examine ratio of largest to smallest variance < 3:1)

## Effect Size: Eta-Squared ($\eta^2$)

Measures the proportion of total variance explained by group membership:

$$\eta^2 = \frac{SSB}{SST}$$

**Interpretation:**
- 0.01 = small effect
- 0.06 = medium effect
- 0.14 = large effect

In our example: $\eta^2 = \frac{317.92}{388.42} = 0.82$ (very large effect)

## Post-Hoc Tests (If ANOVA is Significant)

Common methods to identify which specific groups differ:
- **Tukey's HSD:** Controls family-wise error rate, good for all pairwise comparisons
- **Bonferroni:** Conservative, divides $\alpha$ by number of comparisons
- **Scheffé:** Most conservative, good for complex comparisons

These tests compare: $|\bar{y}_i - \bar{y}_j|$ against a critical value adjusted for multiple comparisons.