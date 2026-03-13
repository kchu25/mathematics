@def title = "Non-Cooperative Game Theory: From Scratch to Nash Equilibrium"
@def published = "27 January 2026"
@def tags = ["game-theory"]

# Non-Cooperative Game Theory: From Scratch to Nash Equilibrium

\toc

## What is a Game?

A **finite game in strategic (normal) form** is a tuple $\Gamma = (N, \{S_i\}_{i \in N}, \{u_i\}_{i \in N})$ where:

- $N = \{1, 2, \ldots, n\}$ is the set of **players**
- $S_i$ is a finite set of **strategies** (or **actions**) available to player $i$
- $u_i : S_1 \times S_2 \times \cdots \times S_n \to \mathbb{R}$ is the **payoff function** (or utility function) for player $i$

A **strategy profile** is a tuple $s = (s_1, s_2, \ldots, s_n) \in S := S_1 \times S_2 \times \cdots \times S_n$.

We write $s_{-i} = (s_1, \ldots, s_{i-1}, s_{i+1}, \ldots, s_n)$ for "everyone's strategy except player $i$'s," so a profile can be written $(s_i, s_{-i})$.

---

> **Cooperative vs. Non-Cooperative**
>
> If you already know cooperative game theory (Shapley values, Banzhaf indices, the core, etc.), the key difference is:
>
> - **Cooperative**: Players can form binding agreements. The main question is *how to divide the payoff* among a coalition.
> - **Non-cooperative**: No binding agreements. Each player independently picks a strategy to maximize their own payoff. The main question is *what will each player actually do?*
>
> Nash equilibrium answers that second question.

## Two-Player Games and Payoff Matrices

The simplest interesting case: two players, each with finitely many strategies. We can represent the payoffs as a **bimatrix**.

If Player 1 has strategies $\{R_1, \ldots, R_m\}$ and Player 2 has $\{C_1, \ldots, C_k\}$, we write:

$$\begin{array}{c|ccc}
 & C_1 & C_2 & \cdots & C_k \\
\hline
R_1 & (a_{11}, b_{11}) & (a_{12}, b_{12}) & \cdots & (a_{1k}, b_{1k}) \\
R_2 & (a_{21}, b_{21}) & (a_{22}, b_{22}) & \cdots & (a_{2k}, b_{2k}) \\
\vdots & \vdots & \vdots & \ddots & \vdots \\
R_m & (a_{m1}, b_{m1}) & (a_{m2}, b_{m2}) & \cdots & (a_{mk}, b_{mk}) \\
\end{array}$$

where $(a_{ij}, b_{ij})$ means: Player 1 gets $a_{ij}$, Player 2 gets $b_{ij}$.

**Example: Prisoner's Dilemma.**

$$\begin{array}{c|cc}
 & \text{Cooperate} & \text{Defect} \\
\hline
\text{Cooperate} & (-1, -1) & (-3, 0) \\
\text{Defect} & (0, -3) & (-2, -2) \\
\end{array}$$

Each player independently decides to cooperate or defect. Higher payoff is better (so $0 > -1 > -2 > -3$). We'll come back to this example shortly.

## Dominant Strategies

Before getting to Nash equilibrium, let's start with the simplest solution concept.

**Definition.** Strategy $s_i \in S_i$ is **strictly dominated** by strategy $s_i' \in S_i$ if

