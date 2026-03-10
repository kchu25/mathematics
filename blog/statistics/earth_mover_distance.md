@def title = "Earth Mover's Distance (Wasserstein): Intuition, Math, and Julia Usage"
@def published = "10 March 2026"
@def tags = ["statistics", "optimization"]

# Earth Mover's Distance (Wasserstein): Intuition, Math, and Julia Usage

## The Core Idea

Suppose you have two histograms — two distributions of "stuff" — and you want to know **how different** they are. Many distances (KL divergence, total variation) just compare bin-by-bin. The Earth Mover's Distance (EMD), also called the **Wasserstein distance**, asks a fundamentally different question:

> **What is the minimum total work needed to reshape one distribution into the other?**

"Work" = amount of mass moved × distance it travels.

This is why it's called the Earth Mover's Distance: imagine one histogram is a pile of dirt and the other is a hole. How much effort does it take to shovel the dirt into the hole?

---

## Why Not Just Use KL Divergence or Total Variation?

### The problem with bin-by-bin metrics

Consider two distributions over positions $\{1, 2, 3, 4, 5\}$:

- $P = (1, 0, 0, 0, 0)$ — all mass at position 1
- $Q_1 = (0, 1, 0, 0, 0)$ — all mass at position 2
- $Q_2 = (0, 0, 0, 0, 1)$ — all mass at position 5

**KL divergence and total variation** say $P$ is equally far from $Q_1$ and $Q_2$ — because they only care that the mass is "in the wrong bin," not *how far* the wrong bin is.

**EMD** says $P$ is much closer to $Q_1$ (move 1 unit of mass by 1 step = cost 1) than to $Q_2$ (move 1 unit of mass by 4 steps = cost 4). This matches geometric intuition: $Q_1$ is a small shift, $Q_2$ is a large shift.

**Takeaway:** EMD respects the **geometry** of the underlying space. Bin-by-bin metrics are blind to it.

---

## Formal Definition

### The general Wasserstein-$p$ distance

Given two probability measures $\mu$ and $\nu$ on a metric space $(M, d)$, the **Wasserstein-$p$ distance** is:

$$W_p(\mu, \nu) = \left( \inf_{\gamma \in \Gamma(\mu, \nu)} \int_{M \times M} d(x, y)^p \; d\gamma(x, y) \right)^{1/p}$$

where $\Gamma(\mu, \nu)$ is the set of all **couplings** (joint distributions) with marginals $\mu$ and $\nu$.

The most common case is $p = 1$:

$$W_1(\mu, \nu) = \inf_{\gamma \in \Gamma(\mu, \nu)} \int_{M \times M} d(x, y) \; d\gamma(x, y)$$

This is the EMD: the cheapest way to transport mass from $\mu$ to $\nu$, where cost = mass × distance.

### Unpacking the coupling

A coupling $\gamma$ is a joint distribution on $(x, y)$ such that:
- The marginal over $x$ is $\mu$: $\int \gamma(x, y) \, dy = \mu(x)$
- The marginal over $y$ is $\nu$: $\int \gamma(x, y) \, dx = \nu(y)$

Think of $\gamma(x, y)$ as "the amount of mass moved from location $x$ to location $y$." The constraints ensure you move all of $\mu$ and fill all of $\nu$.

---

## The Discrete Case (What You'll Actually Compute)

In practice, you usually have two discrete distributions:

- **Source:** weights $\mathbf{a} = (a_1, \ldots, a_m)$ at locations $x_1, \ldots, x_m$
- **Target:** weights $\mathbf{b} = (b_1, \ldots, b_n)$ at locations $y_1, \ldots, y_n$

with $\sum a_i = \sum b_j = 1$ (both are probability distributions).

Define the **cost matrix** $C_{ij} = d(x_i, y_j)$ (distance between source $i$ and target $j$).

The EMD is the solution to a **linear program**:

$$W_1 = \min_{T \geq 0} \sum_{i,j} C_{ij} \, T_{ij}$$

subject to:

$$\sum_j T_{ij} = a_i \quad \forall i, \qquad \sum_i T_{ij} = b_j \quad \forall j$$

Here $T_{ij}$ is the **transport plan**: how much mass flows from source $i$ to target $j$.

---

## The 1D Special Case: It's Just the Area Between CDFs

When your data lives on $\mathbb{R}$ (one dimension), there's a beautiful closed-form:

$$W_1(\mu, \nu) = \int_{-\infty}^{\infty} |F_\mu(t) - F_\nu(t)| \; dt$$

where $F_\mu$ and $F_\nu$ are the cumulative distribution functions. The Wasserstein distance is literally the **area between the two CDFs**.

For discrete samples sorted in order, this reduces to:

$$W_1 = \sum_{k=1}^{n} \left| \sum_{i=1}^{k} (a_i - b_i) \right| \cdot \Delta_k$$

