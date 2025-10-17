@def title = "Probability Concepts in Graphical Models: Intuitive Guide"
@def published = "17 October 2025"
@def tags = ["causal-inference"]

# Probability Concepts in Graphical Models: Intuitive Guide

## 1. Conditional Independence

**What it means:** X and Y don't give you information about each other once you know Z. It's like two people who only talk through a mutual friend - if you already know what the friend said, talking to both people tells you nothing new.

**Formally:** $P(X, Y | Z) = P(X | Z) \cdot P(Y | Z)$

Or equivalently: $P(X | Y, Z) = P(X | Z)$



**Example:** Your alarm (X) and your neighbor's alarm (Y) both depend on an earthquake (Z). If you know whether there was an earthquake, knowing your alarm went off tells you nothing new about whether your neighbor's alarm went off.

---

## 2. d-Separation (Directional Separation)

**What it means:** A graphical way to read off conditional independencies from a causal diagram. Think of it as checking whether information can "flow" between variables through the graph.

**The three structures:**
- **Chain** $X \to Z \to Y$: Conditioning on Z blocks the path
- **Fork** $X \gets Z \to Y$: Conditioning on Z blocks the path  
- **Collider** $X \to Z \gets Y$: Conditioning on Z *opens* the path (the weird one!)

**Intuition:** Information flows through chains and forks unless you condition on the middle. But colliders block information *until* you condition on them - then information starts flowing backward!

---

## 3. The Backdoor Criterion

**What it means:** A recipe for finding a set of variables to control for when estimating causal effects. You want to block all the "sneaky" paths from treatment to outcome that don't go through the treatment's actual causal effect.

**Formally:** Set Z satisfies the backdoor criterion if:
1. No variable in Z is a descendant of treatment X
2. Z blocks all paths from X to Y that have an arrow into X

**Intuition:** You're trying to close all the "back doors" - alternative explanations that make X and Y correlated without X actually causing Y. Think of blocking confounders while avoiding conditioning on consequences of the treatment.

---

## 4. Do-Calculus / Intervention

**What it means:** The difference between seeing ($P(Y|X)$) and doing ($P(Y|do(X))$). Seeing is passive observation; doing is active intervention where you force X to a value.

**Notation:** $P(Y | do(X = x))$

**Intuition:** If you see someone with an umbrella, you predict rain. But if you *give* someone an umbrella (intervention), that doesn't cause rain! The do-operator cuts the arrows pointing into X, representing that we override the natural causes.

**Example:** 
- $P(\text{recovery} | \text{took medicine})$ might be low because sicker people take medicine
- $P(\text{recovery} | do(\text{took medicine}))$ tells you the actual treatment effect

---

## 5. Confounding

**What it means:** When a third variable causes both your treatment and outcome, creating a spurious correlation.

**Visually:** $X \gets Z \to Y$

**Intuition:** Ice cream sales and drowning deaths are correlated, but ice cream doesn't cause drowning. Temperature (the confounder) causes both - hot weather means more swimming (drowning) and more ice cream.

**Why it matters:** $P(Y|X) \neq P(Y|do(X))$ when there's confounding. You need to adjust for Z to get the causal effect.

---

## 6. Collider Bias (Berkson's Paradox)

**What it means:** Conditioning on a common effect of two variables creates a spurious correlation between them, even when they're independent.

**Structure:** $X \to Z \gets Y$

**Intuition:** Among hospitalized patients, there's a negative correlation between having disease X and disease Y, even if they're unrelated in the general population. Why? If someone is hospitalized with mild disease X, they probably don't have disease Y (otherwise they'd be even sicker). Conditioning on hospitalization creates a false relationship.

**Another example:** Among famous people, talent and attractiveness seem negatively correlated. But that's because you can become famous through either route - conditioning on fame creates spurious correlation.

---

## 7. Markov Property

**What it means:** A variable is independent of its non-descendants given its parents. In other words, the parents shield you from needing to know the whole history.

**Formally:** $P(X | \text{parents}(X), \text{non-descendants}(X)) = P(X | \text{parents}(X))$

