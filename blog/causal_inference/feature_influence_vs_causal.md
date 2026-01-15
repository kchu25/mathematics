@def title = "Feature Importance vs Counterfactuals: The Nuances"
@def published = "15 January 2026"
@def tags = ["causal-inference"]

# Feature Importance vs Counterfactuals: The Nuances

You're touching on something really important that trips up a lot of people in ML and causal inference!

## The Core Distinction

**Feature Importance** tells you: "How much does this feature help predict the outcome *in the current data distribution*?"

**Counterfactuals** tell you: "What would happen to the outcome if I *intervened* and changed this feature?"

These are fundamentally different questions.

## Mathematical Nuances

### 1. **Association vs Causation**

Feature importance captures association:
$$I(X_j) \propto \text{how much } X_j \text{ correlates with } Y$$

Counterfactual asks about intervention:
$$Y | \text{do}(X_j = x') - Y | \text{do}(X_j = x)$$

The "do" operator (from causal inference) means we're forcing $X_j$ to take a value, breaking its natural relationship with other variables.

**Graph surgery interpretation**: In a causal graph, $\text{do}(X_j = x)$ is equivalent to:
1. **Cutting all incoming edges** to $X_j$ (severing relationships with its parents)
2. Setting $X_j = x$ as a constant
3. Keeping all outgoing edges from $X_j$ intact (its effects on children remain)

This is why $\text{do}(X_j)$ breaks the dependence: $X_j \perp\!\!\!\perp \text{Parents}(X_j)$ after intervention.

> **Intuition about cutting incoming edges:** The incoming edges represent "what causes $X_j$" in the natural world. When you intervene, **you become the cause** of $X_j$—you've overridden the natural mechanism. The old causes don't matter anymore because you've replaced them with your experimental manipulation.
>
> Think of it like disconnecting a light from its switch and plugging it directly into a battery. The switch (parent) no longer controls the light ($X_j$), even though it used to. You've literally cut that wire.
>
> The outgoing edges stay because you're not changing how $X_j$ affects other things—you're just changing what determines $X_j$ itself.

### 2. **The Confounding Problem**

Consider:
- $Z$ (genes) causes both $X$ (exercise) and $Y$ (health)
- Exercise appears important for predicting health
- But if you force someone to exercise (intervention), the effect might be smaller

Why? The model learned:
$$P(Y | X, Z) \text{ where } X \text{ and } Z \text{ are correlated}$$

But intervention gives:
$$P(Y | \text{do}(X)) \neq P(Y | X)$$

### 3. **Feature Dependencies Matter**

When you "knock down" feature $X_j$, what happens to other features?

**In observational data:** Features co-vary naturally
$$X_1, X_2, ..., X_p \sim P(X_1, ..., X_p)$$

**In intervention:** You break those relationships
$$\text{do}(X_j = x') \implies X_j \perp\!\!\!\perp \text{Parents}(X_j)$$

The model never saw data in this intervened distribution!

### 4. **Out-of-Distribution Problem**

Your model learned:
$$f: X \to Y \text{ where } X \sim P_{\text{train}}(X)$$

After knockout:
$$X \sim P_{\text{intervention}}(X) \neq P_{\text{train}}(X)$$

The model might extrapolate poorly in this new regime.

## Why Knockdowns Sometimes Work

They work when:

1. **Feature is truly causal** (no confounding)
2. **Independence holds**: Other features don't drastically shift when you intervene
3. **Lucky interpolation**: The intervened distribution isn't too far from training data

## A Concrete Example

Imagine predicting house prices:

**Scenario 1: Square footage is important**
- Knockout: Set all houses to 1000 sq ft
- Effect: Prices likely drop (works! it's causal)

**Scenario 2: ZIP code is important**
- Knockout: Move all houses to ZIP 90210
- Effect: ??? You can't actually move houses, and if you could, you'd change neighborhood, schools, etc.
- The prediction assumes all those correlated features stay the same (they won't)

**Scenario 3: Month listed is important** (confounder)
- Model learned: Houses listed in spring sell for more
- But WHY? Better houses are strategically listed then
- Knockout: List a bad house in spring
- Effect: Won't magically make it valuable

## The Math of What Can Go Wrong

If $X_j$ is important with weight $\beta_j$ in a linear model:
$Y = \beta_0 + \beta_j X_j + \sum_{i \neq j} \beta_i X_i + \epsilon$

But $X_j = g(X_{-j}, U)$ where $U$ is unobserved:
$\mathbb{E}[Y | \text{do}(X_j = x')] \neq \mathbb{E}[Y | X_j = x']$

The intervention effect is:
$\frac{\partial}{\partial x_j} \mathbb{E}[Y | \text{do}(X_j = x_j)] = \beta_j + \sum_{i \neq j} \beta_i \frac{\partial x_i}{\partial x_j}$

That second term is the **confounding bias** – it's zero only if features are independent.

> **Important caveat about linear models:** Yes, *if* you know the true linear model and *if* you've measured all the relevant variables, then you can compute the intervention effect. The formula above shows you exactly how!
>
> BUT the big "ifs" are:
> 1. **Do you have the right model structure?** (Is it actually linear? Do you know which variables to include?)
> 2. **Have you measured all confounders?** (If there's an unmeasured $U$ that affects both $X_j$ and $Y$, you're in trouble)
> 3. **Do you know the dependencies $\frac{\partial x_i}{\partial x_j}$?** (How do other features change when you intervene on $X_j$?)
>
> In practice, you rarely know these things with certainty. That's why even with linear models, causal inference requires careful thought about what variables to adjust for (the "adjustment set" problem) and strong assumptions about what's unmeasured.

> **What does $\frac{\partial x_i}{\partial x_j}$ mean in the linear model case?**
>
> This captures how feature $x_i$ **causally depends** on feature $x_j$. It's NOT just the correlation from your regression—it's about the structural relationships between features.
>
> **Example:** Suppose $x_1$ = exercise, $x_2$ = calorie intake, $Y$ = weight loss
>
> If the causal structure is: Exercise → Calorie Intake (people who exercise eat more to compensate), then:
> - When you **intervene** to increase exercise by 1 hour, calorie intake might increase by 200 calories
> - So $\frac{\partial x_2}{\partial x_1} = 200$ (calories per hour)
>
> This means when you force $x_1$ up, $x_2$ mechanistically responds. The intervention effect on $Y$ includes both:
> - Direct effect of exercise: $\beta_1$
> - Indirect effect through induced calorie change: $\beta_2 \cdot \frac{\partial x_2}{\partial x_1} = \beta_2 \cdot 200$
>
> **The problem:** You need to know this causal structure between features! If there's no causal relationship ($X_j$ doesn't cause $X_i$), then $\frac{\partial x_i}{\partial x_j} = 0$. But figuring out which features cause which other features requires... you guessed it... more causal inference!

> **How do people actually figure out $\frac{\partial x_i}{\partial x_j}$ in practice?**
>
> Honestly? It's hard, and people use different strategies depending on their situation:
>
> **1. Domain knowledge / Expert judgment**
> - Most common approach: Subject matter experts draw the causal graph
> - "We know that education affects income, but income doesn't affect your past education"
> - Problem: Experts can be wrong, and it's subjective
>
> **2. Randomized experiments on features**
> - Gold standard: Run experiments varying $X_j$ and measure how $X_i$ responds
> - Example: A/B test showing price changes to see how customer browsing time changes
> - Problem: Expensive, sometimes unethical/impossible
>
> **3. Structural equation modeling (SEM)**
> - Assume a specific causal structure, estimate all relationships simultaneously
> - Fit equations like $X_2 = \alpha X_1 + \epsilon_2$ alongside $Y = \beta_1 X_1 + \beta_2 X_2 + \epsilon_Y$
> - Problem: Results are only as good as your structural assumptions
>
> **4. Causal discovery algorithms**
> - Algorithms like PC, GES, or constraint-based methods try to learn the graph from data
> - Use conditional independence tests to infer causal directions
> - Problem: Requires strong assumptions (faithfulness, causal sufficiency), often can't distinguish causal direction
>
> **5. Instrumental variables**
> - Find a variable that affects $X_j$ but only affects $Y$ through $X_j$
> - Can sometimes identify causal effects without knowing all the dependencies
> - Problem: Finding valid instruments is incredibly hard
>
> **6. The "dodge" approach: Target the total effect**
> - Instead of decomposing $\beta_j + \sum_i \beta_i \frac{\partial x_i}{\partial x_j}$, just estimate the total effect directly
> - Block all backdoor paths (confounders) but leave mediation paths open
> - Problem: You get the total effect, not the direct effect—might not be what you want
>
> **Reality check:** In most ML/industry settings, people either:
> - Make strong assumptions (often implicit) that features are causally independent ($\frac{\partial x_i}{\partial x_j} = 0$ for all $i \neq j$)
> - Use domain knowledge to identify which features are "descendants" of others
> - Just give up on computing true causal effects and settle for predictive models with feature importance
>
> This is why causal inference is an entire field—there's no easy, automatic way to do it!

> **Special case: Can you compute $\frac{\partial x_i}{\partial x_j}$ if the $x_i

> **What is the intervention effect?** The intervention effect is the **causal effect** of changing $X_j$ on the outcome $Y$—what happens to $Y$ when you *force* $X_j$ to change, not just observe it changing naturally.
>
> It measures the difference in outcomes when you intervene to set different values of $X_j$. For example, comparing "force everyone to exercise 3 hours/week" vs "force everyone to exercise 1 hour/week."

$\text{Intervention Effect} = \mathbb{E}[Y | \text{do}(X_j = 3)] - \mathbb{E}[Y | \text{do}(X_j = 1)]$

> This is fundamentally different from the **observational association**—comparing people who naturally exercise 3 hours vs 1 hour.

$\mathbb{E}[Y | X_j = 3] - \mathbb{E}[Y | X_j = 1]$

> The observational version mixes up the true causal effect with confounding (e.g., naturally active people might also have healthier diets, better genes, etc.).

> **Wait, so this lets you know how much Y changes when you change X? Isn't that simple?**
>
> No! The equation *defines* what the intervention effect **is** (conceptually), but it doesn't tell you **how to compute it** from your data.
>
> **You observe:** $\mathbb{E}[Y | X_j = 3]$ (average outcome for people who naturally exercise 3 hours)
>
> **You want:** $\mathbb{E}[Y | \text{do}(X_j = 3)]$ (average outcome if you *forced* everyone to exercise 3 hours)
>
> These are **different numbers**! You can't directly observe the "do" version because you didn't run an experiment. You're stuck trying to estimate $\mathbb{E}[Y | \text{do}(X_j)]$ from observational data, which requires knowing the causal structure, measuring all confounders, making untestable assumptions, and doing complex adjustments.
>
> Think of it like: I can write down $\pi$ easily, but computing its digits is hard. Similarly, writing $\mathbb{E}[Y | \text{do}(X)]$ is easy—**computing** it from real data is where all the difficulty lives.

## Bottom Line

- **Feature importance**: "This variable is useful for prediction"
- **Counterfactual**: "This variable causes the outcome"
- **The gap**: Correlation, confounding, distribution shift

Knockdowns might work, but you're making an implicit **causal assumption** that the feature is both important *and* causally upstream of the outcome with no confounding.

## Why Computing Counterfactuals is Much Harder

**Association/Feature Importance:** Computationally easy
- Reduce to linear algebra: correlations, regression coefficients, SHAP values
- Just matrix operations on observed data: $X^T X$, eigenvalues, gradients
- Complexity: $O(n \cdot p^2)$ or similar—scales well

**Counterfactuals:** Computationally hard (often impossible without assumptions)

### The Fundamental Problem: **Identification**

To compute $P(Y|\text{do}(X=x))$, you need to know the **causal graph structure**. But:

1. **Learning the causal graph from data is NP-hard** in general
   - With $p$ variables, there are super-exponentially many possible DAGs
   - Observational data alone often can't distinguish between graphs (Markov equivalence classes)
   - Example: $X \to Y$ vs $X \gets Y$ can produce identical distributions

2. **Even with the graph, you need identifiability conditions**
   - Need sufficient assumptions (no unmeasured confounders, certain graph structures)
   - The "do-calculus" (Pearl) gives rules, but checking if they apply is itself hard
   - Sometimes $P(Y|\text{do}(X))$ simply **cannot be computed** from observational data alone

### What Makes It Hard Mathematically

For associations:
$$\text{Importance}(X_j) = f(\text{observed data}) \quad \text{// direct computation}$$

For counterfactuals:
$$P(Y|\text{do}(X)) = \int P(Y|X, Z) P(Z) dZ \quad \text{if } Z \text{ is sufficient adjustment set}$$

But which $Z$? You need to:
- Know/learn the causal graph
- Apply backdoor criterion or front-door criterion
- Hope the required variables were measured
- Check positivity (overlap) assumptions

### Practical Approaches (all require strong assumptions)

1. **Randomized experiments**: Gold standard, but expensive/unethical/impossible
2. **Instrumental variables**: Need to find valid instruments (hard to verify)
3. **Regression discontinuity**: Only works in specific settings
4. **Propensity score matching**: Assumes no unmeasured confounding
5. **Difference-in-differences**: Needs parallel trends assumption
6. **Structural causal models**: Requires specifying the full causal structure

All of these are **much more involved** than computing a correlation matrix.

### The Complexity Gap

| Task | Computational Complexity | Main Challenge |
|------|-------------------------|----------------|
| Correlation | $O(np^2)$ | Matrix multiplication |
| Linear regression | $O(np^2)$ | Solve $X^TX\beta = X^Ty$ |
| Feature importance (tree) | $O(n \log n \cdot \text{depth})$ | Greedy splits |
| SHAP values | $O(2^p \cdot n)$ | Exponential in features, but tractable approximations exist |
| Causal discovery | **NP-hard** | Search over graph space |
| Counterfactual (known graph) | $O(f(\text{graph}))$ | Depends on identifiability, integration |
| Counterfactual (unknown graph) | **Intractable** | Need experiments or untestable assumptions |

**TL;DR:** Computing associations = "Look at the data." Computing counterfactuals = "Understand the world's causal structure"—which is fundamentally harder.s are neural net input features?**
>
> This is a really important question! The answer depends on what you mean:
>
> **If you're asking about the causal relationship between input features themselves:**
> - **No, the neural network can't tell you this!**
> - The neural network only learns $f: (x_1, ..., x_p) \to y$
> - It has no information about how $x_i$ causally depends on $x_j$
> - These dependencies exist in the real world, *before* data enters your model
> - Example: A neural net predicting health from (exercise, calories, weight) can't tell you whether exercise causes calorie intake to change
>
> **If you're asking about derivatives through the network:**
> - Yes, you can compute $\frac{\partial y}{\partial x_j}$ via backpropagation (gradients)
> - But this is NOT the same as $\frac{\partial x_i}{\partial x_j}$
> - $\frac{\partial y}{\partial x_j}$ tells you: "how does the output change if I change input $x_j$ while holding other inputs fixed?"
> - This is still just observational/predictive, not causal about feature interactions
>
> **The fundamental issue:**
> - $\frac{\partial x_i}{\partial x_j}$ is about relationships **between features in the world**
> - Your model (neural net or otherwise) only sees the features after they've been determined
> - To know $\frac{\partial x_i}{\partial x_j}$, you need:
>   - Experiments varying $x_j$ and observing $x_i$
>   - Domain knowledge about the data-generating process
>   - Assumptions about the causal structure
>
> **Bottom line:** Neural networks (or any predictive model) fundamentally cannot tell you about causal relationships between their input features. That information has to come from outside the model.

## Can You Learn Feature Dependencies $\frac{\partial x_i}{\partial x_j}$ from Data?

This is a critical question because without knowing these dependencies, you can't properly compute intervention effects. The answer depends heavily on what kind of data you have.

### The Easy Case: You Have Perturbation/Experimental Data

If you have data where $x_j$ was experimentally manipulated, you're in good shape:
>
> **When it's possible (with interventional data):**
> - If you have **experimental data** where you intervened on $x_j$, then yes!
> - Regress $x_i$ on $x_j$ using only the experimental data: $x_i = \gamma x_j + \epsilon$
> - The coefficient $\gamma$ estimates $\frac{\partial x_i}{\partial x_j}$
> - This works because intervention breaks confounding
>
> **When it's very hard (observational data only):**
>
> 1. **Causal discovery algorithms** can sometimes learn it:
>    - Methods: PC algorithm, GES, LiNGAM (for linear non-Gaussian), NOTEARS (for neural nets)
>    - They use conditional independence patterns to infer causal directions
>    - **Major limitations:**
>      - Assume **causal sufficiency** (no unmeasured confounders affecting multiple features)
>      - Assume **faithfulness** (causal structure faithfully generates independencies)
>      - Often can only identify up to **Markov equivalence** (can't distinguish $X \to Y$ from $X \gets Y$ without more assumptions)
>      - Break down with many variables, complex interactions, or when assumptions violated
>
> 2. **Functional Causal Models (FCMs)**:
>    - Assume specific functional forms: $x_i = f_i(\text{Parents}(x_i), \epsilon_i)$
>    - Use asymmetries in the noise distributions to identify causal direction
>    - **Limitations:** Requires very specific assumptions about noise structure
>
> 3. **Time-series data** helps:
>    - If you have temporal ordering: cause must precede effect
>    - Granger causality, VAR models, dynamic causal modeling
>    - **Limitations:** Correlation in time ≠ causation (could still be confounded)
>
> **The fundamental identifiability problem:**
>
> Consider two possibilities:
> - (A) $x_j \to x_i \to y$
> - (B) $x_i \to x_j \to y$
>
> Both can produce the **exact same observational distribution** $P(x_i, x_j, y)$!
>
> Without additional assumptions (functional forms, noise distributions, temporal order, or experiments), pure observational data cannot distinguish them.
>
> **What people actually do:**
> - **Best case:** Run experiments to directly measure $\frac{\partial x_i}{\partial x_j}$
> - **Second best:** Use causal discovery with strong assumptions + domain knowledge to validate
> - **Pragmatic:** Make simplifying assumptions (features independent, known causal order)
> - **Honest:** Acknowledge uncertainty and do sensitivity analysis on different causal structures
>
> **TL;DR:** You can *sometimes* learn $\frac{\partial x_i}{\partial x_j}$ from data, but it requires either experiments or very strong (often untestable) assumptions. Pure observational data alone usually isn't enough.

> **What is the intervention effect?** The intervention effect is the **causal effect** of changing $X_j$ on the outcome $Y$—what happens to $Y$ when you *force* $X_j$ to change, not just observe it changing naturally.
>
> It measures the difference in outcomes when you intervene to set different values of $X_j$. For example, comparing "force everyone to exercise 3 hours/week" vs "force everyone to exercise 1 hour/week."

$\text{Intervention Effect} = \mathbb{E}[Y | \text{do}(X_j = 3)] - \mathbb{E}[Y | \text{do}(X_j = 1)]$

> This is fundamentally different from the **observational association**—comparing people who naturally exercise 3 hours vs 1 hour.

$\mathbb{E}[Y | X_j = 3] - \mathbb{E}[Y | X_j = 1]$

> The observational version mixes up the true causal effect with confounding (e.g., naturally active people might also have healthier diets, better genes, etc.).

> **Wait, so this lets you know how much Y changes when you change X? Isn't that simple?**
>
> No! The equation *defines* what the intervention effect **is** (conceptually), but it doesn't tell you **how to compute it** from your data.
>
> **You observe:** $\mathbb{E}[Y | X_j = 3]$ (average outcome for people who naturally exercise 3 hours)
>
> **You want:** $\mathbb{E}[Y | \text{do}(X_j = 3)]$ (average outcome if you *forced* everyone to exercise 3 hours)
>
> These are **different numbers**! You can't directly observe the "do" version because you didn't run an experiment. You're stuck trying to estimate $\mathbb{E}[Y | \text{do}(X_j)]$ from observational data, which requires knowing the causal structure, measuring all confounders, making untestable assumptions, and doing complex adjustments.
>
> Think of it like: I can write down $\pi$ easily, but computing its digits is hard. Similarly, writing $\mathbb{E}[Y | \text{do}(X)]$ is easy—**computing** it from real data is where all the difficulty lives.

## Bottom Line

- **Feature importance**: "This variable is useful for prediction"
- **Counterfactual**: "This variable causes the outcome"
- **The gap**: Correlation, confounding, distribution shift

Knockdowns might work, but you're making an implicit **causal assumption** that the feature is both important *and* causally upstream of the outcome with no confounding.

## Why Computing Counterfactuals is Much Harder

**Association/Feature Importance:** Computationally easy
- Reduce to linear algebra: correlations, regression coefficients, SHAP values
- Just matrix operations on observed data: $X^T X$, eigenvalues, gradients
- Complexity: $O(n \cdot p^2)$ or similar—scales well

**Counterfactuals:** Computationally hard (often impossible without assumptions)

### The Fundamental Problem: **Identification**

To compute $P(Y|\text{do}(X=x))$, you need to know the **causal graph structure**. But:

1. **Learning the causal graph from data is NP-hard** in general
   - With $p$ variables, there are super-exponentially many possible DAGs
   - Observational data alone often can't distinguish between graphs (Markov equivalence classes)
   - Example: $X \to Y$ vs $X \gets Y$ can produce identical distributions

2. **Even with the graph, you need identifiability conditions**
   - Need sufficient assumptions (no unmeasured confounders, certain graph structures)
   - The "do-calculus" (Pearl) gives rules, but checking if they apply is itself hard
   - Sometimes $P(Y|\text{do}(X))$ simply **cannot be computed** from observational data alone

### What Makes It Hard Mathematically

For associations:
$$\text{Importance}(X_j) = f(\text{observed data}) \quad \text{// direct computation}$$

For counterfactuals:
$$P(Y|\text{do}(X)) = \int P(Y|X, Z) P(Z) dZ \quad \text{if } Z \text{ is sufficient adjustment set}$$

But which $Z$? You need to:
- Know/learn the causal graph
- Apply backdoor criterion or front-door criterion
- Hope the required variables were measured
- Check positivity (overlap) assumptions

### Practical Approaches (all require strong assumptions)

1. **Randomized experiments**: Gold standard, but expensive/unethical/impossible
2. **Instrumental variables**: Need to find valid instruments (hard to verify)
3. **Regression discontinuity**: Only works in specific settings
4. **Propensity score matching**: Assumes no unmeasured confounding
5. **Difference-in-differences**: Needs parallel trends assumption
6. **Structural causal models**: Requires specifying the full causal structure

All of these are **much more involved** than computing a correlation matrix.

### The Complexity Gap

| Task | Computational Complexity | Main Challenge |
|------|-------------------------|----------------|
| Correlation | $O(np^2)$ | Matrix multiplication |
| Linear regression | $O(np^2)$ | Solve $X^TX\beta = X^Ty$ |
| Feature importance (tree) | $O(n \log n \cdot \text{depth})$ | Greedy splits |
| SHAP values | $O(2^p \cdot n)$ | Exponential in features, but tractable approximations exist |
| Causal discovery | **NP-hard** | Search over graph space |
| Counterfactual (known graph) | $O(f(\text{graph}))$ | Depends on identifiability, integration |
| Counterfactual (unknown graph) | **Intractable** | Need experiments or untestable assumptions |

**TL;DR:** Computing associations = "Look at the data." Computing counterfactuals = "Understand the world's causal structure"—which is fundamentally harder.