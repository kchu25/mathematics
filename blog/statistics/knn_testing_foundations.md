@def title = "Mathematical Foundations of k-NN Hypothesis Testing: From Order Statistics to Permutation Power"
@def published = "10 March 2026"
@def tags = ["statistics", "hypothesis-testing", "nearest-neighbors", "order-statistics"]

# Mathematical Foundations of k-NN Hypothesis Testing

*Everything you need to rigorously understand the k-NN clustering test, built from scratch.*

---

## Why This Post Exists

Across the [clustering test post](/blog/statistics/nnd_cluster_test/), the [k hyperparameter post](/blog/statistics/knn_hyperparameter_theory/), and the [small-k justification](/blog/statistics/small_k_hypothesis_testing/), we built and used the k-NN permutation test for detecting subpopulation clustering. Along the way, we invoked a lot of machinery — order statistics, exchangeability, permutation null distributions, power analysis — sometimes without full derivation.

This post is the mathematical companion. It fills in the foundations, starting from material at the level of Casella & Berger (chapters 1–9) and extending to the specific theory that makes the k-NN test work.

The goal: after reading this, nothing in the other posts should feel like a black box.

**Roadmap:**

1. **Order statistics and spacings** — The exact distribution of nearest-neighbor distances.
2. **Exchangeability and the permutation principle** — Why relabeling gives valid tests.
3. **Permutation test theory** — Finite-sample exactness and why you don't need asymptotics.
4. **k-NN distances as random variables** — Their distributions under uniformity and beyond.
5. **The test statistic: mean k-NN distance** — Its expectation, variance, and behavior.
6. **Power analysis for permutation tests** — When and why the test rejects.
7. **Connecting to two-sample theory** — Friedman-Rafsky and the graph-based view.

---

## Part 1: Order Statistics and Spacings

You already know order statistics from Casella (chapter 5). Let's make them do specific work.

### 1.1 Setup and Notation

Let $X_1, X_2, \ldots, X_n$ be i.i.d. continuous random variables with CDF $F$ and PDF $f$. The **order statistics** are:

$$X_{(1)} \leq X_{(2)} \leq \cdots \leq X_{(n)}$$

From Casella & Berger (Theorem 5.4.6), the joint density of all order statistics is:

$$f_{X_{(1)}, \ldots, X_{(n)}}(x_1, \ldots, x_n) = n! \prod_{i=1}^n f(x_i), \quad x_1 < x_2 < \cdots < x_n$$

And the marginal density of the $j$-th order statistic is:

$$f_{X_{(j)}}(x) = \frac{n!}{(j-1)!(n-j)!} [F(x)]^{j-1} [1 - F(x)]^{n-j} f(x)$$

This is a Beta-Binomial structure: $F(X_{(j)}) \sim \text{Beta}(j, n-j+1)$.

### 1.2 Why This Matters for Nearest Neighbors

Here's the connection: **the distance from a point to its $k$-th nearest neighbor is an order statistic of the distances to all other points.**

Fix a point $x_0$. Define the distances from $x_0$ to all other $n$ points:

$$R_i = |X_i - x_0|, \quad i = 1, \ldots, n$$

Order these: $R_{(1)} \leq R_{(2)} \leq \cdots \leq R_{(n)}$.

Then $R_{(k)}$ is the distance to the $k$-th nearest neighbor. **It's the $k$-th order statistic of the distances $\{R_i\}$.**

So to understand k-NN distances, we need the distribution of $R_{(k)}$.

### 1.3 Spacings: The Gaps Between Order Statistics

The **spacings** (or gaps) are:

$$D_j = X_{(j)} - X_{(j-1)}, \quad j = 1, \ldots, n+1$$

where $X_{(0)} = a$ and $X_{(n+1)} = b$ if the support is $[a,b]$.

**Why spacings matter for nearest neighbors:** In 1D, the nearest-neighbor distance of $X_{(j)}$ is:

$$\text{NN distance of } X_{(j)} = \min(D_j, D_{j+1})$$

The $k$-th nearest-neighbor distance involves the $k$ smallest of the surrounding spacings. So the distribution of spacings directly governs the distribution of NN distances.

### 1.4 Spacings Under Uniformity

The simplest and most instructive case: $X_1, \ldots, X_n \overset{iid}{\sim} \text{Uniform}(0, 1)$.

**Key result (Theorem):** The spacings $(D_1, D_2, \ldots, D_{n+1})$ follow a symmetric Dirichlet distribution:

$$(D_1, \ldots, D_{n+1}) \sim \text{Dirichlet}(1, 1, \ldots, 1)$$

This means each spacing has marginal distribution:

$$D_j \sim \text{Beta}(1, n) \quad \text{i.e., } f_{D_j}(d) = n(1-d)^{n-1}, \quad 0 \leq d \leq 1$$

**Expected spacing:** $E[D_j] = \frac{1}{n+1}$

**Variance of spacing:** $\text{Var}(D_j) = \frac{n}{(n+1)^2(n+2)}$

For large $n$, $\text{Var}(D_j) \approx \frac{1}{n^2}$, so standard deviation $\approx 1/n$.

**Intuition:** Throw $n$ points uniformly on $[0,1]$. The gaps are roughly $1/n$ each, with fluctuations of order $1/n$. This is tight — uniform spacings are surprisingly regular.

### 1.5 The 1D Nearest-Neighbor Distance Under Uniformity

For $n$ uniform points on $[0,1]$, the nearest-neighbor distance for the $j$-th order statistic is:

$$\text{NN}_j = \min(D_j, D_{j+1})$$

where $D_j, D_{j+1} \sim \text{Beta}(1, n)$.

The CDF of $\text{NN}_j$ is (since both spacings are approximately independent for interior points):

$$P(\text{NN}_j > r) \approx P(D_j > r) \cdot P(D_{j+1} > r) = (1-r)^{2n}$$

Wait — but this ignores the dependence between adjacent spacings. More carefully: adjacent spacings in a Dirichlet are *negatively* correlated (if one is large, its neighbors tend to be smaller, because they must sum to 1). The correlation is:

$$\text{Corr}(D_j, D_{j+1}) = -\frac{1}{n+1}$$

This is negligible for large $n$, so the approximation above is accurate:

$$P(\text{NN}_j \leq r) \approx 1 - (1-r)^{2n}$$

The expected 1-NN distance is:

$$E[\text{NN}_j] \approx \frac{1}{2n}$$

