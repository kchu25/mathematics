@def title = "The KS Test: When the U Test Fails to See Shape Differences"
@def published = "10 March 2026"
@def tags = ["statistics"]

# The KS Test: When the U Test Fails to See Shape Differences

## The Motivating Problem

You plot contour plots of two subsets of your data. They look **clearly different** — different spread, different shape, different tails. You run a Mann-Whitney U test and get... a non-significant result. What went wrong?

**Nothing went wrong.** The U test simply wasn't designed to detect that kind of difference.

---

## What the Mann-Whitney U Test Actually Tests

The U test answers a very specific question:

> Given a random draw $X$ from group A and a random draw $Y$ from group B, is $P(X > Y) \neq 0.5$?

This is a test of **stochastic dominance** — whether one distribution tends to produce larger values than the other. Under the hood, it works entirely through **ranks**: it pools both samples, ranks them, and checks whether one group's ranks are systematically higher.

**What it detects:** Location shifts. If group B is "shifted right" relative to group A, the U test will catch it.

**What it misses:** Anything that doesn't change the *average rank*:
- Different **spreads** (one group wider than the other)
- Different **shapes** (one skewed, the other symmetric)
- Different **modality** (one bimodal, the other unimodal)
- Different **tails** (same center and spread, but different kurtosis)

### The classic failure case

Suppose group A is $N(0, 1)$ and group B is $N(0, 4)$ — same mean, but 4× the variance. A random draw from A is equally likely to be above or below a random draw from B. So $P(X > Y) = 0.5$ exactly, and the U test has **zero power** — it will never detect this difference no matter how much data you collect.

Yet a contour plot or density plot would show the difference immediately.

---

## Enter the Kolmogorov-Smirnov Test

### The idea

The KS test takes a completely different approach. Instead of comparing ranks or asking "which group tends to be bigger?", it asks:

> What is the maximum gap between the two empirical cumulative distribution functions (CDFs)?

If two distributions differ in **any way** — location, spread, shape, tails — their CDFs must differ somewhere. The KS test finds the point of maximum divergence.

### The math

Given samples $X_1, \ldots, X_m$ from group A and $Y_1, \ldots, Y_n$ from group B, compute the empirical CDFs:

$$F_m(t) = \frac{1}{m}\sum_{i=1}^{m} \mathbf{1}(X_i \leq t), \qquad G_n(t) = \frac{1}{n}\sum_{j=1}^{n} \mathbf{1}(Y_j \leq t)$$

The two-sample KS statistic is:

$$D_{m,n} = \sup_t |F_m(t) - G_n(t)|$$

This is simply the **largest vertical distance** between the two empirical CDF curves at any point $t$.

Under $H_0$ (both samples drawn from the same distribution), the scaled statistic $\sqrt{\frac{mn}{m+n}} \, D_{m,n}$ converges to the **Kolmogorov distribution**, which gives us p-values.

### What it detects

The KS test is **omnibus** — it is sensitive to *any* difference between the two distributions:

| Type of difference | U test detects? | KS test detects? |
|----|----|----|
| Location shift (different means) | ✅ Yes (strong) | ✅ Yes |
| Scale difference (different variances) | ❌ No | ✅ Yes |
| Shape difference (skew vs symmetric) | ❌ No | ✅ Yes |
| Tail difference (light vs heavy) | ❌ No | ✅ Yes (weakly) |
| Modality (unimodal vs bimodal) | ❌ No | ✅ Yes |

---

## Intuition: Why the CDF Sees Everything

A CDF $F(t) = P(X \leq t)$ is a **complete** characterization of a 1D distribution. Two distributions are identical **if and only if** their CDFs are identical. So any difference must appear as a gap somewhere in the CDF.

- A **location shift** slides the CDF left/right → the gap appears in the middle.
- A **scale difference** changes the CDF's slope → the gap appears in the tails.
- A **shape difference** warps the CDF → the gap appears where the warping is strongest.

The KS statistic simply asks: what is the worst-case gap?

### Visual: same median, different spread

```
CDF
1.0 |                    ___________  ← Group A (narrow)
    |                 __/
    |              __/  ________  ← Group B (wide)
    |           __/  __/
0.5 |        __/ ___/
    |     __/ __/     ← Maximum gap D here
    |  __/ __/
    | / __/
0.0 |/___/___________________________
    value →
```

The U test sees both CDFs crossing 0.5 at roughly the same point (same median) and declares "no difference." The KS test sees the large vertical gap at the tails and declares "different."

---

## The Trade-Off: Omnibus Power vs Focused Power

The KS test's generality is both its strength and its weakness:

**Strength:** It catches differences the U test is blind to.

**Weakness:** For pure location shifts, the U test is **more powerful** than the KS test. The U test focuses all of its statistical power on detecting location differences, while the KS test spreads its power across all possible differences.

| Scenario | Better test |
|----------|------------|
| Pure location shift, symmetric data | Mann-Whitney U |
| Pure location shift, skewed data | Mann-Whitney U (still) |
| Different spread, same center | KS test |
| Different shape (skew, kurtosis) | KS test |
| You don't know *what* kind of difference | KS test |
| You want maximum power for location | U test |

**Analogy:** The U test is a sniper rifle (deadly accurate at one target). The KS test is a shotgun (hits broadly but less forcefully at any specific target).

