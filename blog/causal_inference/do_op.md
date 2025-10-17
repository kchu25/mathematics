@def title = "Understanding the Do Operator in Causal Inference"
@def published = "16 October 2025"
@def tags = ["causal-inference"]


# Understanding the Do Operator in Causal Inference

A conversational yet rigorous guide to the do operator, covering its foundations, mechanics, and key distinctions.

---

## Prerequisites

Before diving into the do operator, we need to establish a few foundational concepts:

### 1. Conditional Probability vs. Intervention

The crucial distinction: **observing that X happened naturally is fundamentally different from forcing X to happen.**

Consider: P(cholesterol is low | person takes statins) includes people who were already healthier and more likely to take statins. But if we *force* someone to take statins (randomize them in a trial), we break that selection bias.

### 2. Causal Graphs (DAGs)

We represent causal relationships using Directed Acyclic Graphs (DAGs):
- Nodes represent variables
- Directed edges show direct causal influence
- Example: Smoking → Lung Cancer ← Genetics

These arrows claim actual causal mechanisms, not just correlation.

### 3. Structural Causal Models (SCMs)

SCMs formalize the idea that each variable is determined by its parents in the graph plus some randomness:

```
X = f_X(U_X)
Y = f_Y(X, U_Y)
```

where U's are exogenous noise terms, and the functions encode causal mechanisms.

---

## What IS the Do Operator?

The **do operator**, written as **do(X = x)** or just **do(x)**, is **an operator that modifies probability distributions**.

It is neither a set nor a function in the traditional sense. Think of it as an instruction to modify the causal model itself.

When you write P(Y | do(X = x)), you're asking:
> "What's the probability distribution of Y if we *surgically intervene* to set X to value x?"

### The Mechanics

Here's what happens when you apply do(X = x):

1. **Graph surgery**: Remove all incoming edges to X in your causal graph
2. **Fix X**: Set X = x deterministically, ignoring its usual causes
3. **Propagate**: Let the causal effects flow forward through the remaining edges
4. **Observe**: Read off the distribution of Y

This is called "graph mutilation" - you're literally cutting X off from its causes and forcing it to a value.

### The Key Distinction

- **P(Y | X = x)**: "Among cases where X naturally equals x, what's the distribution of Y?"
- **P(Y | do(X = x))**: "If we force X to equal x, what's the distribution of Y?"

**These can be wildly different!**

Classic example: P(tar on fingers | lung cancer) is high, but P(lung cancer | do(tar on fingers = high)) might be zero - painting tar on your fingers doesn't cause cancer, even though cancer patients often have tar-stained fingers from smoking.

### Mathematical Formalization

In terms of SCMs, doing do(X = x) means replacing the structural equation for X with the constant equation X := x, then computing probabilities in this mutilated model.

For observable Y and intervention X = x:

```
P(Y = y | do(X = x)) = P^{do(x)}(Y = y)
```

This is the probability Y takes value y in the intervened world.

---

## Deep Dive: Conditioning vs. Intervention

This distinction is really the heart of why we need causal inference at all.

### The Fundamental Problem

- **Conditioning** = selecting a subset of the natural world
- **Intervening** = creating a new world that might never naturally occur

### Concrete Example: Ice Cream and Drowning

Suppose we observe: P(drowning | ice cream sales are high) is elevated!

Should we ban ice cream? Of course not. Both ice cream sales and drowning have a common cause: hot weather.

**Causal graph:**
```
Temperature → Ice Cream Sales
     ↓
  Drowning
```

**Conditioning (observation):**

P(drowning | ice cream sales = high) asks: "In the subset of days where ice cream sales happen to be high, what's the drowning rate?"

Those are mostly hot days! So you're inadvertently also conditioning on high temperature, which genuinely increases drowning risk. You're NOT breaking the causal link between temperature and drowning.

**Intervening:**

P(drowning | do(ice cream sales = high)) asks: "If we *force* ice cream sales to be high (through subsidies), what happens to drowning?"

When you do(ice cream = high):
1. Cut the arrow Temperature → Ice Cream Sales
2. Fix ice cream sales to high regardless of temperature
3. Let effects propagate forward

