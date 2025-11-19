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

## Normalization Layers and Homogeneity

### All Normalization Layers Are 0-Homogeneous

The core normalization operation in BatchNorm, LayerNorm, GroupNorm, and InstanceNorm is:
$$\text{normalize}(x) = \frac{x - \mu}{\sigma}$$

where $\mu$ and $\sigma$ are computed over different dimensions depending on the normalization type.

**This is always 0-homogeneous:**
$$\text{normalize}(\alpha x) = \frac{\alpha x - \alpha \mu}{\alpha \sigma} = \frac{\alpha(x - \mu)}{\alpha \sigma} = \frac{x - \mu}{\sigma} = \text{normalize}(x)$$

The key insight: **both the mean and standard deviation scale linearly with $\alpha$**, so they cancel out in the division!

### Comparison of Normalization Types

| Type | Statistics computed over | 0-homogeneous? |
|------|-------------------------|----------------|
| **BatchNorm** | Batch + spatial dimensions (across samples) | ✅ Yes |
| **LayerNorm** | Feature dimensions (per sample) | ✅ Yes |
| **GroupNorm** | Groups of channels (per sample) | ✅ Yes |
| **InstanceNorm** | Spatial dimensions per channel (per sample) | ✅ Yes |

**All of them break 1-homogeneity equally!** The difference is only in:
- Which dimensions they normalize over
- Whether they use batch statistics (BatchNorm) or per-sample statistics (others)
- Training vs inference behavior

### Mathematical Proof (applies to all)

For any normalization that computes $\mu$ and $\sigma$ from the input:

$$\begin{align}
\mu(\alpha x) &= \alpha \mu(x) \\
\sigma(\alpha x) &= \alpha \sigma(x) \\
\text{normalize}(\alpha x) &= \frac{\alpha x - \alpha \mu(x)}{\alpha \sigma(x)} = \frac{x - \mu(x)}{\sigma(x)} = \text{normalize}(x)
\end{align}$$

❌ **All normalization layers are 0-homogeneous**

### Network with Any Normalization
For network $N(x) = W_n \circ \text{Norm} \circ W_{n-1} \circ \cdots \circ W_1(x)$:

$$N(\alpha x) = W_n(\text{Norm}(W_{n-1}(\cdots \alpha x))) = W_n(\text{Norm}(\text{something}))$$

Since normalization is 0-homogeneous, regardless of what comes before:
$$N(\alpha x) = \text{does not scale with } \alpha$$

❌ **The entire network becomes 0-homogeneous**

### Why This Matters

While LayerNorm and GroupNorm have advantages over BatchNorm (no batch dependency, better for small batches, etc.), **they all equally destroy 1-homogeneity**. You cannot fix the homogeneity problem by switching normalization types - you must either:
1. Remove normalization entirely
2. Use Weight Normalization instead
3. Use NFNet-style architectures

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

### The "Impossible" Question: Can We Make BatchNorm 1-Homogeneous?

**Short answer: Not while keeping standard BatchNorm behavior.**

**Why it's fundamentally challenging:**

The entire *point* of BatchNorm is to normalize inputs to have standardized statistics:
$$\text{BN}(x) = \frac{x - \mu}{\sigma}$$

This operation **must** be scale-invariant (0-homogeneous) when $\mu$ and $\sigma$ are computed from the input being normalized.

### The ACTUAL Trick: Decouple Statistics from the Normalized Input! 🎯

**Key insight:** What if we compute $\mu$ and $\sigma$ from a **different source** than the input we're normalizing?

#### Method 1: Ghost BatchNorm

Compute statistics from a **reference batch**:

1. **Reference batch**: Get $\mu_{\text{ref}}, \sigma_{\text{ref}}$ from batch $B_{\text{ref}}$
2. **Normalize current batch** $B$ using those statistics:
   $$y = \frac{x - \mu_{\text{ref}}}{\sigma_{\text{ref}}}$$

**Problem:** The mean centering still breaks homogeneity!

**Solution:** Remove mean centering:
$$y = \frac{x}{\sigma_{\text{ref}}}$$

**Is this 1-homogeneous?**
$$\frac{\alpha x}{\sigma_{\text{ref}}} = \alpha \frac{x}{\sigma_{\text{ref}}}$$

✅ **YES!** Because $\sigma_{\text{ref}}$ is independent of the current input $x$.

#### Method 2: Fixed Statistics (Inference-Mode Trick)

In BatchNorm **inference mode**, statistics are **fixed** (running averages from training):
$$\text{BN}_{\text{inference}}(x) = \gamma \frac{x - \mu_{\text{running}}}{\sigma_{\text{running}}} + \beta$$

**Modified version** (no mean centering, no bias):
$$\text{BN}_{\text{modified}}(x) = \gamma \frac{x}{\sigma_{\text{running}}} = \frac{\gamma}{\sigma_{\text{running}}} x$$

✅ **This is 1-homogeneous!** It's just multiplication by a constant.

#### Method 3: RMS Normalization with Fixed Scale

$$\text{FixedRMSNorm}(x) = \frac{x}{\sigma_{\text{fixed}}} \cdot g$$

