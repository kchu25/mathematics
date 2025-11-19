@def title = "Function Composition and Homogeneity"
@def published = "18 November 2025"
@def tags = ["general-topics"]
# Function Composition and Homogeneity

## Key Insight

**A composition of functions is 1-homogeneous if and only if NO function in the chain is 0-homogeneous.**

## Mathematical Definitions

### k-Homogeneous Function
A function $f$ is **k-homogeneous** (or **homogeneous of degree k**) if:
$$f(\alpha x) = \alpha^k \cdot f(x)$$

- **1-homogeneous**: $f(\alpha x) = \alpha \cdot f(x)$ (scales linearly)
- **0-homogeneous**: $f(\alpha x) = f(x)$ (scale-invariant)

> **Note**: This is the standard definition from mathematics. Formally, a function $f: \mathbb{R}^n \to \mathbb{R}^m$ is homogeneous of degree $k$ if $f(tx) = t^k f(x)$ for all $t > 0$ (or sometimes for all $t \in \mathbb{R}$ depending on context). Also called "positive homogeneity" or "Euler homogeneity". In economics and physics, this concept describes how systems respond to scaling.

## Composition Rules

For a composition $h(x) = f(g(x))$:

### If both are 1-homogeneous:
$$h(\alpha x) = f(g(\alpha x)) = f(\alpha \cdot g(x)) = \alpha \cdot f(g(x)) = \alpha \cdot h(x)$$ 
✅ **Result: 1-homogeneous**

### If g is 0-homogeneous:
$$h(\alpha x) = f(g(\alpha x)) = f(g(x)) = h(x)$$
❌ **Result: 0-homogeneous** (scaling absorbed!)

### General Rule:
If $f$ is $k$-homogeneous and $g$ is $m$-homogeneous:
$$h(\alpha x) = f(g(\alpha x)) = f(\alpha^m \cdot g(x)) = \alpha^{km} \cdot f(g(x)) = \alpha^{km} \cdot h(x)$$

The composition is **$(km)$-homogeneous**.

## Proof: Why ALL Functions Must Be 1-Homogeneous

**Theorem**: For composition $h = f_n \circ f_{n-1} \circ \cdots \circ f_1$ to be 1-homogeneous, **every** $f_i$ must be 1-homogeneous.

**Proof**:

Suppose $f_i$ is $k_i$-homogeneous for each $i$. Then:

$$\begin{align}
h(\alpha x) &= f_n(f_{n-1}(\cdots f_1(\alpha x))) \\
&= f_n(f_{n-1}(\cdots (\alpha^{k_1} f_1(x)))) \\
&= f_n(f_{n-1}(\cdots (\alpha^{k_1 k_2} f_2(f_1(x))))) \\
&= f_n(\alpha^{k_1 k_2 \cdots k_{n-1}} f_{n-1}(\cdots f_1(x))) \\
&= \alpha^{k_1 k_2 \cdots k_n} f_n(f_{n-1}(\cdots f_1(x))) \\
&= \alpha^{k_1 k_2 \cdots k_n} h(x)
\end{align}$$

For 1-homogeneity, we need:
$$\alpha^{k_1 k_2 \cdots k_n} = \alpha$$

This must hold for **all** $\alpha > 0$. 

Taking logarithms: $k_1 k_2 \cdots k_n = 1$

**Key insight**: The only way the product equals 1 **for all possible homogeneities** is if:
$$k_1 = k_2 = \cdots = k_n = 1$$

**Why?**
- If any $k_i = 0$, then product = 0 (0-homogeneous composition)
- If any $k_i \neq 1$, you need other $k_j$ values to compensate, but this only works for specific choices
- Only $k_i = 1$ for all $i$ works **universally**

Therefore, **every function in the composition must be 1-homogeneous**. ∎

## Critical Observation

**Any 0-homogeneous function acts as a "scaling absorber":**
- Input: $\alpha x$
- Output: Same as $f(x)$
- The factor $\alpha$ is completely eliminated

**Once scaling is absorbed, it cannot be recovered by subsequent operations.**

## BatchNorm Example

### BatchNorm Normalization
$$\text{normalize}(x) = \frac{x - \mu}{\sigma}$$

This is **0-homogeneous**:
$$\text{normalize}(\alpha x) = \frac{\alpha x - \alpha \mu}{\alpha \sigma} = \frac{\alpha(x - \mu)}{\alpha \sigma} = \frac{x - \mu}{\sigma} = \text{normalize}(x)$$

### Network with BatchNorm
For network $N(x) = W_n \circ \text{BN} \circ W_{n-1} \circ \cdots \circ W_1(x)$:

$$N(\alpha x) = W_n(\text{BN}(W_{n-1}(\cdots \alpha x))) = W_n(\text{BN}(\text{something}))$$

Since BN is 0-homogeneous, regardless of what comes before:
$$N(\alpha x) = \text{does not scale with } \alpha$$

❌ **The entire network becomes 0-homogeneous**

## Skip Connections (Residual Connections)

### The Addition Problem

Addition is **NOT a homogeneous operation** in general:
$$(f + g)(\alpha x) = f(\alpha x) + g(\alpha x)$$