**This is the punchline for 1D:** If you have $n$ uniform points on $[0,1]$, the typical nearest-neighbor distance is about $\frac{1}{2n}$.

### 1.6 The k-th Nearest-Neighbor Distance Under Uniformity

Generalizing: the $k$-th NN distance is the $k$-th smallest among the $2$ neighboring spacings... no, that's only for $k=1$. For $k \geq 2$ in 1D, you need to look at the $k$ nearest points, which means considering *multiple* spacings.

A cleaner approach: Fix $x_0 \in [0,1]$. The number of points in $[x_0 - r, x_0 + r]$ follows $\text{Binomial}(n, 2r)$ (for uniform data, ignoring boundary effects). The $k$-th NN distance $R_{(k)}$ satisfies:

$$P(R_{(k)} \leq r) = P(\text{Bin}(n, 2r) \geq k) = \sum_{j=k}^{n} \binom{n}{j} (2r)^j (1 - 2r)^{n-j}$$

This is an **incomplete Beta function**. Specifically:

$$P(R_{(k)} \leq r) = I_{2r}(k, n - k + 1)$$

where $I_x(a, b) = \frac{B(x; a, b)}{B(a, b)}$ is the regularized incomplete beta function.

**Expected value:**

$$E[R_{(k)}] = \frac{1}{2} \cdot \frac{k}{n+1} = \frac{k}{2(n+1)}$$

(Using the fact that if $U \sim \text{Beta}(k, n-k+1)$, then $E[U] = k/(n+1)$, and $R_{(k)} \approx U/2$.)

**This is the key formula:** For $n$ uniform points on $[0,1]$, the expected $k$-th NN distance is approximately $\frac{k}{2n}$.

| $k$ | Expected $k$-NN distance | Interpretation |
|-----|---|---|
| 1 | $\frac{1}{2n}$ | Half a mean spacing |
| 3 | $\frac{3}{2n}$ | 1.5 mean spacings |
| 5 | $\frac{5}{2n}$ | 2.5 mean spacings |
| $\sqrt{n}$ | $\frac{\sqrt{n}}{2n} = \frac{1}{2\sqrt{n}}$ | Meso-scale |

**Intuition check:** If you have 100 uniform points on $[0,1]$, the mean spacing is $1/100 = 0.01$. The expected 5-NN distance is $5/200 = 0.025$. The expected $\sqrt{100}=10$-th NN distance is $10/200 = 0.05$. These numbers feel right.

---

## Part 2: Exchangeability and the Permutation Principle

This section gives the theoretical engine behind the permutation test. It's the reason you can get **exact p-values** without knowing the underlying distribution.

### 2.1 Exchangeability

**Definition:** Random variables $Z_1, \ldots, Z_N$ are **exchangeable** if for every permutation $\pi$ of $\{1, \ldots, N\}$:

$$(Z_1, \ldots, Z_N) \overset{d}{=} (Z_{\pi(1)}, \ldots, Z_{\pi(N)})$$

The joint distribution is invariant under relabeling.

**Key fact (Casella ch. 4 flavor):** If $Z_1, \ldots, Z_N$ are i.i.d., they are exchangeable. But exchangeability is weaker — the $Z_i$ don't need to be independent or identically distributed. They just need to be "interchangeable" in distribution.

**Example:** Draw $n$ balls without replacement from an urn. The sequence of colors is exchangeable (any ordering is equally likely) even though the draws are dependent.

### 2.2 The Permutation Null Hypothesis

In our clustering test, the null hypothesis is:

$$H_0: \text{The subpopulation labels are assigned at random.}$$

Formally: Let $\mathcal{P} = \{z_1, \ldots, z_N\}$ be the **pooled data** (subpopulation + background), where $N = m + n$. Under $H_0$, the assignment of which $m$ points are "subpopulation" and which $n$ points are "background" is **uniformly random** over all $\binom{N}{m}$ possible assignments.

**Equivalently:** Under $H_0$, the labels (subpop vs. background) are exchangeable given the data.

This is **not** the same as saying "the two groups have the same distribution" — it's stronger. It says the *labeling itself* carries no information.

### 2.3 Why Exchangeability Gives Exact Tests

Here's the core theorem. It's simple but profound:

**Theorem (Permutation Test Exactness).** Let $T = T(Z_1, \ldots, Z_N; L_1, \ldots, L_N)$ be any test statistic computed from the data $Z_i$ and labels $L_i \in \{\text{sub}, \text{bg}\}$. Under $H_0$ (exchangeable labeling):