where $\Delta_k$ is the gap between consecutive support points. This is $O(n \log n)$ (just sort and sweep).

---

## How to Interpret the Value

| EMD Value | Interpretation |
|-----------|---------------|
| $= 0$ | Distributions are identical |
| Small | Distributions are similar; only small local rearrangements needed |
| Large | Distributions are very different; mass must travel far |

The **units** of $W_1$ are the same as the units of your ground metric $d$. If your points are in meters, $W_1$ is in meters. If you use Euclidean distance on gene expression vectors, $W_1$ is in "expression units." This is one of the nicest properties: the distance is **interpretable**.

**Rule of thumb:** Compare the EMD to the diameter of your space. If $W_1$ is close to $\text{diam}(M)$, the distributions are about as different as possible.

---

## What Data Should You Have at Hand?

### You need two things:

1. **Two distributions** — represented as:
   - Histograms / weighted point clouds: weights $\mathbf{a}$, $\mathbf{b}$ and support locations
   - Empirical samples: raw data points $\{x_1, \ldots, x_m\}$ and $\{y_1, \ldots, y_n\}$ (use uniform weights $a_i = 1/m$, $b_j = 1/n$)
   - Density functions discretized on a grid

2. **A ground metric** $d(x_i, y_j)$ — the cost of moving one unit of mass from $x_i$ to $y_j$. Common choices:
   - Euclidean distance: $\|x_i - y_j\|_2$
   - Manhattan distance: $\|x_i - y_j\|_1$
   - Custom domain-specific distance (e.g., geodesic distance on a graph)

### Checklist before computing

- ✅ Both distributions sum to the same total mass (normalize to 1)
- ✅ Ground metric is defined and makes domain sense
- ✅ Support sizes are manageable ($m, n \lesssim 10^4$ for exact LP; use entropic regularization for larger)

---

## Use Cases

### 1. Comparing distributions when geometry matters
Any time you care about *where* the mass is, not just *whether* it matches: image retrieval, color palette comparison, comparing gene expression profiles across conditions.

### 2. Generative model evaluation
The **Fréchet Inception Distance (FID)** used to evaluate GANs is related to $W_2$. EMD-based metrics penalize generated images that are "close but not quite" less harshly than bin-by-bin metrics.

### 3. Domain adaptation / transfer learning
Measure how different the source and target domain distributions are. Smaller EMD → easier transfer.

### 4. Single-cell biology
Compare cell-type abundance distributions across patients, time points, or perturbations — where cell types have a natural similarity structure.

### 5. Fairness and distribution shift detection
Monitor whether the distribution of predictions or features has drifted between training and deployment.

### 6. Document / text comparison
With word embeddings, the **Word Mover's Distance** is the EMD between the word-cloud distributions of two documents, using embedding distance as the ground metric.

---

## Computational Approaches

| Method | Complexity | When to Use |
|--------|-----------|-------------|
| Exact LP (network simplex) | $O(n^3 \log n)$ | Small problems ($n < 10^4$) |
| 1D closed-form (CDF area) | $O(n \log n)$ | Data on $\mathbb{R}$ |
| Sinkhorn (entropic regularization) | $O(n^2 / \epsilon)$ | Large problems, approximate |
| Sliced Wasserstein | $O(Ln \log n)$ | Very high-dimensional, $L$ = number of projections |

### Entropic regularization (Sinkhorn)

Adding an entropy penalty $\epsilon H(T)$ to the LP makes the problem solvable by **matrix scaling** (Sinkhorn iterations):

$$W_1^\epsilon = \min_{T \geq 0} \sum_{i,j} C_{ij} T_{ij} - \epsilon H(T), \qquad H(T) = -\sum_{i,j} T_{ij} \log T_{ij}$$

This is differentiable and GPU-friendly — hence its popularity in deep learning.

---

## Julia Usage

### Using `OptimalTransport.jl`

