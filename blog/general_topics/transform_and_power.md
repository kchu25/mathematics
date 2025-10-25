@def title = "Why You Can't Transform Power Indices After non-linear trasnformation"
@def published = "25 October 2025"
@def tags = ["general-topics"]

# Why You Can't Transform Power Indices After non-linear trasnformation

## The Question

If we compute a Banzhaf power index in log space, can we simply apply $\exp()$ to transform it back to original space? **No!**

## Counter Example

Consider a simple cooperative game with 3 players and characteristic function in log space:

### Log-space payoffs:
- $v(\{1\}) = \log(2) \approx 0.693$
- $v(\{2\}) = \log(3) \approx 1.099$
- $v(\{1,2\}) = \log(10) \approx 2.303$

### Banzhaf power index computation in log space:

Player 1's marginal contribution when player 2 is present:
$$v(\{1,2\}) - v(\{2\}) = \log(10) - \log(3) = \log(10/3) \approx 1.204$$

Player 2's marginal contribution when player 1 is present:
$$v(\{1,2\}) - v(\{1\}) = \log(10) - \log(2) = \log(5) \approx 1.609$$

In log space: **Player 2 has higher Banzhaf index than Player 1**

### Naively applying exp() to the power indices:
- $\exp(1.204) \approx 3.33$
- $\exp(1.609) \approx 5.00$

### The correct approach (transform first, then compute):

Original space payoffs:
- $v'(\{1\}) = 2$
- $v'(\{2\}) = 3$
- $v'(\{1,2\}) = 10$

Marginal contributions in original space:
- Player 1: $10 - 3 = \boxed{7}$
- Player 2: $10 - 2 = \boxed{8}$

### The Problem

Notice that:
$$7 \neq 3.33 \quad \text{and} \quad 8 \neq 5.00$$

## Why This Happens

The Banzhaf index involves **averaging/summing** marginal contributions across coalitions. Since $\exp()$ is a **non-linear** function:

$$\exp\left(\frac{1}{n}\sum_{i} x_i\right) \neq \frac{1}{n}\sum_{i} \exp(x_i)$$

More generally:
$$\exp(\mathbb{E}[X]) \neq \mathbb{E}[\exp(X)]$$

## The Verdict

**You must transform payoffs to original space FIRST, then compute the power index.**

There is no valid way to transform the power index after computation.

## What Does Each Approach Measure?

- **Power index in log space**: Measures influence in terms of **multiplicative/percentage changes**
- **Power index in original space**: Measures influence in terms of **absolute changes**

Choose the space that matches your notion of "influence" for your application, but compute the index directly in that space.

## What If The Transformation Is Linear?

If the transformation from transformed space to original space is **linear**, i.e.:
$v'(S) = a \cdot v(S) + b$

Then you **CAN** transform the power index afterwards!

### Why Linear Transformations Work

For the Banzhaf index, marginal contributions in original space are:
$v'(S \cup \{i\}) - v'(S) = [a \cdot v(S \cup \{i\}) + b] - [a \cdot v(S) + b]$
$= a \cdot [v(S \cup \{i\}) - v(S)]$

The constant $b$ cancels out, and we get:
$\text{Marginal contribution in original space} = a \times \text{Marginal contribution in transformed space}$

Since the Banzhaf index is computed by averaging/summing marginal contributions:
$\phi_i(\text{original}) = \frac{1}{n}\sum_S [a \cdot \Delta_i(S)] = a \cdot \frac{1}{n}\sum_S \Delta_i(S) = a \cdot \phi_i(\text{transformed})$

**Linearity of expectation/summation allows us to pull out the constant!**

### Practical Application

If your transformation is:
- $v'(S) = a \cdot v(S) + b$ (affine transformation)
- Or simply $v'(S) = a \cdot v(S)$ (linear scaling)

Then:
1. Compute Banzhaf index in transformed space: $\phi_i$
2. Transform back: $\phi'_i = a \cdot \phi_i$ (the constant $b$ doesn't matter)

### Summary Table

| Transformation Type | Can Transform After? | Reason |
|---------------------|----------------------|--------|
| Linear: $v' = a \cdot v + b$ | ✅ **YES** | Linearity of expectation |
| Logarithmic: $v' = \exp(v)$ | ❌ **NO** | Non-linear |
| Polynomial: $v' = v^2$ | ❌ **NO** | Non-linear |
| General non-linear | ❌ **NO** | Non-linearity breaks averaging |