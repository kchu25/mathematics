@def title = "When Efficiency is Given *Before* Banzhaf"
@def published = "26 January 2026"
@def tags = ["game-theory"]

# When Efficiency is Given *Before* Banzhaf

## The Usual Story

Normally, we derive power indices from axioms:
- **Shapley**: Starts from axioms (symmetry, dummy player, additivity, **efficiency**) → derives unique formula
- **Banzhaf**: Starts from different axioms (no efficiency!) → derives formula that typically violates efficiency

## Your Reversed Scenario

You're asking: what if we **start** with efficiency as a constraint on the game itself, *then* compute Banzhaf?

This is actually a really interesting theoretical flip!

### What Changes

**Standard approach:**
$$\text{Axioms} \rightarrow \text{Power Index} \rightarrow \text{Check if } \sum_i \phi_i(v) = v(N)$$

**Your scenario:**
$$v(N) = \sum_i \beta_i(v) \text{ (given)} \rightarrow \text{Compute Banzhaf normally}$$

### Theoretical Interpretation

Here's what's happening: you've discovered that the **game** $v$ has a special property—its Banzhaf values happen to be efficient. This tells us something deep about the game's structure.

#### The Math Behind It

The Banzhaf index for player $i$ is:
$\beta_i(v) = \frac{1}{2^{n-1}} \sum_{S \subseteq N \setminus \{i\}} [v(S \cup \{i\}) - v(S)]$

**What this says:** For each player $i$, look at all coalitions that don't include them (there are $2^{n-1}$ such coalitions). For each coalition $S$, check how much value player $i$ adds when joining: $v(S \cup \{i\}) - v(S)$. Sum all these marginal contributions, then normalize by dividing by $2^{n-1}$.

---

For efficiency to hold:
$\sum_{i=1}^{n} \beta_i(v) = v(N)$

**Translation:** All players' Banzhaf values must add up to the total payoff of the grand coalition.

---

Substituting the formula for $\beta_i(v)$:
$\sum_{i=1}^{n} \left[\frac{1}{2^{n-1}} \sum_{S \subseteq N \setminus \{i\}} [v(S \cup \{i\}) - v(S)]\right] = v(N)$

Multiply both sides by $2^{n-1}$:
$\sum_{i=1}^{n} \sum_{S \subseteq N \setminus \{i\}} [v(S \cup \{i\}) - v(S)] = 2^{n-1} \cdot v(N)$

**What this means:** The left side counts every marginal contribution, summed over all players. The right side says this total must equal $2^{n-1}$ times the grand coalition's value.

---

**Why is this interesting?** Let's think about what the left side is actually counting.