But there's no arrow from ice cream to drowning! So P(drowning | do(ice cream = high)) ≈ P(drowning). The intervention has no effect.

### Why Conditioning "Sees" Spurious Relationships

When you condition on X = x, you're restricting attention to cases where X = x naturally occurred. But X = x occurring naturally might have happened *because of* certain values of X's parents, which might *also* influence Y through other paths.

**Mathematical intuition:**

By Bayes' rule:
```
P(Y | X = x) = P(Y, X = x) / P(X = x)
```

This depends on the joint distribution P(Y, X), which encodes all associations, causal or not.

But P(Y | do(X = x)) is computed in a **different model** where X is exogenous. It only depends on the causal path from X to Y.

### Selection Bias Perspective

Example: Education → Income, and Ability → both Education and Income.

```
    Ability
    ↙   ↘
Education → Income
```

**If you compute P(income | education = college):**
- You're looking at people who went to college
- But who goes to college? Partially determined by ability!
- So you're inadvertently selecting for higher ability
- Higher ability independently boosts income
- You **overestimate** education's causal effect

**But P(income | do(education = college)):**
- We randomly assign people to college (breaking Ability → Education)
- Ability is distributed the same in both groups
- We isolate education's true causal effect

### The Randomized Trial Connection

This is why **randomized controlled trials are the gold standard**. When you randomize treatment:
- You literally implement do(Treatment = x)
- You break all arrows into Treatment
- Treatment becomes independent of all its would-be causes
- So P(Outcome | Treatment = x in RCT) = P(Outcome | do(Treatment = x))

Observation doesn't do this. Observation respects the natural causal structure, warts and all.

---

## Graph Surgery: What Gets Cut?

**Critical clarification**: When you apply do(X = x), you cut **INCOMING arrows to X only**, not all connections!

The outgoing arrows from X to its effects stay intact - that's the whole point. We want to see X's causal influence.

### Example: Education and Income

**Before intervention:**
```
    Ability
    ↙   ↘
Education → Income
```

**After do(Education = college):**
```
    Ability
    ✗   ↘
Education → Income
```

We cut:
- Ability → Education (incoming to Education)

We keep:
- Education → Income (outgoing from Education)
- Ability → Income (unrelated to our intervention target)

### Why This Makes Sense

- **Incoming arrows** represent causes of X - things that normally determine X's value. When we intervene, we override those natural causes.
- **Outgoing arrows** represent effects of X - causal mechanisms by which X influences other variables. We keep these because we're intervening on X precisely to see what happens downstream!

### The Formal Statement

When computing P(Y | do(X = x)), you modify the SCM by:
1. Removing all equations that define X based on its parents
2. Replacing them with X := x (a constant)
3. Keeping all other structural equations unchanged

This is equivalent to:
- Deleting all edges Pa → X (parents pointing INTO X)
- Keeping all edges X → Ch (X pointing to its children)
- Keeping all other edges in the graph untouched

---

## Reconciling Graph Surgery and Averaging

This seems contradictory at first: we "cut" parents but also "average over" them. How does this work?

### Two Perspectives on the Same Operation

**Perspective 1: Graph surgery (cutting parents)**
- Describes how we modify the causal model structurally

**Perspective 2: Averaging over parents**
- Describes how we compute probabilities in the modified model

### What "Cutting" Actually Means

When we "cut" incoming edges to X:
- X is no longer causally determined by its parents in the model
- X becomes exogenous (externally set)
- The structural equation changes: X = f_X(Parents, U_X) → X := x

### But Parents Still Exist!

**Key insight**: Cutting the causal link doesn't make parent variables disappear!
- They still exist in the world
- They still have their natural distribution
- They still influence other variables (like Y) through other paths
- We just broke their influence on X specifically

### The Formula

Suppose we have:
```
Z → X → Y
  ↘___↗
```

Z is a parent of both X and Y (a confounder).

**Computing P(Y | do(X = x)):**

Step 1: Graph surgery - cut Z → X:
```
Z    X → Y
  ↘___↗
```

Step 2: Compute probability in mutilated graph:
```
P(Y | do(X = x)) = ∑_z P(Y | X = x, Z = z) · P(Z = z)
```

### Why We Average Over Z

Even though we cut Z → X, the variable Z:
- Still exists
- Still has its natural distribution P(Z)
- Still affects Y directly