This only equals $\alpha(f(x) + g(x))$ if **both** $f$ and $g$ have the same homogeneity.

### Residual Block with BatchNorm

Consider a typical ResNet block:
$$y = x + F(x)$$

where $F(x) = W_2 \circ \text{BN} \circ \text{ReLU} \circ W_1(x)$

**What happens with scaling?**

- **Skip path**: $x \to \alpha x$ (1-homogeneous, just identity)
- **Residual path**: $F(\alpha x) = F(x)$ (0-homogeneous due to BN)

$$y(\alpha x) = \alpha x + F(\alpha x) = \alpha x + F(x)$$

But for 1-homogeneity we need:
$$y(\alpha x) = \alpha(x + F(x))$$

**These are NOT equal!** ❌

### The Mismatch

$$\begin{align}
\text{Actual:} \quad & \alpha x + F(x) \\
\text{Needed:} \quad & \alpha x + \alpha F(x)
\end{align}$$

The skip connection scales with $\alpha$, but the BatchNorm path doesn't. This creates an **imbalance** that breaks homogeneity.

### Visual Example

With $\alpha = 2$:
- Skip: $2x$ (doubled)
- BN branch: $F(x)$ (unchanged)
- Sum: $2x + F(x)$ (not proportional to $x + F(x)$)

**The relative contribution of each path changes with input scale!**

## Tricks to Restore 1-Homogeneity

### Why Standard BatchNorm Fails

Standard BatchNorm with affine parameters:
$$\text{BN}(x) = \gamma \frac{x - \mu}{\sigma} + \beta$$

Two problems:
1. The normalization $\frac{x - \mu}{\sigma}$ is **0-homogeneous** (absorbs scaling)
2. The affine transform ($\gamma$, $\beta$) makes it not even homogeneous at all

Removing $\gamma$ and $\beta$ doesn't help - you still have the 0-homogeneous normalization.

### Solution 1: Remove Normalization Entirely ✅

Use only 1-homogeneous operations:
$$y = x + W_2(\text{ReLU}(W_1(x)))$$

**Why it works:**
- Linear layers: 1-homogeneous
- ReLU: 1-homogeneous  
- Composition: 1-homogeneous
- Addition of 1-homogeneous functions: 1-homogeneous

**Downside:** Training instability without normalization

### Solution 2: Weight Normalization ✅

**Key idea:** Normalize the *weights*, not the *activations*!

$$W_{\text{norm}} = \frac{g}{\|w\|} w$$

Then: $f(x) = W_{\text{norm}} \cdot x$

**Why it works:**
$$f(\alpha x) = \frac{g}{\|w\|} (w \cdot \alpha x) = \alpha \frac{g}{\|w\|} (w \cdot x) = \alpha f(x)$$

✅ **1-homogeneous in $x$** because we only normalized the parameters, not the activations!

**Advantages:**
- Maintains 1-homogeneity
- Provides training stability
- Works well in practice

### Solution 3: NFNet (Normalizer-Free Networks) ✅

**Key idea:** Carefully scaled skip connections without any normalization

$$y = x + \alpha \cdot F(x)$$

where $\alpha$ is a fixed scalar (e.g., $\alpha = 1/\sqrt{N}$ for $N$ layers).

**Why it works:**
If $F$ is 1-homogeneous:
$$y(\beta x) = \beta x + \alpha F(\beta x) = \beta x + \alpha \beta F(x) = \beta(x + \alpha F(x)) = \beta \, y(x)$$

✅ **1-homogeneous!**

**Advantages:**
- No normalization layers needed
- State-of-the-art results on ImageNet
- Faster training (no batch statistics)

### Solution 4: Use Only 1-Homogeneous Activations ✅

- ✅ ReLU: $\text{ReLU}(\alpha x) = \alpha \cdot \text{ReLU}(x)$ for $\alpha > 0$
- ✅ Leaky ReLU: Also 1-homogeneous
- ❌ Sigmoid, Tanh: Not homogeneous
- ✅ Any piecewise linear function through origin: 1-homogeneous

### Summary Table

| Method | 1-Homogeneous? | Training Stability | Practical? |
|--------|----------------|-------------------|------------|
| Remove all normalization | ✅ | ⚠️ Poor | ⚠️ Challenging |
| Weight Normalization | ✅ | ✅ Good | ✅ Yes |
| NFNet (scaled residuals) | ✅ | ✅ Good | ✅ Yes (SOTA) |
| BatchNorm (no affine) | ❌ Still 0-homo | ✅ Good | ❌ No |
| LayerNorm | ❌ Still 0-homo | ✅ Good | ❌ No |

**Recommended approach:** Use **Weight Normalization** or **NFNet-style** architectures for 1-homogeneous networks with good training stability.

## Conclusion

**To maintain 1-homogeneity through a composition:**
- ✅ All functions must be 1-homogeneous
- ❌ Even ONE 0-homogeneous function breaks the chain
- ❌ Skip connections mixing different homogeneities break it too
- The 0-homogeneous function "absorbs" all scaling information
- Addition requires **matching homogeneity** on both branches
- Subsequent layers cannot recover what was lost
- **Practical solutions exist:** Weight Normalization and NFNet architectures