For each coalition $T$ (that's not empty or the full set $N$), the difference $v(T) - v(T \setminus \{i\})$ appears in the sum exactly once for each player $i \in T$. Similarly, each coalition appears multiple times in different forms as we vary which player we're considering.

The equation is saying: if you sum up all these marginal contributions (counted in the Banzhaf way), you should get exactly $2^{n-1} \cdot v(N)$.

**The constraint:** This is a very specific arithmetic condition on the game's characteristic function $v$. Most games don't satisfy this! When a game does satisfy it, it means the game has a special combinatorial structure where marginal contributions are "balanced" in a particular way.

---

**What does this property imply?**

When a game satisfies the efficiency condition:
$\sum_{i=1}^{n} \sum_{S \subseteq N \setminus \{i\}} [v(S \cup \{i\}) - v(S)] = 2^{n-1} \cdot v(N)$

Several interesting things follow:

1. **Symmetry in marginal contributions**: The left side sums all marginal contributions $v(S \cup \{i\}) - v(S)$ across all players and coalitions. For this to equal exactly $2^{n-1} \cdot v(N)$, these marginal contributions must be "balanced"—no single coalition size or structure can dominate. If value were concentrated in just a few key coalitions, the sum would be skewed and wouldn't hit this precise target.

2. **Banzhaf becomes a valid payoff division**: Remember that $\beta_i(v) = \frac{1}{2^{n-1}} \sum_{S \subseteq N \setminus \{i\}} [v(S \cup \{i\}) - v(S)]$. When the efficiency condition holds, we have:
   $\sum_{i=1}^{n} \beta_i(v) = \frac{1}{2^{n-1}} \cdot 2^{n-1} \cdot v(N) = v(N)$
   
   So the Banzhaf values actually partition the total payoff! Normally you can't use Banzhaf to allocate $v(N)$ because $\sum_i \beta_i(v) \neq v(N)$, but this game is special.

3. **Alignment of power and value**: The term $v(S \cup \{i\}) - v(S)$ measures how pivotal player $i$ is to coalition $S$. The efficiency condition says that when you sum these pivotality measures (weighted equally across all coalitions) and normalize, you get exactly the total value. This means "how often you're pivotal" genuinely translates to "how much you deserve" for this game.

4. **Structural hint about the game**: For the equation $\sum_{i} \sum_{S \subseteq N \setminus \{i\}} [v(S \cup \{i\}) - v(S)] = 2^{n-1} \cdot v(N)$ to hold, the game often exhibits:
   - **Symmetric games**: If all players are identical, each $\beta_i = \frac{v(N)}{n}$, and the equation simplifies nicely
   - **Balanced weighted voting**: The quota and weights create evenly distributed marginal contributions
   - **Specific superadditivity**: Value grows in a way that marginal contributions across coalition sizes balance out

5. **Computational convenience**: Since $\sum_{i=1}^{n} \beta_i(v) = v(N)$, you can compute $\beta_1, \ldots, \beta_{n-1}$ and immediately know:
   $\beta_n(v) = v(N) - \sum_{i=1}^{n-1} \beta_i(v)$
   
   This saves you from computing the last player's marginal contributions entirely.

**Bottom line:** The equation $\sum_{i} \sum_{S} [v(S \cup \{i\}) - v(S)] = 2^{n-1} \cdot v(N)$ is saying that the total "pivotality mass" in the game equals exactly $2^{n-1}$ times the grand coalition value. This is a delicate balance that tells you the game's structure is harmonious with Banzhaf's equal-weight-per-coalition counting method.

---

> **Efficiency Bridges "Pivotal" to "Deserving"**
>
> Here's the key shift: without efficiency, Banzhaf just measures **power**—how often you're the swing vote. It's a descriptive measure of influence, not a prescriptive division of payoffs.
>
> But when efficiency holds ($\sum_i \beta_i(v) = v(N)$), something changes. Now your Banzhaf value isn't just telling you how powerful you are—it's telling you your **fair share** of $v(N)$.
>
> Why? Because the numbers add up! If we use Banzhaf to divide the payoff, we distribute exactly $v(N)$, no more, no less. Suddenly "being pivotal 40% of the time" translates directly to "you deserve 40% of the total value."
>
> Without efficiency, there's a gap: you might be pivotal 40% of the time, but the sum of everyone's Banzhaf values is only 0.85·v(N). So what does your 40% even mean for payoff division? Nothing coherent.
>
> With efficiency, that gap closes. Pivotality becomes not just a measure of power, but a **justified claim** on the payoff. The game's structure makes your swing power directly convertible to economic value.

### What Does This Constraint Mean?

When you impose efficiency *first*, you're saying: "I only care about games where the marginal contribution accounting (Banzhaf-style) happens to distribute everything perfectly."

This is a **restriction on the class of games** you're considering, not a change to Banzhaf itself.

#### Practical Implications

1. **Convergence of interpretations**: For these special games, Banzhaf's "swing power" interpretation and Shapley's "fair distribution" interpretation align in total (though not necessarily in individual allocations)

2. **Structural symmetry**: The game likely has some underlying symmetry or balance that makes different coalition orderings equally weighted—which is rare!

3. **Theoretical comfort**: You get the appealing property that all power "accounts for" the total, making the index more interpretable as a division of $v(N)$

### Does It Matter?

**Philosophically**: Yes! It suggests that for this game, the distinction between "power to swing outcomes" (Banzhaf) and "expected marginal contribution" (Shapley) collapses at the aggregate level.

> **Beyond Games: What Does Efficiency Mean in Other Contexts?**
>
> The game-theoretic interpretation (swing power vs. resource allocation) is clear, but what about when we use Shapley/Banzhaf values in machine learning or biology?
>
> **Neural networks / Gene expression prediction:**
> - **Banzhaf (swing)**: How often does feature $i$ flip the prediction? If you toggle gene $i$ on/off across different contexts, how often does the expression level cross a threshold?
> - **Shapley (contribution)**: What's the average contribution of gene $i$ to the prediction, weighted across all possible orderings of features?
> - **When efficiency holds**: The total "flipping power" across all features equals the total "contribution power." This means the model's prediction is **fully accountable**—every bit of the output can be traced to some feature's pivotal role.
>
> **Resource allocation interpretation:**
> Without efficiency, Banzhaf values don't tell you how to allocate resources (compute, attention, experimental effort) because they don't sum to 100% of the "budget."
>
> But when efficiency holds:
> - If gene $i$ has Banzhaf value $\beta_i = 0.3 \cdot v(N)$, you can justify allocating 30% of your experimental resources to studying that gene
> - If feature $i$ has $\beta_i = 0.15 \cdot v(N)$, you can justify 15% of compute budget for computing that feature
>
> **The key insight**: Efficiency transforms Banzhaf from a **diagnostic tool** ("this feature is important") into a **prescriptive tool** ("allocate resources proportionally to these values"). Without efficiency, you're just ranking importance. With it, you're actually dividing a budget.
>
> For gene expression: efficiency means the importance scores (whether swing-based or contribution-based) genuinely partition the total predictive power, so they can guide where to invest limited experimental resources.

**Computationally**: No. You still compute Banzhaf the same way.

**Practically**: Maybe. If you're using Banzhaf for payoff allocation (which is controversial since it lacks efficiency), finding a game where efficiency holds gives you post-hoc justification for this use.

### The Deep Question

You've essentially asked: "What if the game cooperates with Banzhaf's deficiency?"

The answer is that you've found a **compatible game-index pair**—a game whose structure happens to respect a property that the index doesn't generally guarantee. It's like finding a function that's continuous at a point even though it's not continuous everywhere.

This doesn't make Banzhaf satisfy efficiency *as an axiom*, but it does mean that for this game, the efficiency *consequence* emerges naturally from the game's marginal contribution structure.

---

> **On the Implication Direction**
>
> Exactly—you've got the asymmetry right!
>
> **The usual direction (Shapley):**
> Efficiency axiom → Shapley formula → Efficiency always holds ✓
>
> **The reverse (Banzhaf):**
> Banzhaf formula → Efficiency *sometimes* holds (for special games)
>
> But here's the key: **you can't go backwards with Banzhaf.** Observing that efficiency holds for one game doesn't let you conclude anything about the Banzhaf formula itself. The formula doesn't "imply" efficiency in general.
>
> Think of it like this: Shapley is constructed to guarantee efficiency (forward implication built-in), while Banzhaf just does its own thing and occasionally, by coincidence of the game's structure, the numbers happen to add up. 
>
> So when you impose efficiency as a constraint *before* computing Banzhaf, you're essentially filtering for "Banzhaf-friendly games"—games where the accident becomes a feature. But this doesn't change the fundamental fact that Banzhaf, as a method, doesn't generally preserve efficiency across all games.