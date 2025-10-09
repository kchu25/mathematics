@def title = "Deep Dive: Mutual Information"
@def published = "9 October 2025"
@def tags = ["information-theory"]

# Deep Dive: Mutual Information

## What Is It, Really?

Mutual information (MI) measures how much knowing one variable tells you about another. More precisely, it quantifies the reduction in uncertainty about $X$ when you learn $Y$, and vice versa.

The formula is elegant:

$$I(X; Y) = \sum_{x,y} p(x,y) \log \frac{p(x,y)}{p(x)p(y)}$$

Or equivalently:

$$I(X; Y) = H(X) - H(X|Y) = H(Y) - H(Y|X)$$

where $H$ denotes entropy. It's the amount of information $X$ and $Y$ share.

## Why It's Special

Here's what makes MI genuinely unique:

### 1. **Perfect Symmetry**
$I(X; Y) = I(Y; X)$ always. There's no "direction" to the relationship. This is profound: MI doesn't care about cause and effect, only about information overlap. Like correlation, it's symmetric—but unlike correlation, it captures *all* dependencies, not just linear ones.

### 2. **Nonlinear Relationships**
Correlation only catches linear dependencies. MI catches *everything*. If $Y = X^2$ and $X$ is uniform on $[-1, 1]$, the correlation might be zero (due to symmetry), but MI is positive—it knows they're deeply related.

### 3. **Always Non-Negative**
$I(X; Y) \geq 0$ always, with equality if and only if $X$ and $Y$ are independent. It's a proper "distance from independence."

### 4. **Invariant to Transformations**
If you apply any invertible function $f$ to $X$, then $I(f(X); Y) = I(X; Y)$. The information content is preserved under reparameterization.

## MI vs KL Divergence

Here's the connection people often miss: **MI is a special case of KL divergence**.

$I(X; Y) = D_{KL}(p(x,y) \| p(x)p(y))$

MI is the KL divergence between the joint distribution and the product of marginals. So MI measures how far the joint distribution is from independence.

> **Quick Proof**: Start with the KL divergence definition:
> 
> $D_{KL}(p(x,y) \| p(x)p(y)) = \sum_{x,y} p(x,y) \log \frac{p(x,y)}{p(x)p(y)}$
> 
> Split the logarithm:
> 
> $= \sum_{x,y} p(x,y) \log p(x,y) - \sum_{x,y} p(x,y) \log p(x) - \sum_{x,y} p(x,y) \log p(y)$
> 
> The first term is $-H(X,Y)$ (negative joint entropy). For the second term:
> 
> $\sum_{x,y} p(x,y) \log p(x) = \sum_x p(x) \log p(x) \sum_y p(y|x) = \sum_x p(x) \log p(x) = -H(X)$
> 
> Similarly, the third term gives $-H(Y)$. Therefore:
> 
> $D_{KL}(p(x,y) \| p(x)p(y)) = -H(X,Y) + H(X) + H(Y) = I(X;Y)$
> 
> This also immediately shows why $I(X;Y) \geq 0$ (since KL divergence is always non-negative) and why $I(X;Y) = 0$ iff $X$ and $Y$ are independent (since $D_{KL} = 0$ iff the distributions are equal).

### Key Differences:

| Mutual Information | KL Divergence |
|-------------------|---------------|
| Symmetric: $I(X;Y) = I(Y;X)$ | Asymmetric: $D_{KL}(P \| Q) \neq D_{KL}(Q \| P)$ |
| Special case: divergence from independence | General: divergence between any two distributions |
| Always involves two random variables | Can compare distributions over single variable |
| Bounded by entropies: $I(X;Y) \leq \min(H(X), H(Y))$ | Unbounded (can be infinite) |

Think of it this way: KL divergence is the Swiss Army knife of comparing distributions. MI is what you get when you specifically ask "how far from independent are these variables?"

## Real-Valued Variables: Yes, Absolutely!

People often see MI applied to discrete variables and assume that's the natural domain. **Wrong**. MI applies beautifully to continuous variables—you just need differential entropy:

$$I(X; Y) = \int \int p(x,y) \log \frac{p(x,y)}{p(x)p(y)} \, dx \, dy$$

### The Gaussian Case

For jointly Gaussian $(X, Y)$ with correlation $\rho$:

$$I(X; Y) = -\frac{1}{2} \log(1 - \rho^2)$$

This is remarkable! For Gaussians only, MI and correlation contain the same information. But this is the exception, not the rule.

### Estimation Challenge

Here's the catch: estimating MI for continuous variables is *hard*. With discrete variables, you just count frequencies. With continuous ones, you need to estimate densities, which is notoriously difficult in high dimensions. Common approaches:

