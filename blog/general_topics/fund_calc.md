@def title = "The Fundamental Theorem of Calculus: From 1D to Many Dimensions"
@def published = "19 December 2025"
@def tags = ["general-topics"]

# The Fundamental Theorem of Calculus: From 1D to Many Dimensions

## The Single-Variable Version (The Classic)

You know how derivatives and integrals are "opposite" operations? That's the FTC in a nutshell.

**Part 1 (The "Accumulation" Version):**

If you have a function $f(x)$ and you define:
$$F(x) = \int_a^x f(t) \, dt$$

Then $F'(x) = f(x)$. In other words, if you accumulate $f$ from some starting point $a$ up to $x$, then the rate of change of that accumulation is just $f$ itself!

**Part 2 (The "Evaluation" Version - The One We Care About):**

If $F(x)$ is an antiderivative of $f(x)$ (meaning $F'(x) = f(x)$), then:
$$\int_a^b f(x) \, dx = F(b) - F(a)$$

This is beautiful! To find the total change from $a$ to $b$, you just evaluate the antiderivative at the endpoints.

### The Intuitive Picture

Imagine you're driving a car. Your speedometer shows $f(t)$ (speed at time $t$). The FTC says:

- **Integral** $\int_0^T f(t) \, dt$ = total distance traveled
- **Derivative** $F'(t)$ = instantaneous speed
- **Connection**: Distance change = $F(T) - F(0)$ = integral of speed

The FTC connects these two perspectives!

---

## Why This Matters for the SHAP Paper

The key insight in that blog post is using FTC to write:

$$f(x_i) - f(r_i) = \int_{r_i}^{x_i} \frac{df}{dz} \, dz$$

where we're integrating along a path from reference $r_i$ to input $x_i$.

But here's the thing: **neural networks have many inputs**, not just one! So we need the **multivariate version** of FTC.

---

## The Multivariate Version (Line Integrals)

When you have a function $f(x_1, x_2, \ldots, x_n)$ that depends on multiple variables, things get more interesting.

### The Setup

Suppose we want to go from a reference point $\mathbf{r} = (r_1, r_2, \ldots, r_n)$ to our input $\mathbf{x} = (x_1, x_2, \ldots, x_n)$.

We can parameterize a straight-line path:
$$\mathbf{\gamma}(t) = \mathbf{r} + t(\mathbf{x} - \mathbf{r}) = (1-t)\mathbf{r} + t\mathbf{x}, \quad t \in [0,1]$$

When $t=0$, we're at $\mathbf{r}$. When $t=1$, we're at $\mathbf{x}$.

### The Multivariate FTC (Line Integral Version)

$$f(\mathbf{x}) - f(\mathbf{r}) = \int_0^1 \frac{d}{dt} f(\mathbf{\gamma}(t)) \, dt$$

By the chain rule:
$$\frac{d}{dt} f(\mathbf{\gamma}(t)) = \nabla f \cdot \frac{d\mathbf{\gamma}}{dt} = \sum_{i=1}^n \frac{\partial f}{\partial x_i} \cdot (x_i - r_i)$$

So we get:
$$f(\mathbf{x}) - f(\mathbf{r}) = \int_0^1 \left( \sum_{i=1}^n \frac{\partial f}{\partial x_i}(\mathbf{\gamma}(t)) \cdot (x_i - r_i) \right) dt$$

Rearranging:
$$f(\mathbf{x}) - f(\mathbf{r}) = \sum_{i=1}^n \left( \int_0^1 \frac{\partial f}{\partial x_i}(\mathbf{r} + t(\mathbf{x} - \mathbf{r})) \, dt \right) (x_i - r_i)$$

### What This Means

Each feature $i$ gets an **attribution coefficient**:
$$\phi_i(\mathbf{x}, \mathbf{r}) = \int_0^1 \frac{\partial f}{\partial x_i}(\mathbf{r} + t(\mathbf{x} - \mathbf{r})) \, dt$$

And these satisfy:
$$f(\mathbf{x}) - f(\mathbf{r}) = \sum_{i=1}^n \phi_i(\mathbf{x}, \mathbf{r}) \cdot (x_i - r_i)$$

This is **exactly the DeepLIFT decomposition**!

---

## The Connection to SHAP's Independence Assumption

Here's where it gets subtle. The blog post shows:

$$\mathbb{E}_{S \sim \pi}[f(\mathbf{x}_S, \mathbf{r}_{\bar{S}}) - f(\mathbf{x}_{S \setminus \{i\}}, \mathbf{r}_{\overline{S \setminus \{i\}}})] \approx \phi_i(\mathbf{x}, \mathbf{r}) \cdot (x_i - r_i)$$

The **left side** is the true Shapley value: average over all coalitions.

The **right side** is what DeepLIFT computes: integrate gradients along a single path.

### Why Are These "Equal"?

The multivariate FTC tells us how to decompose $f(\mathbf{x}) - f(\mathbf{r})$ along ONE specific path (the straight line from $\mathbf{r}$ to $\mathbf{x}$).

But the Shapley value wants us to average over ALL possible paths (all coalitions $S$).

**The independence assumption says**: "The gradient $\frac{\partial f}{\partial x_i}$ doesn't depend on which other features are at $\mathbf{x}$ vs $\mathbf{r}$."

If this is true, then:
- Integrating along the straight path (DeepLIFT)
- Averaging over all coalition paths (Shapley)

...give the same answer!

### The Math

For any coalition $S$, the marginal contribution of feature $i$ is:
$$f(\mathbf{x}_{S \cup \{i\}}, \mathbf{r}_{\overline{S \cup \{i\}}}) - f(\mathbf{x}_S, \mathbf{r}_{\bar{S}}) = \int_0^1 \frac{\partial f}{\partial x_i}(\mathbf{x}_S, r_i + t(x_i - r_i), \mathbf{r}_{\overline{S \cup \{i\}}}) \, dt \cdot (x_i - r_i)$$

**Under independence**, the gradient doesn't depend on $S$:
$$\frac{\partial f}{\partial x_i}(\mathbf{x}_S, z_i, \mathbf{r}_{\overline{S \cup \{i\}}}) \approx \frac{\partial f}{\partial x_i}(\mathbf{r} + (z_i - r_i)\mathbf{e}_i)$$

So averaging over coalitions gives:
$$\mathbb{E}_S[\text{marginal}] \approx \int_0^1 \frac{\partial f}{\partial x_i}(\mathbf{r} + t(\mathbf{x} - \mathbf{r})) \, dt \cdot (x_i - r_i) = \phi_i(\mathbf{x}, \mathbf{r}) \cdot (x_i - r_i)$$

Which is exactly what the multivariate FTC gave us!

---

## The Big Picture

1. **Single-variable FTC**: $f(b) - f(a) = \int_a^b f'(x) \, dx$
2. **Multivariate FTC**: $f(\mathbf{x}) - f(\mathbf{r}) = \sum_i \left(\int_0^1 \frac{\partial f}{\partial x_i}(\mathbf{\gamma}(t)) \, dt\right) (x_i - r_i)$
3. **DeepLIFT**: Uses this to decompose output changes into feature contributions
4. **SHAP**: Claims this approximates Shapley values under independence
5. **The Catch**: Independence rarely holds in practice!

The FTC bridges **local information** (gradients) and **global change** (output difference). DeepLIFT extends this from 1D to multi-D, then SHAP claims it approximates the coalition-averaged Shapley value when features don't interact.

Pretty neat, right? Though as the blog notes... it's more of a useful approximation than a rigorous equality!