where $\sigma_{\text{fixed}}$ is from dataset statistics or running average, and $g$ is learnable.

✅ **1-homogeneous!** Because the denominator doesn't depend on $x$.

### Summary: The Decoupling Trick

| Method | Statistics from | Normalized input | 1-homo? |
|--------|----------------|------------------|---------|
| Standard BatchNorm | Current batch | Same batch | ❌ No |
| Ghost BatchNorm (no mean) | Reference batch | Different batch | ✅ Yes! |
| Inference BN (no mean) | Running average | Current input | ✅ Yes! |
| Fixed RMSNorm | Pre-computed | Current input | ✅ Yes! |

**The key:** When $\sigma$ doesn't depend on the input $x$ being normalized, then $x/\sigma$ is 1-homogeneous!

**The trade-off:** You lose the adaptive, input-dependent normalization that makes BatchNorm effective for training stability. These approaches are closer to having a learned constant scaling factor (like Weight Normalization).

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

### Solution 5: Remove Bias Terms! ✅

**The bias problem:**

A linear layer with bias:
$f(x) = Wx + b$

**Is this 1-homogeneous?**
$f(\alpha x) = W(\alpha x) + b = \alpha Wx + b \neq \alpha(Wx + b)$

❌ **No!** The bias term $b$ doesn't scale.

**Why bias breaks homogeneity:**
- The term $Wx$ is 1-homogeneous: $W(\alpha x) = \alpha Wx$
- But $b$ is just a constant offset - it doesn't scale at all
- So $f(\alpha x) = \alpha Wx + b$ while we need $\alpha f(x) = \alpha Wx + \alpha b$

**The fix: Remove all bias terms!**

Use:
$f(x) = Wx$

**Now it's 1-homogeneous:**
$f(\alpha x) = W(\alpha x) = \alpha Wx = \alpha f(x)$ ✅

**Practical implications:**

In PyTorch/TensorFlow, set `bias=False`:
```python
# PyTorch
nn.Linear(in_features, out_features, bias=False)
nn.Conv2d(in_channels, out_channels, kernel_size, bias=False)

# TensorFlow/Keras
Dense(units, use_bias=False)
Conv2D(filters, kernel_size, use_bias=False)
```

**What about the expressiveness loss?**

You might think: "But bias terms are important for expressiveness!"

**Counter-argument:**
1. If you're using normalization (even the decoupled kind), it often includes learnable scale/shift parameters that can compensate
2. For 1-homogeneous networks, the bias would break the property anyway
3. Many successful architectures (like NFNet) work fine without biases in most layers
4. The skip connections in residual networks can provide the "offset" functionality

**Exception:** You can have bias in the **final output layer** if you don't need end-to-end 1-homogeneity (e.g., for classification, you often don't care if the final logits are 1-homogeneous).

### Summary: Building a Fully 1-Homogeneous Network

To make a network 1-homogeneous, you must:

1. ✅ Use linear layers **without bias**: $f(x) = Wx$
2. ✅ Use 1-homogeneous activations: ReLU, Leaky ReLU
3. ✅ Either:
   - Remove normalization entirely, OR
   - Use Weight Normalization, OR
   - Use decoupled normalization (statistics from different source), OR
   - Use NFNet-style scaled skip connections
4. ✅ Ensure skip connections maintain homogeneity: both branches must be 1-homogeneous
5. ❌ **No bias terms anywhere** (except possibly final layer)
6. ❌ **No 0-homogeneous operations** (no standard BatchNorm/LayerNorm with coupled statistics)

**Example 1-homogeneous residual block:**
$y = x + \alpha \cdot W_2(\text{ReLU}(W_1(x)))$

where $W_1, W_2$ have no bias, and $\alpha$ is a fixed scalar.

### Practical Recommendations

| Method | 1-Homogeneous? | Training Stability | Practical? |
|--------|----------------|-------------------|------------|
| Remove all normalization | ✅ | ⚠️ Poor | ⚠️ Challenging |
| Weight Normalization | ✅ | ✅ Good | ✅ Yes |
| NFNet (scaled residuals) | ✅ | ✅ Good | ✅ Yes (SOTA) |
| Ghost BN (no mean) | ✅ | ⚠️ Moderate | ⚠️ Limited use |
| Inference BN (no mean) | ✅ | ❌ Only at inference | ❌ No |
| Standard BatchNorm | ❌ | ✅ Good | ✅ But not 1-homo |
| LayerNorm | ❌ | ✅ Good | ✅ But not 1-homo |

**Best practical approaches:** Use **Weight Normalization** or **NFNet-style** architectures for 1-homogeneous networks with good training stability.

## Conclusion

**To maintain 1-homogeneity through a composition:**
- ✅ All functions must be 1-homogeneous
- ❌ Even ONE 0-homogeneous function breaks the chain
- ❌ Skip connections mixing different homogeneities break it too
- The 0-homogeneous function "absorbs" all scaling information
- Addition requires **matching homogeneity** on both branches
- Subsequent layers cannot recover what was lost
- **Trick exists:** Decouple normalization statistics from the input being normalized
- **Practical solutions:** Weight Normalization and NFNet architectures