- **Binning** (discretize and compute discrete MI—crude but simple)
- **Kernel density estimation** (smooth but slow)
- **k-NN methods** (Kraskov et al., 2004—surprisingly effective)
  - Core idea: estimate $I(X;Y) = H(X) + H(Y) - H(X,Y)$ using nearest neighbor distances
  - For each point $(x_i, y_i)$:
    - Find distance $\epsilon_i$ to its $k$-th nearest neighbor in joint space $(X,Y)$
    - Count $n_x(i)$ = neighbors within $\epsilon_i$ in $X$-space only
    - Count $n_y(i)$ = neighbors within $\epsilon_i$ in $Y$-space only
  - Estimate: $\hat{I}(X;Y) = \psi(k) - \langle \psi(n_x + 1) + \psi(n_y + 1) \rangle + \psi(N)$
  - Where $\psi$ is the digamma function, $N$ is sample size, $\langle \cdot \rangle$ is average over samples
  - **Why it works**: In high-density regions (low entropy), $\epsilon_i$ is small, so $n_x$ and $n_y$ are small. In low-density regions (high entropy), $\epsilon_i$ is large. The estimator cleverly balances these density ratios to approximate MI without explicitly estimating densities
  - No binning needed, adapts to local density, works well even in moderate dimensions
- **Neural estimators** (MINE algorithm—modern deep learning approach)

## Surprising Perspectives People Ignore

### 1. **MI Can Be Zero Even With Perfect Dependence**

Wait, what? If $X \sim \mathcal{N}(0, 1)$ and $Y = |X|$, then $Y$ is completely determined by $X$. But $I(X; Y) < I(X; X) = H(X)$. Why? Because you lose information in the mapping—you can't recover $X$ from $Y$. MI is about *shared* information, not one-way dependence.

Actually, let me be more precise: $I(X; Y)$ can be *less than maximal* even with functional dependence if the function isn't invertible. For truly zero MI with dependence, you'd need something like $X$ uniform on $[-1, 1]$ and $Y = X^2$... but wait, that has positive MI. 

The real surprise: **MI can be zero for dependent variables only when they're uncorrelated in a very specific information-theoretic sense.** If $Y$ is a function of $X$, MI is zero only if that function is constant (trivial dependence).

### 2. **Conditional MI Can Be Higher Than Unconditional MI**

This is wild: $I(X; Y | Z)$ can exceed $I(X; Y)$. Learning $Z$ can reveal *more* relationship between $X$ and $Y$ than existed marginally. This happens with "explaining away" in Bayesian networks.

Example: $X$ and $Y$ are independent coin flips. $Z = X \oplus Y$ (XOR). Marginally, $I(X; Y) = 0$. But $I(X; Y | Z) = 1$ bit! Once you know $Z$, learning $X$ tells you $Y$ exactly.

### 3. **MI Is Not a Metric**

Despite being a "distance from independence," MI doesn't satisfy the triangle inequality. $I(X; Y) + I(Y; Z) \not\geq I(X; Z)$ in general. There are normalized versions (variation of information) that are metrics, but plain MI isn't.

### 4. **The Data Processing Inequality**

If you form a Markov chain $X \to Y \to Z$, then:

$$I(X; Z) \leq I(X; Y)$$

Processing $Y$ to get $Z$ can only lose information about $X$, never gain it. This is fundamental in information theory and explains why lossy compression is lossy—you can't recover what you've thrown away.

### 5. **Multivariate MI Gets Weird**

For three variables, there's no single "mutual information of $X$, $Y$, and $Z$." You have various options:
- **Total correlation**: $I(X; Y; Z) = H(X) + H(Y) + H(Z) - H(X,Y,Z)$
- **Interaction information**: $I(X; Y; Z) = I(X; Y|Z) - I(X; Y)$

And here's the kicker: **interaction information can be negative**! It measures whether $Z$ creates or destroys dependence between $X$ and $Y$. This has no intuitive analog in correlation.

### 6. **MI and Machine Learning**

MI is all over ML, sometimes hiding:
- **Information Bottleneck**: Deep learning as successive compression preserving task-relevant information
- **InfoGAN**: Learning disentangled representations by maximizing MI between latent codes and observations
- **Attention mechanisms**: Attention weights approximately maximize MI between queries and the relevant parts of keys/values, routing information where it's most informative (though this connection is more conceptual than rigorously proven)
- **Feature selection**: Maximizing MI with target while minimizing redundancy

But here's what people miss: **maximizing MI can lead to overfitting**. Just because two variables share information doesn't mean one should be used to predict the other in a finite sample. The bias-variance tradeoff applies to information too.

## The Philosophy of MI

At its core, MI is about **surprise reduction**—or more precisely, uncertainty reduction. 

Here's the concrete intuition: Suppose you don't know $Y$ yet. Your uncertainty about $Y$ is its entropy $H(Y)$, measured in bits. Now someone tells you $X$. Your remaining uncertainty about $Y$ is now $H(Y|X)$ (conditional entropy). 