**Intuition:** To predict what happens at a node, you only need to know its immediate causes (parents), not the entire past. The causal structure captures all relevant information through the parent variables.

---

## 8. Faithfulness Assumption

**What it means:** The only independencies in your data are the ones implied by the causal structure - no weird coincidental cancellations.

**Counter-example:** Suppose $X \to Z \gets Y$ where $Z = X + Y$ normally creates dependence. But if the specific coefficients make $Z = X - X$, you get independence by coincidence. Faithfulness rules this out.

**Intuition:** Nature doesn't conspire to perfectly cancel out effects. If two things are dependent in the causal graph, they'll be dependent in the data (with enough samples).


---

## 9. The Front-Door Criterion

**What it means:** An alternative to the backdoor criterion when you can't measure confounders. You find a mediator that captures all of X's effect on Y.

**Structure:** Need $X \to M \to Y$ where:
- M blocks all directed paths from X to Y
- No backdoor paths from X to M  
- All backdoor paths from M to Y are blocked by X

**Intuition:** Even if smoking and lung cancer are confounded by an unmeasured gene, you can still estimate the effect by going through tar deposits (mediator). You break the problem into two smaller problems that are each identifiable.

---

## 10. Instrumental Variable

**What it means:** A variable that affects your treatment but only affects the outcome through the treatment - it gives you leverage to tease out the causal effect.

**Structure:** $Z \to X \to Y$ (with possible unmeasured $X \gets U \to Y$)

**Intuition:** Draft lottery number (Z) affects military service (X) which affects earnings (Y). Since lottery is random, it's not confounded with earnings. We can use variation in service caused by the lottery to estimate the effect of service.

**Requirements:**
1. Relevance: Z actually affects X  
2. Exclusion: Z affects Y only through X
3. Exogeneity: Z is independent of confounders

---

## 11. Exchangeability (No Unmeasured Confounding)

**What it means:** If you swapped treated and control groups, their outcomes would also swap. The groups are comparable except for the treatment.

**Formally:** $Y(1), Y(0) \perp X | Z$ 

Where $Y(1)$ is potential outcome under treatment, $Y(0)$ under control.

**Intuition:** In a perfect randomized trial, treatment assignment is exchangeable - you could swap who got treatment and who got control without changing anything systematic about the groups. In observational studies, we try to achieve conditional exchangeability by adjusting for confounders Z.

---

## 12. Positivity (Overlap)

**What it means:** Every combination of confounders must have some chance of receiving both treatment and control.

**Formally:** $0 < P(X=1|Z) < 1$ for all observed values of Z

**Intuition:** You can't estimate treatment effects for groups that never get treated or never get control. If all elderly patients always receive treatment, you have nothing to compare to - you can't know the counterfactual.

**Example:** If only women take a certain drug, you can't say what would happen if men took it using observational data alone.

---

## 13. Consistency (SUTVA - part 1)

**What it means:** The potential outcome under treatment equals the observed outcome when treatment is actually received. No "hidden versions" of treatment.

**Formally:** If $X = x$, then $Y = Y(x)$

**Intuition:** "Treatment" must be well-defined. If the "same" treatment can be delivered different ways with different effects, you can't reason about causal effects cleanly. Taking aspirin made by Bayer vs generic must have the same effect.

---

## 14. No Interference (SUTVA - part 2)

**What it means:** One person's treatment doesn't affect another person's outcome.

**Formally:** $Y_i(x_1, ..., x_n) = Y_i(x_i)$

**Intuition:** Your outcome from taking a vaccine depends only on whether *you* took it, not whether your neighbor did. This is violated for contagious diseases (spillover effects) or social networks. Also called "no spillover."

---

## 15. Selection Bias

**What it means:** Your sample isn't representative because selection into the sample depends on both treatment and outcome.

**Structure:** Conditioning on a collider of treatment and outcome (or their causes)

**Intuition:** A study of surgery effectiveness done only on surgical patients is biased - people select into surgery based on health status (which also affects outcomes). This creates collider bias.

**Example:** Studying hospital mortality rates unfairly penalizes hospitals that take sicker patients - conditioning on hospitalization creates selection bias.