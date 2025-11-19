@def title = "Hoeffding Bound for Banzhaf with Noisy Player Values"
@def published = "19 November 2025"
@def tags = ["general-topics"]

# Hoeffding Bound for Banzhaf with Noisy Player Values

## Problem Setup

**Given:**
- True player values $v_1^*, v_2^*, \ldots, v_n^*$ (unknown)
- Observed player values $v_1, v_2, \ldots, v_n$ (known, but noisy)
- Error bound: $\left|\sum_{i=1}^n v_i - \sum_{i=1}^n v_i^*\right| \leq \epsilon_1$
- Want to approximate Banzhaf power index by sampling

**Question:** How does the noise in player values affect the error bound when approximating Banzhaf?

---

## Key Insight

The error in approximating Banzhaf comes from **two independent sources**:

1. **Value noise**: Using noisy $v_i$ instead of true $v_i^*$
2. **Sampling error**: Using finite samples instead of all $2^{n-1}$ coalitions

These errors **compound**, and we need to bound both.

---

## Notation

- **True Banzhaf index**: $\beta_i^* = \frac{1}{2^{n-1}} \sum_{S \subseteq N \setminus \{i\}} [v^*(S \cup \{i\}) - v^*(S)]$
- **Noisy Banzhaf index**: $\beta_i = \frac{1}{2^{n-1}} \sum_{S \subseteq N \setminus \{i\}} [v(S \cup \{i\}) - v(S)]$
- **Sampled estimator**: $\hat{\beta}_i = \frac{1}{m} \sum_{j=1}^{m} [v(S_j \cup \{i\}) - v(S_j)]$

where $v(S) = \sum_{k \in S} v_k$ and $v^*(S) = \sum_{k \in S} v_k^*$.

---

## Error Decomposition

By triangle inequality:
$$|\hat{\beta}_i - \beta_i^*| \leq |\hat{\beta}_i - \beta_i| + |\beta_i - \beta_i^*|$$

**Two error terms:**
1. $|\hat{\beta}_i - \beta_i|$ = sampling error (using noisy values)
2. $|\beta_i - \beta_i^*|$ = value noise bias

---

## Part 1: Bounding Value Noise Bias

**Theorem 1:** If $\left|\sum_{i=1}^n v_i - \sum_{i=1}^n v_i^*\right| \leq \epsilon_1$, then:
$$|\beta_i - \beta_i^*| \leq \epsilon_1$$

**Proof:**

For any coalition $S$:
$$v(S) - v^*(S) = \sum_{k \in S} v_k - \sum_{k \in S} v_k^* = \sum_{k \in S} (v_k - v_k^*)$$

Therefore, for any marginal contribution:
$$[v(S \cup \{i\}) - v(S)] - [v^*(S \cup \{i\}) - v^*(S)]$$
$$= [v(S \cup \{i\}) - v^*(S \cup \{i\})] - [v(S) - v^*(S)]$$
$$= \sum_{k \in S \cup \{i\}} (v_k - v_k^*) - \sum_{k \in S} (v_k - v_k^*)$$
$$= v_i - v_i^*$$

Now, the difference in Banzhaf indices:
$$\beta_i - \beta_i^* = \frac{1}{2^{n-1}} \sum_{S \subseteq N \setminus \{i\}} [(v_i - v_i^*)]$$
$$= \frac{1}{2^{n-1}} \cdot 2^{n-1} \cdot (v_i - v_i^*)$$
$$= v_i - v_i^*$$

Therefore:
$$|\beta_i - \beta_i^*| = |v_i - v_i^*| \leq \sum_{j=1}^n |v_j - v_j^*| \leq \left|\sum_{j=1}^n (v_j - v_j^*)\right| = \epsilon_1$$

**Wait, this bound is too loose!** We can do better.

### Improved Bound

Since $\beta_i - \beta_i^* = v_i - v_i^*$, we actually have:
$$|\beta_i - \beta_i^*| = |v_i - v_i^*|$$

**But we only know the sum has bounded error, not individual values!**

In the worst case, all error could be in one player:
$$|\beta_i - \beta_i^*| \leq \epsilon_1$$

In typical cases with distributed error:
$$|\beta_i - \beta_i^*| \ll \epsilon_1$$

$\square$

---

## Part 2: Bounding Sampling Error

**Theorem 2:** When sampling $m$ coalitions using noisy values $v_i$, with probability at least $1 - \delta$:
$$|\hat{\beta}_i - \beta_i| \leq (v_{max} - v_{min})\sqrt{\frac{\ln(2/\delta)}{2m}}$$