$$P_{H_0}(T \leq t) = \frac{\#\{\pi : T_\pi \leq t\}}{\binom{N}{m}}$$

where $T_\pi$ is the test statistic computed after permuting the labels according to $\pi$.

**Proof sketch.** Under $H_0$, every assignment of the $m$ "subpopulation" labels to the $N$ data points is equally likely. There are $\binom{N}{m}$ such assignments. So:

$$P_{H_0}(T \leq t) = \frac{1}{\binom{N}{m}} \sum_{\pi} \mathbf{1}[T_\pi \leq t]$$

which is exactly the permutation p-value. $\square$

**Why this is remarkable:**
- No distributional assumptions (non-parametric).
- Works for any sample size (finite-sample exact, not asymptotic).
- Works for any test statistic $T$ (mean k-NN distance, max, median — anything).
- The only assumption is exchangeability under $H_0$.

**Comparison with Casella-style tests:** In Casella & Berger, you derive null distributions by assuming a parametric model (e.g., normal), then using properties of that distribution (e.g., $t$-statistic follows Student's $t$). The permutation test sidesteps this entirely: the null distribution is constructed from the data itself.

### 2.4 The Monte Carlo Approximation

In practice, $\binom{N}{m}$ is astronomically large (e.g., $\binom{1000}{100} \approx 10^{140}$), so we can't enumerate all permutations. Instead, we sample $B$ random permutations:

$$\hat{p} = \frac{\#\{b : T^{(b)} \leq T_{\text{obs}}\}}{B}$$

**How accurate is this approximation?**

The Monte Carlo p-value $\hat{p}$ is a sample mean of Bernoulli random variables with success probability $p_{\text{true}}$. By the CLT:

$$\hat{p} \approx N\left(p_{\text{true}}, \frac{p_{\text{true}}(1 - p_{\text{true}})}{B}\right)$$

The standard error of $\hat{p}$ is:

$$\text{SE}(\hat{p}) = \sqrt{\frac{p(1-p)}{B}}$$

For $B = 10{,}000$ and $p_{\text{true}} = 0.05$:

$$\text{SE} = \sqrt{\frac{0.05 \times 0.95}{10{,}000}} = 0.0022$$

So a 95% confidence interval for the true p-value is approximately $0.05 \pm 0.004$. This is precise enough for most applications.

| $B$ (permutations) | SE at $p=0.05$ | SE at $p=0.01$ | SE at $p=0.001$ |
|---|---|---|---|
| 1,000 | 0.0069 | 0.0031 | 0.0010 |
| 10,000 | 0.0022 | 0.0010 | 0.0003 |
| 100,000 | 0.0007 | 0.0003 | 0.0001 |

**Rule of thumb:** Use $B \geq 1/\alpha$. If you want to detect $p = 0.001$, use $B \geq 10{,}000$.

---

## Part 3: k-NN Distances as Random Variables

Now we combine Parts 1 and 2 to analyze the actual test statistic.

### 3.1 Setup

We have:
- **Pooled data:** $\mathcal{P} = \{z_1, \ldots, z_N\}$ in $\mathbb{R}^d$, with $N = m + n$.
- **Labels:** $L_i \in \{S, B\}$ (subpopulation or background), with $|\{i : L_i = S\}| = m$.
- **Test statistic:**

$$T_k = \frac{1}{m} \sum_{i : L_i = S} d_k(z_i, \mathcal{P})$$

where $d_k(z_i, \mathcal{P})$ is the distance from $z_i$ to its $k$-th nearest neighbor in $\mathcal{P}$.

Note: we compute k-NN distances in the **full pooled dataset**, not just within the subpopulation. This is important — the KD-tree is built once on $\mathcal{P}$, and both the observed statistic and permutation statistics use the same tree.

### 3.2 Distribution of a Single k-NN Distance

Fix a point $z_i \in \mathcal{P}$. Its $k$-th nearest-neighbor distance $d_k(z_i)$ is determined entirely by the **configuration of the other $N-1$ points** around $z_i$.

#### 3.2.1 Under Uniformity in 1D

If all $N$ points are i.i.d. Uniform on $[0, L]$, then from Part 1:

$$E[d_k(z_i)] \approx \frac{kL}{2N}$$

$$\text{Var}[d_k(z_i)] \approx \frac{k(N-k)L^2}{4N^2(N+1)}$$

For large $N$:

$$\text{Var}[d_k] \approx \frac{kL^2}{4N^2}$$

So the coefficient of variation (CV) is:

$$\text{CV}[d_k] = \frac{\text{sd}[d_k]}{E[d_k]} = \frac{\sqrt{k}L/(2N)}{kL/(2N)} = \frac{1}{\sqrt{k}}$$

**Insight:** Larger $k$ gives lower relative variability. At $k=1$, CV $= 1$ (very noisy). At $k=25$, CV $= 0.2$ (relatively stable). This is the variance reduction from averaging over more neighbors.

#### 3.2.2 Under a General Density in $\mathbb{R}^d$

For a general density $f$ in $\mathbb{R}^d$, the $k$-th NN distance from $z_i$ satisfies (asymptotically as $N \to \infty$):

$$d_k(z_i) \approx \left(\frac{k}{N \cdot c_d \cdot f(z_i)}\right)^{1/d}$$

where $c_d = \pi^{d/2} / \Gamma(d/2 + 1)$ is the volume of the unit ball in $\mathbb{R}^d$.

**Derivation sketch:** The expected number of points in a ball of radius $r$ around $z_i$ is approximately $N \cdot c_d \cdot r^d \cdot f(z_i)$. Set this equal to $k$ and solve for $r$:

$$N \cdot c_d \cdot r^d \cdot f(z_i) = k \implies r = \left(\frac{k}{N c_d f(z_i)}\right)^{1/d}$$

The actual $d_k(z_i)$ fluctuates around this value. More precisely, $d_k(z_i)$ has an asymptotic distribution related to the Gamma distribution:

$$N \cdot c_d \cdot f(z_i) \cdot d_k(z_i)^d \sim \text{Gamma}(k, 1)$$

This is because the number of points in a ball of radius $r$ is approximately Poisson with rate $N c_d f(z_i) r^d$ (for large $N$, the binomial approaches Poisson), and the $k$-th arrival time of a Poisson process is Gamma-distributed.

**This is worth absorbing:** The k-NN distance, raised to the $d$-th power and scaled by local density, is asymptotically Gamma. This is a fundamental result that undergirds all of k-NN theory.

### 3.3 Distribution of the Mean k-NN Distance

The test statistic is:

$$T_k = \frac{1}{m} \sum_{i=1}^{m} d_k(z_i)$$

Under the null (random labeling), the $m$ points labeled "subpopulation" are a random subset of $\mathcal{P}$. So:

$$E_{H_0}[T_k] = \frac{1}{m} \sum_{i=1}^{m} E[d_k(z_i)] = \frac{1}{N} \sum_{j=1}^{N} d_k(z_j)$$

Wait — let me be more careful. Under the null, the subpopulation is a random size-$m$ subset of $\mathcal{P}$. So $T_k$ is the mean of $d_k$ values for $m$ randomly chosen points from $\mathcal{P}$. This is sampling without replacement from a finite population.

Let $D_j = d_k(z_j, \mathcal{P})$ for $j = 1, \ldots, N$ be the precomputed k-NN distances for *all* points.

**Under $H_0$:**

$$E_{H_0}[T_k] = \bar{D} = \frac{1}{N} \sum_{j=1}^{N} D_j$$

(The mean of a random sample equals the population mean — this is true for sampling without replacement.)

$$\text{Var}_{H_0}[T_k] = \frac{S_D^2}{m} \cdot \frac{N - m}{N - 1}$$

where $S_D^2 = \frac{1}{N-1} \sum_{j=1}^N (D_j - \bar{D})^2$ is the population variance of the k-NN distances.

The factor $\frac{N-m}{N-1}$ is the **finite population correction (FPC)** — it accounts for sampling without replacement. When $m \ll N$, FPC $\approx 1$ and this reduces to the usual $S_D^2/m$.

**Why this matters:** The null distribution of $T_k$ is completely determined by:
1. The $N$ precomputed k-NN distances $\{D_1, \ldots, D_N\}$ (fixed, not random — they depend only on the spatial configuration).
2. The hypergeometric-like sampling of which $m$ distances are averaged.

This is elegant: the permutation test doesn't need to know $f$ or $d$ or anything about the underlying density. It just shuffles labels and recomputes the mean.

### 3.4 Asymptotic Normality Under $H_0$

For large $m$, the CLT for finite populations gives:

$$\frac{T_k - \bar{D}}{\sqrt{S_D^2 / m \cdot (N-m)/(N-1)}} \xrightarrow{d} N(0, 1) \quad \text{under } H_0$$

This means the null distribution of $T_k$ is approximately normal — centered at $\bar{D}$ (the overall mean k-NN distance) with variance determined by the spread of k-NN distances across all $N$ points.

**Practical check:** You can verify this by plotting the histogram of $\{T_k^{(1)}, \ldots, T_k^{(B)}\}$ from your permutation replicates. It should look approximately bell-shaped.

**When the CLT fails:** If $m$ is very small (say $m < 15$), the CLT may be inaccurate. In that case, the permutation p-value is still exact (it doesn't rely on CLT), but the normal approximation for analytical power calculations breaks down.

---

## Part 4: The Mechanics of Rejection

Now we can precisely describe when and why the test rejects.

### 4.1 The Observed Test Statistic Under Clustering

Under $H_a$ (the subpopulation is genuinely clustered), the $m$ subpopulation points are concentrated in a small region. Their k-NN distances $\{D_i\}_{i \in S}$ are systematically smaller than average.

So the observed $T_k^{\text{obs}}$ falls in the **left tail** of the null distribution.

**How far into the left tail?** This depends on the **effect size**, which we can now express precisely:

$$\delta = \frac{\bar{D} - T_k^{\text{obs}}}{\sqrt{S_D^2 / m}}$$

This is the number of standard deviations the observed statistic lies below the null mean.

**When is $\delta$ large?**

- When the subpopulation's k-NN distances are much smaller than the average k-NN distance across all points.
- When the subpopulation is large enough ($m$) to reduce the standard error.
- When the k-NN distances across all points have low variance (the denominator is small).

### 4.2 A Concrete Calculation

**Setup:** $N = 1000$ points on $[0, 100]$, with $n = 900$ background (uniform) and $m = 100$ subpopulation (uniform on $[40, 60]$).

**Step 1: Compute all k-NN distances** (using $k=5$).

For background points (scattered on $[0,100]$):
- Expected 5-NN distance $\approx \frac{5 \times 100}{2 \times 1000} = 0.25$

For subpopulation points (concentrated on $[40,60]$):
- They live in a dense region. In $[40,60]$, there are $\approx 100$ subpop points + $\approx 180$ background points = 280 points in an interval of width 20.
- Local 5-NN distance $\approx \frac{5 \times 20}{2 \times 280} \approx 0.18$

**Step 2: Null distribution parameters.**

$$\bar{D} \approx \frac{900 \times 0.25 + 100 \times 0.18}{1000} = \frac{225 + 18}{1000} = 0.243$$

**Step 3: Observed statistic.**

$$T_5^{\text{obs}} \approx 0.18$$

**Step 4: Effect size.**

Rough estimate of $S_D^2$: The k-NN distances have mean $\approx 0.243$ and range from about 0.1 to 0.5 (background points near the edges have larger NN distances). Say $S_D \approx 0.08$.

$$\delta \approx \frac{0.243 - 0.18}{0.08 / \sqrt{100}} = \frac{0.063}{0.008} = 7.9$$

That's nearly 8 standard deviations below the null mean. The p-value is astronomically small.

**What happens if the cluster is less tight?** Say subpopulation on $[30, 70]$ (width 40 instead of 20):

- Subpop local density increases less dramatically.
- $T_5^{\text{obs}}$ is larger (maybe 0.22).
- $\delta \approx \frac{0.243 - 0.22}{0.008} = 2.9$.
- Still significant ($p \approx 0.002$), but much less extreme.

**What if the cluster is barely distinguishable?** Subpopulation on $[20, 80]$ (width 80):

- $T_5^{\text{obs}} \approx 0.24$ (barely different from $\bar{D} \approx 0.243$).
- $\delta \approx \frac{0.003}{0.008} = 0.375$.
- Not significant ($p \approx 0.35$).

**Lesson:** The test has power proportional to how much the subpopulation's k-NN distances deviate from the population average. Tighter clusters → larger deviation → smaller p-value.

---

## Part 5: Power Theory for Permutation Tests

### 5.1 What Determines Power?

The power of the k-NN permutation test is:

$$\text{Power} = P_{H_a}(T_k \leq t_\alpha)$$

where $t_\alpha$ is the $\alpha$-quantile of the permutation null distribution.

Using the normal approximation (valid for moderate $m$):

$$\text{Power} \approx \Phi\left(\frac{\bar{D} - T_k^{H_a} - z_{1-\alpha} \cdot \sigma_{H_0}}{\sigma_{H_a}}\right)$$

Wait, let me be more careful. Let me derive this properly.

Under $H_0$: $T_k \sim N(\mu_0, \sigma_0^2)$ where $\mu_0 = \bar{D}$, $\sigma_0^2 = S_D^2/m \cdot (N-m)/(N-1)$.

We reject when $T_k \leq t_\alpha$, where $t_\alpha = \mu_0 - z_{1-\alpha} \sigma_0$.

Under $H_a$: $T_k$ has some distribution with mean $\mu_a < \mu_0$ (the subpopulation's k-NN distances are smaller). Approximately $T_k \sim N(\mu_a, \sigma_a^2)$.

$$\text{Power} = P_{H_a}(T_k \leq t_\alpha) = \Phi\left(\frac{t_\alpha - \mu_a}{\sigma_a}\right) = \Phi\left(\frac{\mu_0 - z_{1-\alpha}\sigma_0 - \mu_a}{\sigma_a}\right)$$

If we assume $\sigma_a \approx \sigma_0$ (the variance of the mean k-NN distance is similar under $H_0$ and $H_a$ — reasonable when $m$ is small relative to $N$):

$$\text{Power} \approx \Phi\left(\frac{\mu_0 - \mu_a}{\sigma_0} - z_{1-\alpha}\right) = \Phi(\delta - z_{1-\alpha})$$

where $\delta = (\mu_0 - \mu_a)/\sigma_0$ is the effect size.

**Standard power formula:** For a one-sided test at level $\alpha$:

$$\text{Power} = \Phi(\delta - z_{1-\alpha})$$

| Effect size $\delta$ | Power (at $\alpha = 0.05$, $z_{0.95} = 1.645$) |
|---|---|
| 1.0 | $\Phi(1.0 - 1.645) = \Phi(-0.645) = 0.26$ |
| 2.0 | $\Phi(2.0 - 1.645) = \Phi(0.355) = 0.64$ |
| 3.0 | $\Phi(3.0 - 1.645) = \Phi(1.355) = 0.91$ |
| 4.0 | $\Phi(4.0 - 1.645) = \Phi(2.355) = 0.99$ |
| 5.0 | $\Phi(5.0 - 1.645) = \Phi(3.355) \approx 1.00$ |

**This is Casella-style power analysis**, applied to the permutation test. The only difference from classical power analysis is that $\mu_0$ and $\sigma_0$ come from the permutation null distribution rather than a parametric model.

### 5.2 How $k$ Affects Power: A Rigorous Treatment

Now we can answer the question precisely: **which $k$ gives highest power?**

The effect size is:

$$\delta(k) = \frac{\mu_0(k) - \mu_a(k)}{\sigma_0(k)}$$

We need to understand how each quantity depends on $k$.

**The numerator: $\mu_0(k) - \mu_a(k)$.**

$\mu_0(k)$ is the mean k-NN distance under random labeling (average over all $N$ points).
$\mu_a(k)$ is the mean k-NN distance for the actual subpopulation.

For a tight cluster of $m$ points in a region of radius $R_{\text{cluster}}$ within a background of radius $R_{\text{bg}}$:

Under $H_a$ (true subpopulation), each subpopulation point has $k$ neighbors within the cluster. Its $k$-NN distance is governed by the local density within the cluster:

$$\mu_a(k) \approx \left(\frac{k}{m \cdot c_d}\right)^{1/d} \cdot R_{\text{cluster}}$$

Under $H_0$ (random labeling), the $m$ randomly chosen points are scattered across $R_{\text{bg}}$:

$$\mu_0(k) \approx \left(\frac{k}{N \cdot c_d}\right)^{1/d} \cdot R_{\text{bg}}$$

So the numerator is:

$$\mu_0(k) - \mu_a(k) \propto k^{1/d} \cdot \left[\frac{R_{\text{bg}}}{N^{1/d}} - \frac{R_{\text{cluster}}}{m^{1/d}}\right]$$

**The denominator: $\sigma_0(k)$.**

The variance of the mean k-NN distance under $H_0$ comes from sampling $m$ of the $N$ precomputed k-NN distances. From Part 3.3:

$$\sigma_0(k) = \sqrt{\frac{S_D^2(k)}{m} \cdot \frac{N-m}{N-1}}$$

The population variance $S_D^2(k)$ depends on how much the individual k-NN distances vary across all $N$ points. For a mix of clustered and background points, this variance is driven by the contrast between high-density and low-density regions.

**Putting it together:** The effect size scales as:

$$\delta(k) \propto \frac{k^{1/d} \cdot (\text{density contrast})}{S_D(k) / \sqrt{m}}$$

Now, how does $S_D(k)$ scale with $k$?

For uniform data, $S_D(k) \propto k^{1/d}$ (since the mean scales as $k^{1/d}$, so does the standard deviation — the shape of the distribution is roughly preserved).

Therefore:

$$\delta(k) \propto \frac{k^{1/d} \cdot C}{k^{1/d} / \sqrt{m}} = C\sqrt{m}$$

**A surprising prediction:** The effect size is *independent of $k$* (to first order), and depends mainly on the density contrast $C$ and the subpopulation size $\sqrt{m}$.

### 5.3 So Why Does $k$ Matter at All?

The first-order analysis above predicts $\delta$ is independent of $k$. But this uses crude asymptotic scaling. In practice, second-order effects break the symmetry:

**Effect 1: Boundary contamination.** For large $k$, some of the $k$ neighbors of subpopulation points lie *outside* the cluster. This reduces $\mu_0(k) - \mu_a(k)$ — the signal degrades.

This kicks in when $k > m_{\text{local}}$, where $m_{\text{local}}$ is the number of points within the cluster radius. For a tight cluster with all $m$ points inside, this starts mattering at $k \approx m/2$ (when you're reaching beyond the cluster for neighbors).

**Effect 2: Non-uniform background.** If the background density varies (not perfectly uniform), then large $k$ averages together NN distances from different density regimes, inflating $S_D(k)$ relative to $\mu_0(k) - \mu_a(k)$.

**Effect 3: Discrete sampling noise.** For very small $k$ (especially $k=1$), the NN distance is driven by the nearest single point, which is stochastic. The variance $S_D^2(1)$ is large relative to $\mu_0(1) - \mu_a(1)$ because of this noise.

**The net result:**

- **Very small $k$ ($k=1$):** Numerator is large (strong signal), but denominator is also large (high noise). Net effect: moderate power, sensitive to noise.
- **Small $k$ ($k=3$–$5$):** Numerator is still large (signal hasn't degraded much). Denominator is smaller (averaging over 3–5 neighbors reduces noise). **This is the sweet spot.**
- **Medium $k$ ($k \approx \sqrt{m}$):** Numerator starts degrading (boundary contamination). Denominator continues to decrease, but more slowly. Power may increase or decrease, depending on cluster geometry.
- **Large $k$ ($k \gg \sqrt{m}$):** Numerator is small (neighbors are mostly outside the cluster). Test approaches a variance test. Low power for detecting tight clustering.

### 5.4 Optimal $k$ for a Given Cluster Geometry

We can now state a principled result:

**Proposition.** For a cluster of $m$ points with local density $f_S$ embedded in a background with density $f_B < f_S$, the power-maximizing $k$ satisfies:

$$k^* \approx \arg\max_k \left[\frac{\mu_0(k) - \mu_a(k)}{\sigma_0(k)}\right]$$

For a well-separated compact cluster (all $m$ subpopulation points within a ball, minimal overlap with background):

$$k^* \approx \min\left(m/2, \; \text{number of points within the cluster boundary}\right)$$

In practice, this means:

- If your cluster has $m=100$ points, all tightly packed, the test has high power for any $k \leq 50$. The sweet spot is $k = 3$–$10$ (high signal, low noise).
- If your cluster is diffuse (only $m/3$ points are actually tightly packed, the rest are spread out), the optimal $k$ is smaller — you need to detect the tight core, not the entire group.

---

## Part 6: Why Permutation Tests Don't Need Distribution Assumptions

This section explains the deepest reason permutation tests are powerful: **they are conditional tests.**

### 6.1 Conditional Inference

Classical Casella-style inference (e.g., a $t$-test) reasons as:

> "If we repeated the experiment many times, drawing new samples each time, how often would we reject $H_0$ when it's true?"

The randomness comes from **sampling new data**.

Permutation inference reasons differently:

> "Given the data we observed (fixed!), if we randomly reassigned labels, how extreme would our test statistic be?"

The randomness comes from **relabeling**, not resampling. The data points themselves are treated as fixed.

**Why this is liberating:**
- You don't need to assume any parametric form for the data distribution.
- You don't need i.i.d. assumptions (only exchangeability under $H_0$).
- You don't need large-sample theory (finite-sample exact).
- You don't need to know the dimension $d$ or the density $f$.

**What you do need:**
- Exchangeability under $H_0$: the null hypothesis must imply that all labelings are equally likely.
- A sensible test statistic: one that is small (or large) under the alternative.

### 6.2 Conditioning on Sufficient Statistics

There's a Casella-friendly way to see why permutation tests work. Consider the pooled data $\{z_1, \ldots, z_N\}$ as fixed. Under $H_0$, the labels $\{L_1, \ldots, L_N\}$ are a random partition into two groups of sizes $m$ and $n$.

The **sufficient statistic** for the labeling is the set of labeled data $\{(z_i, L_i)\}$. But under $H_0$, the labels are uninformative about the data — they're just a random assignment. So the test is **conditional on the data configuration**.

This is analogous to Fisher's exact test for contingency tables: condition on the margins, then compute the exact probability of the observed (or more extreme) table.

**The permutation test is Fisher's exact test generalized to continuous data.**

### 6.3 Exchangeability vs. i.i.d.

A subtle but important distinction:

**i.i.d.** implies exchangeability, but exchangeability does *not* imply i.i.d.

Under $H_0$ in our setting:
- The data points are **not** i.i.d. (they come from a mix of subpopulation and background).
- But the **labels** are exchangeable (any assignment of $m$ labels to $N$ points is equally likely).

This weaker condition is all we need. The permutation test is valid even if the data has complex dependence structure, as long as the labels are exchangeable under the null.

**When might exchangeability fail?**
- If the subpopulation was selected *because* it appeared clustered (post-hoc selection bias).
- If the labeling depends on spatial location (e.g., you labeled points as "subpopulation" based on their proximity to other labeled points).
- If there's a temporal or spatial correlation that makes some labelings more likely than others under the null.

In practice, if the subpopulation was defined by an *external criterion* (e.g., a genetic marker, a demographic variable) rather than by the spatial configuration itself, exchangeability holds.

---

## Part 7: The Graph-Based View (Friedman-Rafsky)

There's an elegant way to see the k-NN test through the lens of graph theory. This connects to the Friedman-Rafsky framework and gives additional geometric intuition.

### 7.1 The k-NN Graph

Given $N$ points in $\mathbb{R}^d$, the **k-NN graph** $G_k$ is:

- **Vertices:** The $N$ points.
- **Edges:** Draw a directed edge from $z_i$ to $z_j$ if $z_j$ is among the $k$ nearest neighbors of $z_i$.

(Often symmetrized: an undirected edge exists between $z_i$ and $z_j$ if either is among the other's $k$ nearest neighbors.)

### 7.2 The Test as a Graph Statistic

Our test statistic $T_k = \frac{1}{m} \sum_{i \in S} d_k(z_i)$ can be rewritten in terms of the k-NN graph:

$$T_k = \frac{1}{m} \sum_{i \in S} w_k(i)$$

where $w_k(i)$ is the **weight of the $k$-th edge from vertex $i$** (i.e., the distance to the $k$-th nearest neighbor).

### 7.3 The Friedman-Rafsky Approach

Friedman and Rafsky (1979) proposed a different but related graph-based test using the **minimum spanning tree (MST)**.

**MST test:**
1. Compute the MST of the $N$ data points (the tree that connects all points with minimum total edge length).
2. Count the number of edges that connect a subpopulation point to a background point (call this $R$).
3. Under $H_0$ (random labeling), $R$ has a known distribution.
4. If $R$ is unusually small, the two groups are separated (subpopulation points are connected to each other, not to background points).

**Connection to k-NN test:** Both tests measure whether the subpopulation points are "near each other" in the graph structure. The MST test uses the global graph; the k-NN test uses local neighborhoods.

**Key difference:** The MST test uses $k=1$ implicitly (the MST connects each point to its most efficient neighbor). The k-NN test generalizes this to larger neighborhoods.

### 7.4 Graph Runs and the Run Test

There's an even simpler version in 1D. Order all $N$ points by value and look at the sequence of labels:

$$B, B, S, S, S, B, B, B, S, B, B, S, S, \ldots$$

A **run** is a maximal consecutive subsequence of the same label. Under $H_0$ (random labeling), the expected number of runs is:

$$E[\text{runs}] = \frac{2mn}{N} + 1$$

If the subpopulation is clustered (all $S$ labels are consecutive), the number of runs is **small** (perhaps just 2 or 3 runs of $S$, instead of many scattered runs).

The **Wald-Wolfowitz runs test** formalizes this. It's the 1D special case of the Friedman-Rafsky test, and it's closely related to the k-NN test:

- If subpopulation points are clustered in 1D, they form a long run → few runs.
- If subpopulation points are clustered, their NN distances are small → small $T_k$.

Both statistics capture the same geometric phenomenon: spatial concentration.

### 7.5 The Henze-Penrose Divergence

For completeness: the most theoretically sophisticated graph-based two-sample test uses the **Henze-Penrose test statistic**, based on the k-NN graph:

$$T_{HP} = \frac{1}{m} \sum_{i \in S} \frac{\#\{\text{neighbors of } z_i \text{ in } S\}}{k}$$

This counts the **fraction of each subpopulation point's neighbors that are also in the subpopulation**. Under $H_0$, this fraction should be approximately $m/N$ (by random chance). Under $H_a$ (clustering), this fraction is much higher (subpopulation points have mostly subpopulation neighbors).

**The relationship to our test:**
- Our test: "How far are the neighbors?" (distance-based).
- Henze-Penrose: "What fraction of neighbors are same-group?" (count-based).

Both detect the same phenomenon, but the Henze-Penrose statistic has the advantage of being **independent of the distance metric** — it only cares about neighborhood membership, not actual distances.

---

## Part 8: Bringing It All Together

### 8.1 The Complete Theoretical Chain

Here's the full logical chain from foundations to the test:

1. **Order statistics** (Part 1): k-NN distances are order statistics of inter-point distances. Their distributions are known under uniformity (Beta, Gamma) and approximately known under general densities.

2. **Exchangeability** (Part 2): Under $H_0$, labels are exchangeable, so the permutation test has exact Type I error $= \alpha$. No distributional assumptions needed.

3. **Test statistic distribution** (Part 3): The mean k-NN distance under $H_0$ is approximately normal (by CLT for finite populations), with known mean and variance derived from the pooled k-NN distances.

4. **Power analysis** (Part 5): Power depends on the effect size $\delta = (\mu_0 - \mu_a)/\sigma_0$, which in turn depends on how much the cluster's k-NN distances deviate from the population average. Tighter clusters → larger $\delta$ → higher power.

5. **Choice of $k$** (Part 5): Optimal $k$ depends on cluster geometry. For tight clusters, small $k$ captures the signal before boundary contamination dilutes it. For diffuse clusters, larger $k$ provides noise reduction.

6. **Graph interpretation** (Part 7): The test can be understood as measuring whether the subpopulation points form a dense subgraph in the k-NN graph. This connects to the Friedman-Rafsky MST test and the Wald-Wolfowitz runs test.

### 8.2 What You Can Now Justify

With this foundation, you can rigorously justify every step of the k-NN clustering test:

| Step | Justification |
|------|-------------|
| Using mean k-NN distance as test statistic | It's the sample mean of order statistics of inter-point distances. It converges to a well-defined population quantity (Parts 1, 3). |
| Using permutation for the null distribution | Exchangeability under $H_0$ guarantees exact Type I error (Part 2). |
| Using $B = 10{,}000$ permutations | Monte Carlo SE is $\leq 0.002$ at $\alpha = 0.05$ (Part 2.4). |
| Choosing small $k$ for tight clusters | Effect size is maximized before boundary contamination sets in (Part 5.3). |
| Pre-specifying $k$ | Avoids inflating Type I error from multiple testing. |
| Sensitivity analysis across $k$ | Reveals the scale at which clustering exists, using the geometry of the k-NN graph (Part 7). |
| Interpreting the p-value | "Under random labeling, fraction of assignments producing a mean k-NN distance this small or smaller" (Part 2). |

### 8.3 What This Framework Does *Not* Cover

To be honest about limitations:

1. **Optimal test statistic.** We chose the *mean* k-NN distance, but other functions of k-NN distances (median, trimmed mean, maximum) might have higher power in some settings. The permutation test is valid for *any* statistic, but we haven't proven that the mean is uniformly best.

2. **Asymptotic optimality.** The permutation test is exact for Type I error, but we don't have a proof that it's asymptotically optimal for power among all possible tests. (It likely isn't — for specific parametric alternatives, a likelihood ratio test would dominate.)

3. **High-dimensional behavior.** In very high dimensions ($d \gg 10$), k-NN distances concentrate (all distances become similar), reducing the test's discriminative power. This is the curse of dimensionality, and no nonparametric test fully escapes it.

4. **Multiple testing.** When testing many subpopulations simultaneously, you need to correct for multiple comparisons (Bonferroni, BH, etc.). The permutation test itself is fine, but the overall false discovery rate must be controlled.

---

## Appendix A: Proofs and Derivations

### A.1 Proof: Spacing Distribution Under Uniformity

**Claim:** If $X_1, \ldots, X_n \sim \text{Uniform}(0,1)$ i.i.d., then the spacings $(D_1, \ldots, D_{n+1})$ follow $\text{Dirichlet}(1, \ldots, 1)$.

**Proof:** The joint density of the order statistics is $f_{X_{(1)}, \ldots, X_{(n)}}(x_1, \ldots, x_n) = n!$ on $0 < x_1 < \cdots < x_n < 1$.

Define $D_j = X_{(j)} - X_{(j-1)}$ (with $X_{(0)} = 0$, $X_{(n+1)} = 1$). This is a linear transformation from $(X_{(1)}, \ldots, X_{(n)})$ to $(D_1, \ldots, D_n)$ (with $D_{n+1} = 1 - X_{(n)} = 1 - \sum_{j=1}^n D_j$).

The Jacobian of this transformation is 1. The constraint $0 < x_1 < \cdots < x_n < 1$ transforms to $D_j > 0$ for all $j$ and $\sum D_j = 1$. So:

$$f_{D_1, \ldots, D_n}(d_1, \ldots, d_n) = n!, \quad d_j > 0, \quad \sum_{j=1}^{n+1} d_j = 1$$

This is exactly the $\text{Dirichlet}(1, 1, \ldots, 1)$ density (up to the normalizing constant, which is $\Gamma(n+1)/\Gamma(1)^{n+1} = n!$). $\square$

### A.2 Proof: Finite Population CLT for the Permutation Test

**Claim:** Under $H_0$, the standardized test statistic converges in distribution to $N(0,1)$ as $m, n \to \infty$ with $m/N \to \lambda \in (0,1)$.

**Proof sketch:** Let $D_1, \ldots, D_N$ be the k-NN distances (fixed). The test statistic is:

$$T = \frac{1}{m} \sum_{j \in S} D_j$$

where $S$ is a random subset of size $m$ drawn uniformly from $\{1, \ldots, N\}$.

This is the mean of a simple random sample (without replacement) from a finite population $\{D_1, \ldots, D_N\}$.

By the **Hájek CLT for finite populations** (Hájek, 1960):

If $\max_{1 \leq j \leq N} (D_j - \bar{D})^2 / (N \cdot S_D^2) \to 0$ (no single observation dominates), then:

$$\frac{T - \bar{D}}{\sqrt{S_D^2/m \cdot (N-m)/(N-1)}} \xrightarrow{d} N(0,1)$$

The condition holds when the k-NN distances are not too extreme (no single distance is a macroscopic fraction of the total variance), which is satisfied in all practical clustering settings. $\square$

### A.3 Derivation: Gamma Approximation for k-NN Distances

**Claim:** For points from density $f$ in $\mathbb{R}^d$, $N c_d f(z_i) [d_k(z_i)]^d \sim \text{Gamma}(k, 1)$ asymptotically.

**Derivation:** The number of points in a ball of radius $r$ around $z_i$ is:

$$N_r = \sum_{j \neq i} \mathbf{1}[\|z_j - z_i\| \leq r]$$

For large $N$, $N_r \approx \text{Poisson}(\Lambda(r))$ where:

$$\Lambda(r) = N \int_{\|z - z_i\| \leq r} f(z) \, dz \approx N c_d r^d f(z_i)$$

(The last approximation uses that $f$ is approximately constant on a small ball around $z_i$.)

The $k$-th NN distance satisfies $d_k = \inf\{r : N_r \geq k\}$. This is the $k$-th arrival time of a Poisson process with rate function $\Lambda(r)$.

By the time-change theorem for Poisson processes, $\Lambda(d_k) \sim \text{Gamma}(k, 1)$:

$$N c_d f(z_i) [d_k(z_i)]^d \sim \text{Gamma}(k, 1) \quad \square$$

**Consequences:**

$$E[d_k^d] = \frac{k}{N c_d f(z_i)}, \quad \text{Var}[d_k^d] = \frac{k}{[N c_d f(z_i)]^2}$$

---

## Appendix B: Connecting to Casella & Berger

For readers who want to map the concepts in this post back to specific sections of Casella & Berger:

| Concept in this post | Casella & Berger reference |
|---|---|
| Order statistics, joint density | **Theorem 5.4.6** (joint density of order statistics) |
| Beta distribution for order statistics | **Section 5.4**, especially the result that $F(X_{(j)}) \sim \text{Beta}(j, n-j+1)$ |
| Sufficiency and conditioning | **Chapter 6** (sufficient statistics, Rao-Blackwell) |
| Hypothesis testing framework | **Chapter 8** (Neyman-Pearson, UMP tests, p-values) |
| Power and sample size | **Section 8.3** (power functions, Neyman-Pearson lemma) |
| Likelihood ratio tests | **Section 8.2** — the permutation test is *not* a LRT, but understanding LRTs helps see what the permutation test sacrifices (optimality under parametric alternatives) in exchange for (distribution-free validity) |
| Confidence intervals via inverting tests | **Section 9.2** — you could invert the permutation test to get a confidence interval for the "degree of clustering," though this is rarely done in practice |
| Nonparametric methods | Casella doesn't cover permutation tests extensively, but the **rank-based tests in Section 5.4** use order statistics in a similar spirit |
| CLT and asymptotic normality | **Chapter 5** (convergence in distribution, CLT) |
| Monte Carlo methods | **Section 5.5** (simulation, though Casella focuses on parametric bootstrap) |

**What Casella does not cover (and this post extends):**
- Permutation tests as conditional tests (Fisher's framework)
- Finite population CLT (Hájek's theorem)
- k-NN distances and their Gamma approximation
- Graph-based two-sample tests (Friedman-Rafsky)
- Power analysis for nonparametric permutation tests

---

## Appendix C: Historical and Intellectual Context

### C.1 The Two Traditions

The k-NN clustering test sits at the intersection of two intellectual traditions that rarely talk to each other:

**Tradition 1: Parametric hypothesis testing (Neyman-Pearson, Fisher).**
This is what Casella & Berger teaches. You assume a model, derive the distribution of your test statistic under $H_0$, and compute a p-value analytically. The gold standard is the **Neyman-Pearson lemma**: the likelihood ratio test is the most powerful test at any given significance level.

**Tradition 2: Nonparametric and resampling-based inference (Fisher's exact test, permutation tests, bootstrap).**
Here, you don't assume a parametric model. Instead, you construct the null distribution from the data itself (by permuting, bootstrapping, or conditioning). You give up optimality (no Neyman-Pearson guarantee) but gain robustness (valid for any distribution).

The k-NN permutation test is firmly in Tradition 2. Understanding *why* someone trained in Tradition 1 might feel uneasy — and why that unease is unwarranted — is important.

### C.2 Why Permutation Tests Feel "Weird" to a Casella-Trained Statistician

If you've internalized chapters 8-9 of Casella, you're used to:
1. Writing down a parametric model.
2. Computing the likelihood ratio.
3. Using the Neyman-Pearson lemma to get the UMP test.
4. Computing power analytically.

The permutation test skips all of this. It feels like it's "cheating" — how can you get a valid test without specifying a model?

**The resolution:** The permutation test is **conditional** on the observed data. It doesn't need a model for the data because it treats the data as fixed and only randomizes the labels. This is actually *more* conservative than parametric testing: the permutation test controls Type I error under weaker assumptions.

The trade-off: you lose the Neyman-Pearson optimality guarantee. For a specific parametric alternative, the LRT would have higher power. But if your alternative is "the subpopulation is somehow more tightly clustered" — a vague, geometry-dependent alternative that doesn't map cleanly to any parametric family — the permutation test is the right framework.

### C.3 The Role of the Test Statistic

In parametric testing, the optimal test statistic is the likelihood ratio (by Neyman-Pearson). In permutation testing, **any** statistic gives a valid test, but the choice of statistic determines power.

The mean k-NN distance is a good choice because:
- It directly measures what you care about (inter-point proximity).
- It's computationally efficient (KD-tree queries).
- It has known distributional properties (Gamma approximation, CLT for the mean).
- It has a tuning parameter ($k$) that lets you target clustering at different scales.

But it's not the *only* choice. Others (Henze-Penrose count, pairwise distance quantiles, the MST edge count) could have higher power for specific alternatives. The theory doesn't tell you which is best — that depends on the geometry of your specific problem.

---

## Summary

This post derived the mathematical machinery behind the k-NN clustering test:

1. **k-NN distances are order statistics** of inter-point distances, with known distributions (Beta for spacings, Gamma for the general case).

2. **Permutation tests are exact** under exchangeability, requiring no distributional assumptions. The Monte Carlo approximation is accurate and quantifiably so.

3. **The null distribution of the mean k-NN distance** follows from finite-population sampling theory (Hájek CLT), with mean and variance computable from the pooled k-NN distances.

4. **Power depends on effect size** $\delta = (\mu_0 - \mu_a)/\sigma_0$, which is governed by how much the cluster's k-NN distances deviate from the population average.

5. **Optimal $k$ emerges from balancing signal strength against noise**, not from density estimation theory. For tight clusters, small $k$ preserves signal before boundary effects dilute it.

6. **The graph-based view** (k-NN graph, MST, Friedman-Rafsky) provides geometric intuition: clustering means the subpopulation forms a dense subgraph.

Everything in the [clustering test post](/blog/statistics/nnd_cluster_test/), the [hyperparameter post](/blog/statistics/knn_hyperparameter_theory/), and the [small-k post](/blog/statistics/small_k_hypothesis_testing/) now has a foundation you can trace back to order statistics, exchangeability, and finite-population CLT — concepts that extend naturally from Casella & Berger.