The difference $H(Y) - H(Y|X)$ is how much uncertainty you lost—how much less surprised you'll be when you finally learn $Y$. That's exactly $I(X; Y)$.

**Concrete Example**: You're waiting for a package.
- $Y$ = which day it arrives (Mon-Fri), each equally likely
- $H(Y) = \log_2(5) \approx 2.32$ bits of uncertainty
- Now you learn $X$ = "tracking says it's in your city"
- If packages in your city always arrive the next day, $H(Y|X) = 0$
- Then $I(X; Y) = 2.32$ bits—learning $X$ eliminated all surprise about $Y$
- If tracking status is completely uninformative, $H(Y|X) = H(Y)$ and $I(X; Y) = 0$

The "surprise" language comes from information theory's roots: $-\log p(y)$ is called the "surprisal" of outcome $y$ (rare events are surprising). MI is the expected reduction in surprisal about $Y$ when you learn $X$.

It's also beautifully agnostic. MI doesn't care if the relationship is:
- Linear or nonlinear
- Monotonic or non-monotonic
- Causal or just correlated
- Simple or hideously complex

It just asks: does knowing one reduce uncertainty about the other?

### Making This Intuitive: The "Guessing Game" Framing

Here's the clearest way to think about MI: **It measures how much better you can guess $Y$ after seeing $X$**.

**Example 1: Perfect Circle**
- $X$ is angle around a circle: $X \sim \text{Uniform}[0, 2\pi)$
- $Y = \sin(X)$

This is highly nonlinear, non-monotonic, and complicated. But here's what matters: **if I tell you $X$, you know $Y$ exactly**. Your uncertainty about $Y$ goes from $H(Y) \approx 1$ bit (for uniform $Y$ on $[-1,1]$) to $H(Y|X) = 0$. So $I(X;Y)$ is maximal for this relationship!

Correlation? Near zero (sine wave averages out). But MI says: "These variables share maximum information."

**Example 2: The XOR Gate**
- $X, Y$ are independent fair coin flips
- $Z = X \oplus Y$ (XOR: equals 1 if $X \neq Y$)

Try guessing $Y$ without any info: you're right 50% of the time. Now I tell you $Z$:
- If $Z=0$, then $Y=X$
- If $Z=1$, then $Y \neq X$

Wait, you don't know $X$ either! So you still guess $Y$ correctly 50% of the time. $I(Y; Z) = 0$.

But now I tell you **both** $Z$ and $X$: suddenly you know $Y$ with certainty! $I(Y; Z|X) = 1$ bit.

This shows MI doesn't care about *how* the relationship works—just whether knowing one helps predict the other.

**Example 3: Backwards Causation Doesn't Matter**
- $X$ = it rained yesterday
- $Y$ = the ground is wet today

Clearly $X$ causes $Y$. $I(X; Y)$ is positive.

Now flip it:
- $X$ = the ground is wet today  
- $Y$ = it rained yesterday

Same $I(X; Y)$! MI is perfectly symmetric. It doesn't care that wet ground doesn't *cause* rain. It only cares that observing wet ground **reduces your uncertainty** about whether it rained.

**Example 4: Hideously Complex**
- $X \sim \mathcal{N}(0, 1)$
- $Y = \begin{cases} X^3 - 2X & \text{if } X > 0 \\ \sin(5X) + X^2 & \text{if } X \leq 0 \end{cases}$

This relationship is a nightmare. Different formulas in different regions, non-monotonic, weird. 

But MI doesn't care! It just asks: "If I give you $X=0.7$, can you tell me $Y$?" Yes, exactly! $Y = 0.7^3 - 1.4 = -1.057$. So $H(Y|X) = 0$, meaning $I(X;Y) = H(Y)$ (maximal MI given $H(Y)$).

### The Core Insight

MI is like a universal detector that says: **"I don't care HOW $X$ and $Y$ are related—through multiplication, sine waves, logical operations, causal chains, or spaghetti code. I only care: does $X$ contain clues about $Y$?"**

Think of it as playing 20 questions:
- High MI = learning $X$ is like getting lots of good answers to your questions about $Y$
- Low MI = learning $X$ is like getting useless answers
- Zero MI = learning $X$ tells you literally nothing

The relationship could be simple ($Y=2X$) or baroque ($Y = \lfloor \sin(17X) \cdot e^{|X|} \rfloor \mod 42$), but if knowing $X$ lets you narrow down $Y$, MI captures it.

## Final Thought

The most underappreciated fact about MI: **it's the only way to measure dependence that satisfies certain natural axioms** (Rényi, 1959). If you want a symmetric, non-negative measure that's zero iff independence holds and that decomposes properly over variable combinations, you basically must use MI (or a monotonic function of it).

So when someone asks "why mutual information?", the answer is: because the axioms of information theory lead us there inevitably. It's not just *a* way to measure dependence—it's *the* way, in a precise mathematical sense.