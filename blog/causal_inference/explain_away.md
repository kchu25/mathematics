@def title = "Do Operator and Explaining Away: Connection Summary"
@def published = "17 October 2025"
@def tags = ["causal-inference"]

# Do Operator and Explaining Away: Connection Summary

## What is Explaining Away?

**Explaining away** (Berkson's paradox/collider bias) occurs when two independent causes of a common effect become negatively correlated when you **condition** on that effect.

### Classic Example

```
Talent → Admission ← Hard Work
```

**Setup**: $T, H \in \{0,1\}$ are independent with $P(T=1) = P(H=1) = 0.5$

**Admission rule**: $A = 1 \iff T = 1 \text{ OR } H = 1$ (logical OR)

**Before conditioning** (independence holds):
$$P(T=1 \mid H=0) = P(T=1) = 0.5$$

**After conditioning on $A=1$** (explaining away occurs):

**Computing the joint probabilities:**

Since $T$ and $H$ are independent: $P(T,H) = P(T) \cdot P(H) = 0.5 \times 0.5 = 0.25$ for each combination.

The admission rule is deterministic: $A=1$ if $(T=1 \text{ OR } H=1)$, otherwise $A=0$.

```
Case 1: T=0, H=0  →  P(T,H) = 0.25, A=0  →  P(T,H,A=0) = 0.25 (not admitted)
Case 2: T=0, H=1  →  P(T,H) = 0.25, A=1  →  P(T,H,A=1) = 0.25 (admitted)
Case 3: T=1, H=0  →  P(T,H) = 0.25, A=1  →  P(T,H,A=1) = 0.25 (admitted)
Case 4: T=1, H=1  →  P(T,H) = 0.25, A=1  →  P(T,H,A=1) = 0.25 (admitted)
```

Therefore: $P(A=1) = 0.25 + 0.25 + 0.25 = 0.75$

**Now compute the explaining away effect:**

$$P(T=1 \mid H=0, A=1) = \frac{P(T=1, H=0, A=1)}{P(H=0, A=1)} = \frac{0.25}{0.25} = 1.0$$

versus:

$$P(T=1 \mid A=1) = \frac{P(T=1, A=1)}{P(A=1)} = \frac{0.5}{0.75} = \frac{2}{3}$$

**Result**: $P(T=1 \mid H=0, A=1) = 1.0 > \frac{2}{3}$

**Interpretation**: Learning $H=0$ among admitted students makes you *certain* $T=1$ (they had to get in somehow!). Independent causes become negatively correlated when conditioning on their common effect.

---



## The Connection to Do Operator

### Key Insight

**Explaining away happens through conditioning, and the do operator prevents it.**

### Conditioning Creates Explaining Away

When you **condition on a collider** (common effect):
- Induces spurious correlation between independent causes
- Creates statistical dependency with no causal basis
- "Opens" a blocked path in the causal graph

### Intervention Prevents It

When you **intervene on a collider**:
$$P(T=1 \mid \text{do}(A=1)) = P(T=1) = 0.5$$

Why no explaining away?
- $\text{do}(A=1)$ cuts: $T \rightarrow A$ and $H \rightarrow A$
- $A$ becomes independent of its former causes
- No spurious correlation arises

---

## The Deeper Pattern

### What is a "Cause" in the Probability Sense?

**Important distinction**: In probability theory alone, there is no notion of causation!

- **Probability**: $P(Y \mid X)$ tells us how $Y$ is distributed given knowledge of $X$
- **Causation**: $X$ causes $Y$ means changing $X$ would change $Y$

A "cause" in causal inference is a variable that appears as a parent in a causal DAG—meaning it has a direct causal mechanism influencing another variable. This is a **structural assumption** beyond probability.

**Example:**
```
Ice Cream Sales ← Temperature → Drowning
```

- Temperature is a **cause** of both Ice Cream Sales and Drowning
- Ice Cream Sales is **not a cause** of Drowning (no arrow)
- Yet probabilistically: $P(\text{Drowning} \mid \text{Ice Cream}=\text{high})$ is elevated!

Probability sees correlation. Causation requires the DAG structure.

### Why Conditioning on a Cause Fails

When we condition on a cause, we're asking: "Among cases where the cause took value $x$, what's the distribution of the effect?"

**The problem**: The cause naturally took value $x$ for a reason!

**Example: Education → Income**
```
    Ability
    ↙   ↘
Education → Income
```

**Conditioning**: $P(\text{Income} \mid \text{Education}=\text{college})$

This asks: "Among people who went to college, what's their income?"

But **who goes to college?** People with:
- High ability (which also boosts income directly)
- Wealthy families (which also affects income)
- Good schools (another confounder)

When you condition on Education=college, you're **inadvertently selecting** for these confounders!

$P(\text{Income} \mid \text{Education}=\text{college}) = \sum_a P(\text{Income} \mid \text{Education}=\text{college}, \text{Ability}=a) \cdot P(\text{Ability}=a \mid \text{Education}=\text{college})$

Notice: $P(\text{Ability} \mid \text{Education}=\text{college}) \neq P(\text{Ability})$

People who went to college have **different ability distribution** than the general population!

**Intervention**: $P(\text{Income} \mid \text{do}(\text{Education}=\text{college}))$

This asks: "If we forced random people to go to college, what would their income be?"

$P(\text{Income} \mid \text{do}(\text{Education}=\text{college})) = \sum_a P(\text{Income} \mid \text{Education}=\text{college}, \text{Ability}=a) \cdot P(\text{Ability}=a)$

Notice: We use $P(\text{Ability})$—the **unconditional** distribution!

By breaking the arrow $\text{Ability} \rightarrow \text{Education}$, we ensure ability is distributed the same as in the general population.

### The Core Issue

**Conditioning respects the natural causal structure**—it includes all the reasons why $X=x$ occurred.

**Intervention breaks the structure**—it makes $X=x$ happen regardless of the usual causes.

To isolate the causal effect of $X$ on $Y$, we need to:
1. Remove the influence of $X$

### The Unified View

The do operator exists because **conditioning can mislead about causation** in multiple ways:

1. **Confounding**: $P(Y \mid X=x)$ includes spurious paths through confounders
2. **Collider bias**: Conditioning on effects creates spurious associations

**Solution**: Replace problematic conditioning with intervention
- $\text{do}(X=x)$ breaks incoming arrows to $X$
- Isolates true causal effects
- Prevents both confounding and collider-induced correlations

---



## Summary

Yes, the do operator is intimately related to explaining away:

- **Explaining away** shows how conditioning on colliders misleads
- **Do operator** provides intervention-based alternative to conditioning
- Both illustrate why **observation ≠ causation**
- The do operator is motivated by phenomena like explaining away that demonstrate how conditioning can create or hide causal relationships

**Core principle**: True causal inference requires intervention (or techniques that simulate it), not mere conditioning.s parents on $X$ (graph surgery)
2. Let only $X$'s direct causal effect on $Y$ remain (keep outgoing arrows). (to clarify that we preserve the outgoing edges from $X$ during graph surgery).

### The Unified View

The do operator exists because **conditioning can mislead about causation** in multiple ways:

1. **Confounding**: $P(Y \mid X=x)$ includes spurious paths through confounders
2. **Collider bias**: Conditioning on effects creates spurious associations

**Solution**: Replace problematic conditioning with intervention
- $\text{do}(X=x)$ breaks incoming arrows to $X$
- Isolates true causal effects
- Prevents both confounding and collider-induced correlations

---

## Summary

Yes, the do operator is intimately related to explaining away:

- **Explaining away** shows how conditioning on colliders misleads
- **Do operator** provides intervention-based alternative to conditioning
- Both illustrate why **observation ≠ causation**
- The do operator is motivated by phenomena like explaining away that demonstrate how conditioning can create or hide causal relationships

**Core principle**: True causal inference requires intervention (or techniques that simulate it), not mere conditioning.s causal effect on $Y$ remain

Conditioning does neither. Intervention does both.

### Summary Table

| Problem | Issue | What Goes Wrong | Solution |
|---------|-------|-----------------|----------|
| **Confounding** | Conditioning on cause doesn't isolate causal effects | $P(Y \mid X=x)$ includes selection bias via $P(\text{Confounders} \mid X=x)$ | Use $P(Y \mid \text{do}(X=x))$ with $P(\text{Confounders})$ |
| **Explaining Away** | Conditioning on effect creates spurious associations | Independent causes become correlated via $P(\text{Cause1} \mid \text{Effect})$ | Avoid conditioning on colliders; use intervention if needed |

### The Unified View

The do operator exists because **conditioning can mislead about causation** in multiple ways:

1. **Confounding**: $P(Y \mid X=x)$ includes spurious paths through confounders
2. **Collider bias**: Conditioning on effects creates spurious associations

**Solution**: Replace problematic conditioning with intervention
- $\text{do}(X=x)$ breaks incoming arrows to $X$
- Isolates true causal effects
- Prevents both confounding and collider-induced correlations

---

## Summary

Yes, the do operator is intimately related to explaining away:

- **Explaining away** shows how conditioning on colliders misleads
- **Do operator** provides intervention-based alternative to conditioning
- Both illustrate why **observation ≠ causation**
- The do operator is motivated by phenomena like explaining away that demonstrate how conditioning can create or hide causal relationships

**Core principle**: True causal inference requires intervention (or techniques that simulate it), not mere conditioning.