The main Julia package for this is [`OptimalTransport.jl`](https://github.com/JuliaOptimalTransport/OptimalTransport.jl).

```julia
using OptimalTransport, Distances

# --- Two discrete distributions ---
# Weights (must sum to the same value, typically 1)
a = [0.4, 0.3, 0.2, 0.1]  # source distribution
b = [0.1, 0.2, 0.3, 0.4]  # target distribution

# Support locations (1D points as columns, or multi-dim)
x = [1.0, 2.0, 3.0, 4.0]
y = [1.5, 2.5, 3.5, 4.5]

# Cost matrix: pairwise distances
C = pairwise(Euclidean(), x', y', dims=2)  # 4×4 matrix

# --- Exact EMD via network simplex ---
T = emd(a, b, C)             # optimal transport plan
cost = emd2(a, b, C)          # optimal transport cost (the EMD value)

println("Transport plan T:")
display(T)
println("EMD = $cost")
```

### Sinkhorn (regularized, for larger problems)

```julia
using OptimalTransport

# Entropic regularization parameter
ε = 0.01

T_reg = sinkhorn(a, b, C, ε)           # regularized transport plan
cost_reg = sinkhorn2(a, b, C, ε)        # regularized cost

println("Sinkhorn cost = $cost_reg")
```

### 1D Wasserstein (fast, closed-form)

For 1D empirical samples, you can compute it directly:

```julia
"""
    wasserstein_1d(x, y)

Compute the W₁ distance between two 1D empirical samples
via the CDF-area formula. O(n log n).
"""
function wasserstein_1d(x::Vector{Float64}, y::Vector{Float64})
    sx = sort(x)
    sy = sort(y)
    n = length(sx)
    m = length(sy)

    # Interpolate to common quantile grid
    # For equal-length samples, it's simply:
    if n == m
        return sum(abs.(sx .- sy)) / n
    end

    # General case: use CDF area
    all_pts = sort(unique(vcat(sx, sy)))
    area = 0.0
    for k in 1:(length(all_pts)-1)
        mid = (all_pts[k] + all_pts[k+1]) / 2
        F_x = count(≤(mid), sx) / n
        F_y = count(≤(mid), sy) / m
        area += abs(F_x - F_y) * (all_pts[k+1] - all_pts[k])
    end
    return area
end

# Example
x_samples = randn(500)           # samples from N(0,1)
y_samples = randn(500) .+ 1.0    # samples from N(1,1)

println("W₁ = ", wasserstein_1d(x_samples, y_samples))
# Expected ≈ 1.0 (the shift between the means)
```

### Wasserstein-2 between Gaussians (analytic)

For Gaussian distributions $\mathcal{N}(\mu_1, \Sigma_1)$ and $\mathcal{N}(\mu_2, \Sigma_2)$, there is a closed-form:

$$W_2^2 = \|\mu_1 - \mu_2\|^2 + \text{tr}\!\left(\Sigma_1 + \Sigma_2 - 2\left(\Sigma_1^{1/2}\Sigma_2\,\Sigma_1^{1/2}\right)^{1/2}\right)$$

```julia
using LinearAlgebra

function wasserstein2_gaussian(μ₁, Σ₁, μ₂, Σ₂)
    mean_term = norm(μ₁ - μ₂)^2
    sqrtΣ₁ = sqrt(Symmetric(Σ₁))
    M = sqrtΣ₁ * Σ₂ * sqrtΣ₁
    cov_term = tr(Σ₁ + Σ₂ - 2 * sqrt(Symmetric(M)))
    return sqrt(mean_term + cov_term)
end

# Example: two 2D Gaussians
μ₁ = [0.0, 0.0];  Σ₁ = [1.0 0.0; 0.0 1.0]
μ₂ = [1.0, 1.0];  Σ₂ = [2.0 0.5; 0.5 2.0]

println("W₂ = ", wasserstein2_gaussian(μ₁, Σ₁, μ₂, Σ₂))
```

---

## Duality: The Kantorovich–Rubinstein Theorem

For $W_1$, there is a powerful dual formulation:

$$W_1(\mu, \nu) = \sup_{\|f\|_{\text{Lip}} \leq 1} \left( \int f \, d\mu - \int f \, d\nu \right)$$

where the supremum is over all 1-Lipschitz functions $f$ (i.e., $|f(x) - f(y)| \leq d(x, y)$).

**Interpretation:** Find the "most discriminating" 1-Lipschitz function — one that assigns high values where $\mu$ has more mass and low values where $\nu$ has more mass. The EMD is the maximum expected difference under such a function.

This duality is exactly what **Wasserstein GANs (WGANs)** exploit: the critic network approximates the optimal 1-Lipschitz function.

---

## Quick Reference

```
What kind of data do you have?
│
├─ 1D samples
│  └─ Sort + CDF area formula → O(n log n), exact
│
├─ Both distributions are Gaussian
│  └─ Closed-form W₂ (Bures–Wasserstein)
│
├─ Discrete, moderate size (n < 10⁴)
│  └─ Exact LP via emd2() from OptimalTransport.jl
│
├─ Discrete, large (n > 10⁴)
│  └─ Sinkhorn (entropic regularization) via sinkhorn2()
│
└─ High-dimensional point clouds
   └─ Sliced Wasserstein (project to 1D, average)
```

---

## Key Takeaways

1. **EMD respects geometry** — unlike KL, TV, or chi-squared, it knows that "nearby" bins are more similar than distant ones.
2. **Units are interpretable** — $W_1$ is in the same units as your ground metric.
3. **1D is cheap** — just the area between CDFs. Higher dimensions require LP or approximation.
4. **Sinkhorn** makes it scalable and differentiable — the workhorse for ML applications.
5. **You need a ground metric** — the choice of $d$ matters enormously. Always think about what "distance between points" means in your domain.
6. **Kantorovich duality** connects optimal transport to Lipschitz functions, which is the mathematical backbone of WGANs.