where $v_{max}$ and $v_{min}$ are bounds on the noisy coalition values.

**Proof:**

This is standard Hoeffding as before. Define:
$$X_j = v(S_j \cup \{i\}) - v(S_j)$$

These are i.i.d. with $\mathbb{E}[X_j] = \beta_i$ (the noisy Banzhaf index).

Since $v(S) \in [v_{min}, v_{max}]$, we have $X_j \in [-(v_{max} - v_{min}), v_{max} - v_{min}]$.

By Hoeffding's inequality:
$$\Pr[|\hat{\beta}_i - \beta_i| \geq \epsilon_2] \leq 2\exp\left(-\frac{2m\epsilon_2^2}{(2(v_{max} - v_{min}))^2}\right)$$

Setting the right side to $\delta$ and solving:
$$\epsilon_2 = (v_{max} - v_{min})\sqrt{\frac{\ln(2/\delta)}{2m}}$$

$\square$

---

## Combined Error Bound

**Main Result:**

Given noisy player values with $\left|\sum_{i=1}^n v_i - \sum_{i=1}^n v_i^*\right| \leq \epsilon_1$, and sampling $m$ coalitions, with probability at least $1 - \delta$:

$$|\hat{\beta}_i - \beta_i^*| \leq \epsilon_1 + (v_{max} - v_{min})\sqrt{\frac{\ln(2/\delta)}{2m}}$$

**Interpretation:**
- **Bias term** $\epsilon_1$: unavoidable error from noisy values (worst-case per player)
- **Variance term** $(v_{max} - v_{min})\sqrt{\frac{\ln(2/\delta)}{2m}}$: sampling error, decreases with $m$

**Key observation:** The bias $\epsilon_1$ is actually just $|v_i - v_i^*|$, so if your constraint on the sum is tight and errors are distributed across players, the actual bias for any single player could be much smaller than $\epsilon_1$.

---

## Practical Implications

### Choosing Sample Size

To achieve total error at most $\epsilon_{total}$ with confidence $1 - \delta$:

1. Set sampling error target: $\epsilon_2 = \epsilon_{total} - \epsilon_1$

2. Sample size needed:
$$m \geq \frac{(v_{max} - v_{min})^2 \ln(2/\delta)}{2\epsilon_2^2}$$

### When Does Noise Dominate?

- If $\epsilon_1$ is large relative to $\epsilon_{total}$, **more sampling doesn't help much**
- If $\epsilon_1$ is small, you can achieve high accuracy by sampling more

### Example

Suppose:
- $v_i \in [0, 1]$ for all $i$, so $v(S) \in [0, n]$
- Value noise: $\epsilon_1 = 0.05$
- Desired total error: $\epsilon_{total} = 0.1$
- Confidence: $\delta = 0.05$

Then:
- Sampling error budget: $\epsilon_2 = 0.1 - 0.05 = 0.05$
- Sample size: $m \geq \frac{n^2 \ln(40)}{2(0.05)^2} = \frac{3.69n^2}{0.005} \approx 738n^2$

---

## Important Note on the Constraint

Your constraint $\left|\sum_{i=1}^n v_i - \sum_{i=1}^n v_i^*\right| \leq \epsilon_1$ is relatively **weak** for bounding individual player errors.

**Why?** In the worst case:
- Player 1: $v_1 = v_1^* + \epsilon_1$
- All others: $v_j = v_j^*$ for $j \neq 1$
- Sum constraint: $|(v_1 + \sum_{j\neq 1} v_j) - (v_1^* + \sum_{j\neq 1} v_j^*)| = \epsilon_1$ ✓
- But: $|v_1 - v_1^*| = \epsilon_1$

So the bound $|\beta_i - \beta_i^*| \leq \epsilon_1$ is tight in worst case.

**If you have stronger constraints** (e.g., $|v_i - v_i^*| \leq \epsilon_0$ for all $i$), then:
$$|\beta_i - \beta_i^*| = |v_i - v_i^*| \leq \epsilon_0$$

which could be much better!

---

## Summary

**Yes, you can still use Hoeffding for sampling**, and the total error is:

$$\boxed{|\hat{\beta}_i - \beta_i^*| \leq \epsilon_1 + (v_{max} - v_{min})\sqrt{\frac{\ln(2/\delta)}{2m}}}$$

The noise in player values adds a **bias term** that doesn't decrease with more samples, while the sampling error decreases as $O(1/\sqrt{m})$.