---

## Back to Your Contour Plots

You see different contour shapes. This means the distributions differ in **spread, shape, or correlation structure** — not just location. The U test was never going to find this because:

1. The U test is **univariate** — if your contour plots are 2D, you've already lost information by projecting to 1D for the U test.
2. Even in 1D, the U test only detects location shifts.

### Will the 1D KS test solve this?

**Partially.** If you project your 2D data onto a single axis and the shape/spread difference is visible on that axis, the KS test will likely detect it. But you're still losing the 2D structure.

### Better: the multivariate approach

For 2D (or higher) data where you see different contour shapes, consider:

**1. Multivariate KS-type tests**

There is no canonical multivariate KS test (the CDF supremum doesn't generalize cleanly to $\mathbb{R}^d$), but several extensions exist:
- **Fasano-Franceschini test** — a 2D generalization of KS
- **Peacock test** — another 2D KS variant

**2. Energy distance / E-test**

The energy distance between two distributions $P$ and $Q$ is:

$$\mathcal{E}(P, Q) = 2\mathbb{E}\|X - Y\| - \mathbb{E}\|X - X'\| - \mathbb{E}\|Y - Y'\|$$

where $X, X' \sim P$ and $Y, Y' \sim Q$. This is a true multivariate test that detects **any** distributional difference and naturally uses the geometry (Euclidean distance) of the data. The associated permutation test (E-test) is a powerful general-purpose two-sample test.

**3. Maximum Mean Discrepancy (MMD)**

A kernel-based test that maps distributions into a reproducing kernel Hilbert space:

$$\text{MMD}^2(P, Q) = \mathbb{E}[k(X,X')] + \mathbb{E}[k(Y,Y')] - 2\mathbb{E}[k(X,Y)]$$

With a Gaussian kernel, MMD detects any distributional difference and is sensitive to the geometric structure of the data. Permutation-based p-values.

**4. Multivariate Wasserstein distance**

The EMD/Wasserstein distance (see the [Earth Mover's Distance post](/blog/statistics/earth_mover_distance/)) naturally handles multivariate data with a ground metric that respects geometry. You'd compute $W_1$ or $W_2$ between your two groups and use permutation testing for inference.

---

## KS Test: Practical Details

### Assumptions

- Observations within each group are i.i.d.
- The two groups are independent.
- The distribution is **continuous** (ties cause conservatism — the test becomes less powerful).
- **No** assumption about the form of the distribution (fully non-parametric).

### Interpreting the output

| Output | Meaning |
|--------|---------|
| $D$ statistic | Maximum CDF gap. Ranges from 0 (identical) to 1 (completely separated). |
| $D = 0.05$ | The CDFs differ by at most 5 percentage points — very similar. |
| $D = 0.30$ | Substantial difference — at some value $t$, 30% more of one group lies below $t$ than the other. |
| p-value | Probability of observing a gap ≥ $D$ if both groups come from the same distribution. |

### Effect size

The $D$ statistic itself is an interpretable effect size:
- $D < 0.1$: trivial difference
- $0.1 \leq D < 0.2$: small
- $0.2 \leq D < 0.4$: moderate
- $D \geq 0.4$: large

These are rough guidelines; context matters.

### One-sided variants

The two-sided test uses $D = \sup |F - G|$. You can also test directional hypotheses:

$$D^+ = \sup_t [F_m(t) - G_n(t)] \quad (\text{A stochastically smaller than B})$$
$$D^- = \sup_t [G_n(t) - F_m(t)] \quad (\text{A stochastically larger than B})$$

---

## Summary: Choosing Between U Test and KS Test

```
What kind of difference are you looking for?
│
├─ "Is one group generally larger/smaller?"
│   └─ Mann-Whitney U test (focused, powerful for location)
│
├─ "Do these groups differ in ANY way?" (spread, shape, tails, ...)
│   └─ KS test (omnibus, detects everything)
│
├─ "My contour plots look different in 2D"
│   ├─ If effect is visible on a 1D projection → KS test on that axis
│   └─ If you need the full 2D structure →
│       ├─ Energy test (simple, general)
│       ├─ MMD test (flexible, kernel-based)
│       └─ Wasserstein distance + permutation test (geometric)
│
└─ "I want to detect location AND shape differences"
    └─ Run both U test and KS test, or use Energy/MMD
```

---

## Key Takeaways

1. **The U test is a location test.** It compares $P(X > Y)$ to 0.5. If two distributions have the same median/center but different shapes, the U test is blind.
2. **The KS test is an omnibus test.** It detects any difference between distributions by finding the maximum CDF gap. It will catch the spread/shape differences your contour plots reveal.
3. **The KS test pays for generality with power.** For pure location shifts, the U test wins. For everything else, the KS test wins.
4. **Neither is ideal for multivariate data.** If your contour plots are 2D, consider the energy test, MMD, or Wasserstein distance — these naturally handle multivariate geometry.
5. **The $D$ statistic is interpretable.** It tells you the maximum fraction of data that is "misallocated" between the two CDFs.
6. **When in doubt, run both.** A significant U + non-significant KS → pure location shift. Non-significant U + significant KS → shape/spread difference. Both significant → probably both location and shape.
