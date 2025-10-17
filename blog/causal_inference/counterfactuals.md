@def title = "Counterfactuals in Causal Inference"
@def published = "17 October 2025"
@def tags = ["causal-inference"]

# Counterfactuals in Causal Inference

## The Intuitive Idea

A **counterfactual** asks: "What would have happened if things had been different?"

It's the alternate reality we never observed. For example:
- You took medicine and recovered. The counterfactual: What would have happened if you *hadn't* taken the medicine?
- A student studied and passed. The counterfactual: Would they have passed if they *hadn't* studied?

The key insight: We can only observe one reality, but causal questions require comparing what happened to what *would have* happened.

## Mathematical Framework

### Potential Outcomes Notation

For each unit $i$ and treatment $T \in \{0, 1\}$, we define:

- $Y_i(1)$ = potential outcome if unit $i$ receives treatment ($T=1$)
- $Y_i(0)$ = potential outcome if unit $i$ does not receive treatment ($T=0$)

**The Fundamental Problem**: We observe only one of these for each unit!

$
Y_i^{\text{obs}} = T_i \cdot Y_i(1) + (1-T_i) \cdot Y_i(0)
$

> **Note**: This is a **switching equation**, not a linear model. It simply says "we observe whichever potential outcome matches our treatment status." The outcome $Y$ can be binary, continuous, count, or any other type. $T_i$ acts as a selector (indicator function), not as a regression coefficient.

### Individual Treatment Effect (ITE)

The causal effect for individual $i$ is:

$$
\tau_i = Y_i(1) - Y_i(0)
$$

This is fundamentally unobservable because we cannot observe both $Y_i(1)$ and $Y_i(0)$ for the same person.

> **Why is this useful if we can't observe it?** The individual treatment effect $\tau_i$ gives us the *conceptual framework* for thinking about causation. Even though we can't observe it for any single person, understanding that it exists helps us:
> 1. **Define what we're trying to estimate** (average effects across people)
> 2. **Identify heterogeneous treatment effects** (how effects vary by subgroups)
> 3. **Design better interventions** (target treatments to those most likely to benefit)
> 4. **Understand limitations** (recognize when personalized causal claims are unfounded)
>
> Think of it like quantum mechanics: we can't observe an electron's exact position and momentum simultaneously, but the framework lets us make probabilistic predictions that work in practice.
>
> **Can we estimate individual effects?** Yes, under certain conditions:
> - **Conditional Average Treatment Effects (CATE)**: Estimate $\mathbb{E}[\tau_i \mid X_i = x]$ for people with similar characteristics
> - **Machine learning methods**: Random forests, causal trees, meta-learners can predict individual effects
> - **Strong assumptions needed**: We're predicting based on observables, assuming people similar in X have similar treatment effects
> - **Never perfect**: We get predictions of $\hat{\tau}_i$, not the true $\tau_i$, and prediction accuracy varies

### Average Treatment Effect (ATE)

Since we can't compute individual effects, we estimate average effects:

$$
\text{ATE} = \mathbb{E}[Y_i(1) - Y_i(0)] = \mathbb{E}[Y_i(1)] - \mathbb{E}[Y_i(0)]
$$

### Counterfactual Example

Consider patient $i$ who took medicine ($T_i = 1$) and recovered ($Y_i^{\text{obs}} = 1$):

- **Factual**: $Y_i(1) = 1$ (observed: took medicine, recovered)
- **Counterfactual**: $Y_i(0) = ?$ (unobserved: would they have recovered without medicine?)

The causal effect is $\tau_i = Y_i(1) - Y_i(0) = 1 - Y_i(0)$, but $Y_i(0)$ is the **counterfactual** we can never observe.

## Key Assumptions for Identification

### 1. SUTVA (Stable Unit Treatment Value Assumption)

$$
Y_i = Y_i(T_i)
$$

No interference between units, and treatment is well-defined.

### 2. Ignorability (Unconfoundedness)

$$
\{Y_i(1), Y_i(0)\} \perp\!\!\!\perp T_i \mid X_i
$$

Given covariates $X_i$, treatment assignment is independent of potential outcomes.

### 3. Positivity (Overlap)

$$
0 < P(T_i = 1 \mid X_i = x) < 1 \quad \forall x
$$

Every unit has a positive probability of receiving either treatment.

## Estimating Counterfactuals

Under these assumptions, we can identify the ATE:

$$
\begin{align}
\text{ATE} &= \mathbb{E}[Y_i(1) - Y_i(0)] \\
&= \mathbb{E}_X[\mathbb{E}[Y_i \mid T_i=1, X_i] - \mathbb{E}[Y_i \mid T_i=0, X_i]]
\end{align}
$$

**Methods to estimate**:
- Randomized experiments (gold standard)
- Matching on covariates
- Propensity score weighting
- Regression adjustment
- Instrumental variables
- Difference-in-differences
- Regression discontinuity

## The Philosophical Core

Counterfactuals represent **causal thinking**. When we ask "Did treatment cause the outcome?" we're asking:

> Would the outcome have been different in the counterfactual world where treatment was different?

This is why correlation ≠ causation: correlation tells us about the observed world, causation requires comparing observed reality to unobserved counterfactual alternatives.