Since Z is random in the population, we **average over all possible values of Z** weighted by their natural probabilities.

### The Crucial Difference from Conditioning

**Interventional:**
```
P(Y | do(X = x)) = ∑_z P(Y | X = x, Z = z) · P(Z)
```

**Observational:**
```
P(Y | X = x) = ∑_z P(Y | X = x, Z = z) · P(Z | X = x)
```

See the difference? 
- Interventional uses **P(Z)** - unconditional distribution
- Observational uses **P(Z | X = x)** - conditioned on X

The conditioning term P(Z | X = x) encodes the fact that Z → X exists in the observational world. In the interventional world, we broke that link, so we use P(Z).

### What's Constant vs. What's Random

**Important clarification:**

When we do(X = x):
- **X is made constant** (fixed at value x)
- **Parents of X remain random** (still vary according to P(Parents))
- **We average over parents because they're still random and still influence Y**

### Concrete Picture

Imagine 1000 people:
- Natural ability Z varies: 500 high ability, 500 low ability
- **Without intervention**: High ability → more likely to get education
- **With do(Education = college)**: Force ALL 1000 to get college
  - Z still varies (500 high, 500 low)
  - But Z no longer determines education
  - Both high and low ability people get college

When computing P(Income | do(Education = college)):
- Education is constant (everyone has college)
- Ability Z is NOT constant (still 50/50 split)
- We average: 50% of outcomes come from high-ability people, 50% from low-ability

---

## Causal Influence vs. Conditioning

There's a subtle connection between "cutting causal influence" and statistical conditioning.

### The Distinction

**Causal influence** (edges in graph): Z → X means Z causally determines X through some mechanism.

**Statistical dependence** (conditioning): P(X | Z) ≠ P(X) means knowing Z tells you something about X.

These often go together but are conceptually different!

### When We Cut Causal Influence

When we cut Z → X in do(X = x):
- We remove the causal mechanism by which Z determines X
- We make X statistically independent of Z in the new distribution

In the intervened distribution:
- X is always equal to x (deterministic)
- Therefore X ⊥ Z under the intervention
- P(X | Z, do(x)) is the same for all values of Z

### The Connection

In the **observational world**, the causal arrow Z → X creates statistical dependence expressible through conditioning:

P(X | Z) captures how Z causally influences X

When we **cut** that arrow with do(X = x), we're essentially:
- Ignoring P(X | Z)
- Replacing it with X := x

So cutting causal influence is like saying "stop conditioning X on Z."

### In Terms of SCMs

**Before intervention:**
```
X = f_X(Z, U_X)  // X depends on Z
```

**After do(X = x):**
```
X = x  // X does NOT depend on Z
```

### In Terms of Joint Distributions

**Observational:**
```
P(Z, X, Y) = P(Z) · P(X | Z) · P(Y | X, Z)
```

**Interventional:**
```
P(Z, X, Y | do(X=x)) = P(Z) · δ(X=x) · P(Y | X=x, Z)
```

where δ(X=x) is 1 if X=x, 0 otherwise.

Notice: P(X | Z) disappears, replaced by the intervention value.

### But Be Careful!

While cutting causal influence removes P(X | Z), we still need to **condition on Z when computing P(Y | ...)** if Z also influences Y!

That's why:
```
P(Y | do(X = x)) = ∑_z P(Y | X = x, Z = z) · P(Z)
```

We removed P(X | Z), but kept P(Y | X, Z) because Z → Y still exists.

---

## Summary

The do operator is a powerful tool for reasoning about causation:

1. **P(Y | do(X = x))** asks what would happen if we intervened to set X = x
2. This is fundamentally different from **P(Y | X = x)**, which conditions on observing X = x
3. The do operator performs **graph surgery**: cutting incoming edges to X while preserving outgoing edges
4. We still **average over parent variables** because they remain random and may influence Y through other paths
5. The key difference: interventional distributions use P(Parents), not P(Parents | X), breaking the selection bias
6. This formalizes what randomized controlled trials do: break confounding by making treatment assignment independent of confounders

The do operator bridges the gap between observational data and causal claims, making it possible (under certain conditions) to estimate causal effects from purely observational data.