$$u_i(s_i', s_{-i}) > u_i(s_i, s_{-i}) \quad \text{for every } s_{-i} \in S_{-i}.$$

In words: no matter what everyone else does, $s_i'$ always gives a strictly higher payoff than $s_i$. If a strategy is strictly dominated, a rational player would never play it.

**Definition.** Strategy $s_i^*$ is a **strictly dominant strategy** for player $i$ if it strictly dominates every other strategy in $S_i$.

**Example: Prisoner's Dilemma (continued).**

For Player 1:
- If Player 2 cooperates: Defect gives $0 > -1$ (Cooperate). ✓
- If Player 2 defects: Defect gives $-2 > -3$ (Cooperate). ✓

So Defect strictly dominates Cooperate for Player 1. By symmetry, the same holds for Player 2. Both players have a strictly dominant strategy: Defect.

The outcome $(\text{Defect}, \text{Defect})$ with payoff $(-2, -2)$ is the unique prediction—even though $(\text{Cooperate}, \text{Cooperate})$ gives $(-1, -1)$, which is better for both. This tension between individual rationality and collective welfare is the whole point of the Prisoner's Dilemma.

> **Dominant strategy equilibria are rare.** Most games don't have them, which is why we need a more general concept.

## Iterated Elimination of Strictly Dominated Strategies (IESDS)

Even when no player has a dominant strategy, we can still make progress by iteratively removing strategies that no rational player would use.

**The procedure:**
1. Remove all strictly dominated strategies for all players.
2. In the reduced game, check again for strictly dominated strategies.
3. Repeat until no more strategies can be eliminated.

**Example.** Consider:

$$\begin{array}{c|ccc}
 & L & M & R \\
\hline
U & (1, 0) & (1, 2) & (0, 1) \\
D & (0, 3) & (0, 1) & (2, 0) \\
\end{array}$$

**Round 1:** For Player 1, neither $U$ nor $D$ is strictly dominated (check: $U$ gives $(1,1,0)$ across columns while $D$ gives $(0,0,2)$—neither is uniformly better). For Player 2, compare $R$ against $M$: $M$ gives $(2,1)$ across rows while $R$ gives $(1,0)$. So $M$ strictly dominates $R$. Eliminate $R$.

**Reduced game:**

$$\begin{array}{c|cc}
 & L & M \\
\hline
U & (1, 0) & (1, 2) \\
D & (0, 3) & (0, 1) \\
\end{array}$$

**Round 2:** Now for Player 1: $U$ gives $(1, 1)$ while $D$ gives $(0, 0)$. So $U$ strictly dominates $D$. Eliminate $D$.

**Round 3:** We're left with Player 1 playing $U$. Player 2 chooses between $L$ (payoff 0) and $M$ (payoff 2). Player 2 picks $M$.

**Result:** $(U, M)$ with payoff $(1, 2)$.

**Key fact:** IESDS is **order-independent**—you get the same surviving strategies no matter which order you eliminate. (This is a nice exercise to verify; the proof relies on the fact that if $s_i$ is dominated in the original game, it remains dominated in any reduced game.)

**Limitation:** IESDS often doesn't pin down a unique outcome. When it does, we say the game is **dominance solvable**.


## Best Response

Here's the concept that directly leads to Nash equilibrium.

**Definition.** Strategy $s_i^*$ is a **best response** to $s_{-i}$ if

$$u_i(s_i^*, s_{-i}) \geq u_i(s_i, s_{-i}) \quad \text{for all } s_i \in S_i.$$

We write the **best response correspondence** as

$$BR_i(s_{-i}) = \arg\max_{s_i \in S_i} \; u_i(s_i, s_{-i}).$$

Note: $BR_i(s_{-i})$ is a *set*—there may be multiple best responses.

**Example.** In the Prisoner's Dilemma:

- $BR_1(\text{Cooperate}) = \{\text{Defect}\}$ (since $0 > -1$)
- $BR_1(\text{Defect}) = \{\text{Defect}\}$ (since $-2 > -3$)

Defect is a best response to *everything*—which is just another way to see it's a dominant strategy.

**A more interesting example (Coordination Game):**

$$\begin{array}{c|cc}
 & A & B \\
\hline
A & (2, 2) & (0, 0) \\
B & (0, 0) & (1, 1) \\
\end{array}$$

- $BR_1(A) = \{A\}$, $BR_1(B) = \{B\}$
- $BR_2(A) = \{A\}$, $BR_2(B) = \{B\}$

No dominant strategy here. But notice: at $(A, A)$, each player is best-responding to the other. Same for $(B, B)$.

This is exactly Nash equilibrium.

## Nash Equilibrium (Pure Strategy)

**Definition.** A strategy profile $s^* = (s_1^*, s_2^*, \ldots, s_n^*)$ is a **Nash equilibrium** if for every player $i \in N$:

$$u_i(s_i^*, s_{-i}^*) \geq u_i(s_i, s_{-i}^*) \quad \text{for all } s_i \in S_i.$$

Equivalently: $s_i^* \in BR_i(s_{-i}^*)$ for every $i$.

**In words:** No player can improve their payoff by unilaterally changing their strategy, holding everyone else's strategy fixed. It's a mutual best response.

**Important:** Nash equilibrium says nothing about groups deviating together. Only individual deviations are considered. It also says nothing about whether the outcome is "good" (Prisoner's Dilemma has a Nash equilibrium that's Pareto dominated).

**To find all pure-strategy Nash equilibria in a bimatrix game:**

For each cell $(R_j, C_k)$:
1. Check if $R_j$ is a best response for Player 1 given $C_k$ (is $a_{jk} \geq a_{ik}$ for all $i$?).
2. Check if $C_k$ is a best response for Player 2 given $R_j$ (is $b_{jk} \geq b_{jl}$ for all $l$?).
3. If both, it's a Nash equilibrium.

**Underline method (quick trick):** In each column, underline the largest $a_{jk}$ (Player 1's best response). In each row, underline the largest $b_{jk}$ (Player 2's best response). Any cell with *both* entries underlined is a Nash equilibrium.

### Examples

**Prisoner's Dilemma:** Only $(\text{Defect}, \text{Defect})$. (Unique Nash equilibrium.)

**Coordination Game:** Both $(A, A)$ and $(B, B)$. (Multiple Nash equilibria—the theory alone doesn't tell you which one players will coordinate on.)

**Matching Pennies:**

$$\begin{array}{c|cc}
 & H & T \\
\hline
H & (1, -1) & (-1, 1) \\
T & (-1, 1) & (1, -1) \\
\end{array}$$

Check every cell:
- $(H, H)$: Player 2 would rather switch to $T$. ✗
- $(H, T)$: Player 1 would rather switch to $T$. ✗
- $(T, H)$: Player 1 would rather switch to $H$. ✗
- $(T, T)$: Player 2 would rather switch to $H$. ✗

**No pure-strategy Nash equilibrium exists!** This motivates the next section.

## Mixed Strategies

**Definition.** A **mixed strategy** for player $i$ is a probability distribution over $S_i$. If $S_i = \{s_i^1, \ldots, s_i^{m_i}\}$, a mixed strategy is

$$\sigma_i = (\sigma_i(s_i^1), \ldots, \sigma_i(s_i^{m_i})) \in \Delta(S_i)$$

where $\sigma_i(s_i^j) \geq 0$ for all $j$ and $\sum_j \sigma_i(s_i^j) = 1$.

Here $\Delta(S_i)$ is the **simplex** over $S_i$. A pure strategy is a special case where all probability mass is on one action.

**Expected payoff** under a mixed strategy profile $\sigma = (\sigma_1, \ldots, \sigma_n)$:

$$u_i(\sigma) = \sum_{s \in S} \left(\prod_{j=1}^{n} \sigma_j(s_j)\right) u_i(s)$$

where the product reflects independence—each player randomizes independently.

### The Indifference Principle

This is the key insight for computing mixed-strategy Nash equilibria.

**Proposition.** If $\sigma^*$ is a mixed-strategy Nash equilibrium and player $i$ plays actions $s_i^a$ and $s_i^b$ with positive probability under $\sigma_i^*$, then

$$u_i(s_i^a, \sigma_{-i}^*) = u_i(s_i^b, \sigma_{-i}^*).$$

**Proof.** Suppose not—say $u_i(s_i^a, \sigma_{-i}^*) > u_i(s_i^b, \sigma_{-i}^*)$. Then player $i$ could shift all probability from $s_i^b$ to $s_i^a$, strictly increasing expected payoff. That contradicts $\sigma^*$ being a Nash equilibrium.

In words: **in equilibrium, a player must be indifferent among all actions they play with positive probability.** Otherwise they'd concentrate on the better one.

This gives us a system of equations to solve.

### Example: Matching Pennies

Let Player 1 play $H$ with probability $p$ (and $T$ with $1 - p$). Let Player 2 play $H$ with probability $q$ (and $T$ with $1 - q$).

For Player 1 to be willing to mix, Player 2 must make Player 1 indifferent:

$$u_1(H, \sigma_2) = u_1(T, \sigma_2)$$
$$q \cdot 1 + (1-q) \cdot (-1) = q \cdot (-1) + (1-q) \cdot 1$$
$$2q - 1 = 1 - 2q$$
$$q = \frac{1}{2}$$

By symmetry, $p = \frac{1}{2}$.

The unique mixed-strategy Nash equilibrium is: both players randomize uniformly. Expected payoff is $0$ for each player.

**Sanity check:** If Player 2 plays $q = 1/2$, then Player 1's expected payoff from $H$ is $\frac{1}{2}(1) + \frac{1}{2}(-1) = 0$, and from $T$ is $\frac{1}{2}(-1) + \frac{1}{2}(1) = 0$. Indeed indifferent. ✓

### Example: Coordination Game (Mixed Equilibrium)

$$\begin{array}{c|cc}
 & A & B \\
\hline
A & (2, 2) & (0, 0) \\
B & (0, 0) & (1, 1) \\
\end{array}$$

We already found two pure Nash equilibria: $(A, A)$ and $(B, B)$. Let's find the mixed one.

Let Player 1 play $A$ with probability $p$, Player 2 play $A$ with probability $q$.

Player 1 indifferent requires:
$$u_1(A, \sigma_2) = u_1(B, \sigma_2)$$
$$2q + 0(1-q) = 0 \cdot q + 1 \cdot (1-q)$$
$$2q = 1 - q$$
$$q = \frac{1}{3}$$

By symmetry, $p = 1/3$.

The mixed Nash equilibrium is $(p, q) = (1/3, 1/3)$. The expected payoff for each player is $2/3$.

Notice: both pure equilibria give higher payoffs ($2$ or $1$) than the mixed one ($2/3$). This is typical—mixed equilibria often have lower expected payoffs because players are deliberately randomizing, sometimes landing on miscoordination.

This game has **three** Nash equilibria total: $(A,A)$, $(B,B)$, and the mixed equilibrium.

## Nash's Theorem

The big result.

**Theorem (Nash 1950).** Every finite game (finite number of players, each with finitely many strategies) has at least one Nash equilibrium in mixed strategies.

This is why mixed strategies are so important—they guarantee existence.

### Proof Sketch

The proof uses **Brouwer's fixed point theorem** (or, equivalently for the multi-player case, **Kakutani's fixed point theorem**). Here is the key idea.

**Step 1: Set up the domain.** The set of mixed strategy profiles is

$$\Sigma = \Delta(S_1) \times \Delta(S_2) \times \cdots \times \Delta(S_n).$$

Each $\Delta(S_i)$ is a simplex (compact and convex), so $\Sigma$ is compact and convex.

**Step 2: Define the best response correspondence.** For each player $i$, define

$$BR_i(\sigma_{-i}) = \arg\max_{\sigma_i \in \Delta(S_i)} \; u_i(\sigma_i, \sigma_{-i}).$$

Since $u_i$ is linear in $\sigma_i$ (it's an expected value) and $\Delta(S_i)$ is compact, the maximum is attained. Moreover, $BR_i(\sigma_{-i})$ is convex: if $\sigma_i'$ and $\sigma_i''$ are both best responses, then so is any convex combination (because $u_i$ is linear in $\sigma_i$, so the expected payoff of the mixture equals the common maximum value).

**Step 3: Apply Kakutani's fixed point theorem.** Define the combined best response correspondence

$$BR(\sigma) = BR_1(\sigma_{-1}) \times BR_2(\sigma_{-2}) \times \cdots \times BR_n(\sigma_{-n}).$$

This is a correspondence from $\Sigma$ to $\Sigma$ (a set-valued map). Kakutani's theorem says: if $\Sigma$ is compact and convex, and $BR$ is upper hemicontinuous with non-empty convex values, then there exists a fixed point $\sigma^*$ such that

$$\sigma^* \in BR(\sigma^*).$$

We already argued that $BR_i$ has non-empty convex values. Upper hemicontinuity follows from the continuity of $u_i$ and the maximum theorem.

**Step 4: Fixed point = Nash equilibrium.** A fixed point $\sigma^* \in BR(\sigma^*)$ means $\sigma_i^* \in BR_i(\sigma_{-i}^*)$ for every $i$—which is exactly the definition of Nash equilibrium.  $\blacksquare$

> **Kakutani's fixed point theorem** (1941) generalizes Brouwer's theorem to set-valued (correspondence) maps. The conditions are:
> 1. The domain is compact and convex (here: $\Sigma$) ✓
> 2. The correspondence is upper hemicontinuous ✓
> 3. Values are non-empty and convex ✓
>
> If you know Brouwer's theorem, the extension to correspondences is the main technical step. Nash's original proof actually used Brouwer directly by constructing a continuous function whose fixed points correspond to equilibria—Kakutani just makes the argument cleaner.

## Properties and Limitations

### What Nash Equilibrium Gives You

- **Existence guarantee**: Every finite game has at least one (in mixed strategies).
- **Self-enforcing**: If everyone plays their part of a Nash equilibrium, no one has incentive to deviate.
- **Mutual consistency**: Each player's strategy is optimal given their (correct) beliefs about others.

### What It Doesn't Give You

- **Uniqueness**: Many games have multiple Nash equilibria, and the theory alone doesn't select among them (this spawned an entire literature on **equilibrium refinements**: subgame perfection, trembling hand perfection, etc.).
- **Efficiency**: Nash equilibria can be Pareto inefficient (Prisoner's Dilemma).
- **Group deviations**: Only unilateral deviations are considered. If players can coordinate deviations, you need stronger concepts (strong Nash, coalition-proof, etc.).
- **How players get there**: Nash equilibrium is a *static* concept. It says what a stable outcome looks like, not how players learn or converge to it.

### Relationship to Dominant Strategies

$$\text{Strictly dominant strategy equilibrium} \subseteq \text{Survives IESDS} \subseteq \text{Nash equilibrium (pure)} \subseteq \text{Nash equilibrium (mixed)}$$

More precisely:
- If a game has a strictly dominant strategy equilibrium, it's the unique Nash equilibrium.
- Every pure Nash equilibrium survives IESDS.
- If IESDS leaves a unique strategy profile, that profile is the unique Nash equilibrium.

These inclusions are strict in general.

## Computing Nash Equilibria: The Support Enumeration Method

For two-player games, here's a systematic approach.

**Idea:** Enumerate all possible **supports** (the set of pure strategies played with positive probability) for each player. For each pair of supports, use the indifference principle to solve for the mixing probabilities.

**For a 2-player game with $|S_1| = m$ and $|S_2| = k$:**

1. For each non-empty $T_1 \subseteq S_1$ and $T_2 \subseteq S_2$:
   - Set up the indifference equations: Player 1 must be indifferent over all actions in $T_2$'s support (that's $|T_1| - 1$ equations from Player 2's indifference, plus the probability constraint).
   - Wait—let me state this correctly. Player 2's mixing must make Player 1 indifferent among the actions in $T_1$. Player 1's mixing must make Player 2 indifferent among actions in $T_2$.
   - Solve the resulting linear system.
   - Check that all probabilities are in $[0, 1]$ and sum to 1.
   - Check that strategies *outside* the support are not profitable deviations (i.e., actions in $S_1 \setminus T_1$ give Player 1 payoff $\leq$ what they get in equilibrium, and similarly for Player 2).

2. Collect all valid solutions.

The worst case is exponential in the number of strategies (you check $2^m \cdot 2^k$ support pairs), but for small games this is perfectly tractable.

> **Computational complexity.** Computing Nash equilibria is hard in general. For two-player games, finding *one* Nash equilibrium is in the complexity class **PPAD** (Polynomial Parity Arguments on Directed graphs). It is PPAD-complete even for two-player games, which means there's unlikely to be a polynomial-time algorithm—though the problem is probably not NP-hard either. This was shown by Daskalakis, Goldberg, and Papadimitriou (2009) and Chen, Deng, and Teng (2009). For games with more than two players, things get even harder.

## Summary

| Concept | What it tells you |
|---|---|
| Dominant strategy | "Always do this, no matter what" |
| IESDS | "A rational player would never do *that*" |
| Best response | "Given what others do, here's my best move" |
| Nash equilibrium (pure) | "Nobody wants to change—we're all best-responding to each other" |
| Nash equilibrium (mixed) | "Same as above, but players may randomize; always exists" |

The logical flow:

$$\text{Dominant strategy} \xrightarrow{\text{too restrictive}} \text{IESDS} \xrightarrow{\text{still incomplete}} \text{Best response} \xrightarrow{\text{mutual}} \text{Nash equilibrium} \xrightarrow{\text{existence via}} \text{Mixed strategies}$$

Nash equilibrium is the foundational solution concept of non-cooperative game theory. Everything that follows in the field—subgame perfection, Bayesian games, signaling games, mechanism design—builds on this foundation by either strengthening, extending, or applying it to richer settings.
