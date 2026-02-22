@def title = "Modular Arithmetic Is Linear Algebra (Over a Finite Field)"
@def published = "21 February 2026"
@def tags = ["linear-algebra", "group-theory", "cryptography"]

# Modular Arithmetic Is Linear Algebra (Over a Finite Field)

*A companion to [The Math You Need for Zero-Knowledge Proofs](/blog/group_theory/math_for_zk/). That post builds the crypto story through groups and number theory. This post shows that almost everything there is secretly linear algebra — and the linear algebra view is often cleaner.*

---

## The punchline, up front

Every time you see modular arithmetic in cryptography, you can mentally substitute:

| Crypto / number theory language | Linear algebra language |
|---|---|
| Integers mod $p$ | The finite field $\mathbb{F}_p$ (a 1-dimensional vector space over itself) |
| Exponents mod $q$ (group order) | Vectors in $\mathbb{F}_q$ |
| $g^x \bmod p$ | A linear map $\mathbb{F}_q \to G$ (the "encoding" map) |
| $g^{x+y} = g^x \cdot g^y$ | Linearity: $\phi(x + y) = \phi(x) \cdot \phi(y)$ |
| Schnorr response $z = r + ex$ | A linear equation in two unknowns over $\mathbb{F}_q$ |
| Pedersen commitment $g^v h^r$ | A linear combination of two basis vectors |
| Soundness extraction | Solving a $2 \times 2$ linear system |
| Zero-knowledge | The solution space of an underdetermined system has a free variable |

Once you see it this way, the proofs almost write themselves.

---

## 1. The exponent space is a vector space

Fix a prime-order group $G = \langle g \rangle$ of order $q$ (for concreteness, think of a subgroup of $\mathbb{Z}_p^*$ with $p = 2q + 1$). Every element of $G$ can be written $g^x$ for a unique $x \in \{0, 1, \ldots, q-1\}$. The exponents live in $\mathbb{F}_q$ — the integers mod $q$, which form a **field** (you can add, subtract, multiply, and divide, since $q$ is prime).

A field is exactly what you need to do linear algebra. $\mathbb{F}_q$ plays the role that $\mathbb{R}$ plays in ordinary linear algebra. "Vectors" are elements of $\mathbb{F}_q^n$ for various $n$, and "linear maps" are maps that respect addition and scalar multiplication — over $\mathbb{F}_q$ instead of $\mathbb{R}$.

> **Key shift in perspective:** Stop thinking of $g^x$ as "a number raised to a power." Think of $x$ as a vector (in $\mathbb{F}_q$), and $g^{(\cdot)}$ as a linear encoding map that sends $x$ to a group element. The group $G$ is the "image space." The exponent $x$ is the object you're really doing algebra on.

---

## 2. The encoding map is linear

Define $\phi: \mathbb{F}_q \to G$ by $\phi(x) = g^x$. Then:

$$\phi(x + y) = g^{x+y} = g^x \cdot g^y = \phi(x) \cdot \phi(y)$$

This is a **group homomorphism**, but if you squint, it's the finite-field version of linearity. The operation on the left is addition in $\mathbb{F}_q$; the operation on the right is multiplication in $G$. The map $\phi$ translates between them faithfully.

Scalar multiplication works too. For $c \in \mathbb{F}_q$:

$$\phi(c \cdot x) = g^{cx} = (g^x)^c = \phi(x)^c$$

So $\phi$ respects both addition and scaling — it's a linear map from the $\mathbb{F}_q$-module $\mathbb{F}_q$ to the $\mathbb{F}_q$-module $G$ (where $G$'s "addition" is written multiplicatively and "scalar multiplication" is exponentiation).

> **Why this matters:** Every property of the Schnorr protocol, Pedersen commitments, and their security proofs can be stated and verified as facts about linear maps over $\mathbb{F}_q$. You don't need number theory intuition — linear algebra intuition works.

---

## 3. Schnorr verification is a linear equation

Recall the Schnorr protocol. The prover holds secret $x \in \mathbb{F}_q$, publishes $y = g^x$. The proof is:

1. **Commit:** pick random $r \in \mathbb{F}_q$, send $a = g^r$.
2. **Challenge:** verifier sends random $e \in \mathbb{F}_q$.
3. **Respond:** send $z = r + e \cdot x \in \mathbb{F}_q$.
4. **Verify:** check $g^z = a \cdot y^e$.

Now look at step 3. In $\mathbb{F}_q$:

$$z = r + e \cdot x$$

That's a **linear equation** in the unknowns $(r, x)$ with known coefficient $e$ and known result $z$. One equation, two unknowns. The linear algebra practically shouts the security properties at you.

---

## 4. Soundness = solving a determined linear system

Suppose a cheating prover can answer **two** different challenges $e_1, e_2$ for the same commitment $a = g^r$. Then:

$$z_1 = r + e_1 \cdot x$$
$$z_2 = r + e_2 \cdot x$$

Write this as a matrix equation:

$$\begin{pmatrix} 1 & e_1 \\ 1 & e_2 \end{pmatrix} \begin{pmatrix} r \\ x \end{pmatrix} = \begin{pmatrix} z_1 \\ z_2 \end{pmatrix}$$

The determinant is $e_2 - e_1$. Since $e_1 \neq e_2$ and we're in $\mathbb{F}_q$ (a field — $q$ is prime), the determinant is nonzero, so the system has a **unique solution**. Cramer's rule gives:

$$x = \frac{z_1 - z_2}{e_1 - e_2} \pmod{q}$$

That's the knowledge extractor. It's not a clever trick — it's just solving a $2 \times 2$ linear system. The fact that $q$ is prime guarantees the determinant is invertible, which is why prime order is essential.

> **Compare to the number-theory proof:** The standard presentation says "subtract the two equations and use Fermat's little theorem to invert $(e_1 - e_2)$." That's correct, but it obscures the structure. The linear algebra version makes it obvious: two equations, two unknowns, full rank, unique solution. Done.

---

## 5. Zero-knowledge = an underdetermined system

Now look at a single transcript $(a, e, z)$ from the verifier's perspective. The verifier knows $e$ and $z$, and knows $y = g^x$ (so implicitly knows $x$ exists). The linear equation is:

$$z = r + e \cdot x$$

One equation, two unknowns $(r, x)$. This system is **underdetermined** — it has a one-dimensional solution space (a line in $\mathbb{F}_q^2$). For every possible value of $x$, there's exactly one $r$ that makes the equation work: $r = z - ex$.

This means: the transcript $(e, z)$ is **consistent with every possible secret** $x \in \mathbb{F}_q$. No value of $x$ is ruled out. That's zero-knowledge — the transcript carries no information about $x$.

The simulator makes this explicit. To fake a transcript without knowing $x$:

1. Pick $z$ and $e$ uniformly at random from $\mathbb{F}_q$.
2. Compute $a = g^z \cdot y^{-e}$ (i.e., set $r = z - ex$ implicitly).
3. Output $(a, e, z)$.

The simulator is just picking a random point on the solution line. Since $r$ was uniform in the real protocol and determines a unique point on the same line, the distributions are identical.

> **The geometric picture:** The secret $(r, x) \in \mathbb{F}_q^2$ lives on the line $z = r + ex$. The transcript reveals the line (via $e$ and $z$), but every point on the line is equally likely. Observing the line tells you nothing about which point the prover used. That's zero-knowledge, geometrically.

---

## 6. Pedersen commitments are linear combinations

A Pedersen commitment uses two generators $g, h$ of the same prime-order group (nobody knows $\log_g h$):

$$C(v, r) = g^v \cdot h^r$$

In exponent space, define $\phi: \mathbb{F}_q^2 \to G$ by:

$$\phi(v, r) = g^v \cdot h^r$$

This is a **linear map** from $\mathbb{F}_q^2$ to $G$. The commitment $C$ is a linear combination of the two "basis vectors" $g$ and $h$, with coefficients $v$ and $r$. Everything you know about linear combinations transfers immediately.

### Hiding = the map has a kernel

For any target $C \in G$, the preimage $\phi^{-1}(C)$ is a **coset** — a translated copy of the kernel. Since $\phi$ maps $\mathbb{F}_q^2$ (2-dimensional) to $G$ (1-dimensional), the kernel is 1-dimensional: a line through the origin. Every commitment $C$ is consistent with a line's worth of $(v, r)$ pairs.

Concretely: if $C = g^v h^r$, then $C = g^{v'} h^{r'}$ for any $v' = v + t$ and $r' = r - t \cdot \log_g h$, as $t$ ranges over $\mathbb{F}_q$. Since $\log_g h$ is unknown, an adversary can't navigate this line — but the *existence* of the line is what makes the commitment information-theoretically hiding.

### Binding = the kernel is hard to find

If you could open the same $C$ to two different values $(v, r)$ and $(v', r')$, you'd have:

$$g^v h^r = g^{v'} h^{r'} \implies g^{v - v'} = h^{r' - r} \implies \log_g h = \frac{v - v'}{r' - r}$$

You'd have computed $\log_g h$ — breaking the discrete log assumption. So binding reduces to the hardness of finding a nonzero element of the kernel.

### Homomorphism = linearity

$$C(v_1, r_1) \cdot C(v_2, r_2) = g^{v_1} h^{r_1} \cdot g^{v_2} h^{r_2} = g^{v_1 + v_2} h^{r_1 + r_2} = C(v_1 + v_2, \; r_1 + r_2)$$

That's just $\phi(\mathbf{u}_1 + \mathbf{u}_2) = \phi(\mathbf{u}_1) \cdot \phi(\mathbf{u}_2)$ — linearity of $\phi$. The "additive homomorphism" that makes confidential transactions work is nothing more than the linearity of a map $\mathbb{F}_q^2 \to G$.

---

## 7. The Schnorr protocol as a linear map, unified

Let's put the whole Schnorr protocol in one picture. Define the map:

$$\Phi: \mathbb{F}_q^2 \to G \times \mathbb{F}_q, \qquad \Phi(r, x) = \big(g^r \cdot (g^x), \; r + ex\big) = (a \cdot y,\; z)$$

Actually, let's be cleaner. The prover's private state is $\mathbf{s} = (r, x) \in \mathbb{F}_q^2$. The public output is:

$$\text{commitment: } a = g^r \quad (\text{image of } r \text{ under } \phi)$$
$$\text{response: } z = r + ex \quad (\text{a linear functional applied to } \mathbf{s})$$

The challenge $e$ determines a linear functional $\ell_e: \mathbb{F}_q^2 \to \mathbb{F}_q$ defined by $\ell_e(r, x) = r + ex$.

**Verification** checks that $\phi(z) = \phi(r) \cdot \phi(x)^e$, i.e., $\phi(\ell_e(\mathbf{s})) = a \cdot y^e$. This is just checking that $\phi$ and $\ell_e$ are compatible — which they are, because both are linear.

**Soundness** = two distinct functionals $\ell_{e_1}, \ell_{e_2}$ jointly determine $\mathbf{s}$ uniquely (full-rank $2 \times 2$ system).

**Zero-knowledge** = a single functional $\ell_e$ leaves a 1-dimensional kernel (one equation, two unknowns).

The entire security argument is: *one linear equation doesn't determine a 2D vector; two do.* That's it.

---

## 8. Generalizing: $k$-out-of-$k$ and OR proofs

This linear perspective scales beautifully.

### Multiple secrets

Suppose you want to prove knowledge of $k$ secrets $x_1, \ldots, x_k$ simultaneously (with public keys $y_i = g^{x_i}$). You commit with $k$ random nonces $r_1, \ldots, r_k$ and respond with $k$ linear equations:

$$z_i = r_i + e \cdot x_i \pmod{q}, \quad i = 1, \ldots, k$$

The private state is $\mathbf{s} = (r_1, x_1, r_2, x_2, \ldots, r_k, x_k) \in \mathbb{F}_q^{2k}$. Each challenge gives you $k$ equations in $2k$ unknowns — still underdetermined (zero-knowledge). Two challenges give $2k$ equations in $2k$ unknowns — determined (sound).

### OR proofs (Cramer-Damgård-Schoenmakers)

An OR proof says "I know $x_1$ OR $x_2$." The trick: simulate one leg and give a real proof for the other, with the constraint $e_1 + e_2 = e$ (the combined challenge).

In linear algebra terms: you have the system

$$z_1 = r_1 + e_1 x_1, \qquad z_2 = r_2 + e_2 x_2, \qquad e_1 + e_2 = e$$

If you know $x_1$ but not $x_2$, you can freely choose $e_2, z_2, r_2$ (the simulated leg — 3 free variables), set $e_1 = e - e_2$, then solve $z_1 = r_1 + e_1 x_1$ using your knowledge of $x_1$. The verifier sees the same algebraic structure either way and can't distinguish which leg is real.

The reason this works is linear: the constraint $e_1 + e_2 = e$ is one equation, so you have one degree of freedom in how you split the challenge. That freedom is exactly what lets you simulate one leg.

---

## 9. Why finite fields make this cleaner than $\mathbb{R}$

You might wonder: if this is "just" linear algebra, why doesn't it work over $\mathbb{R}$?

It does, algebraically. But **cryptographically** it doesn't, because:

1. **Over $\mathbb{R}$, there's no one-way map.** The map $\phi(x) = g^x$ over the reals is invertible by taking $\log$ — there's no discrete log problem. The hardness comes specifically from the discrete structure of $\mathbb{F}_q$.

2. **Over $\mathbb{R}$, uniform sampling is impossible.** ZK proofs need $r$ to be uniformly random. You can't sample uniformly from $\mathbb{R}$, but you can sample uniformly from $\mathbb{F}_q = \{0, 1, \ldots, q-1\}$.

3. **Over $\mathbb{R}$, rounding leaks information.** Floating-point representations have finite precision, so operations leak bits about the inputs. $\mathbb{F}_q$ arithmetic is exact.

So the linear algebra is identical in structure, but the **finiteness** of $\mathbb{F}_q$ is what makes it cryptographically useful. You get perfect algebraic cancellation (like the simulator's fake transcripts matching real ones exactly) *plus* computational hardness (the encoding map $\phi$ can't be inverted).

---

## 10. A dictionary

For reference, here's how to translate between the standard crypto presentation and the linear algebra view:

| Standard presentation | Linear algebra over $\mathbb{F}_q$ |
|---|---|
| Pick random nonce $r$ | Sample a random vector component |
| Compute $a = g^r$ | Apply the encoding map $\phi(r)$ |
| Response $z = r + ex$ | Evaluate a linear functional $\ell_e(\mathbf{s})$ at the secret vector $\mathbf{s} = (r, x)$ |
| Verify $g^z = a \cdot y^e$ | Check $\phi(\ell_e(\mathbf{s})) = \phi(r) \cdot \phi(x)^e$ (compatibility of $\phi$ and $\ell_e$) |
| Knowledge extraction | Solve a full-rank linear system |
| Simulator | Pick a random point in the solution set of an underdetermined system |
| Pedersen commitment $g^v h^r$ | Linear map $\phi: \mathbb{F}_q^2 \to G$, evaluate at $(v, r)$ |
| Hiding | $\phi$ has a nontrivial kernel (rank < input dimension) |
| Binding | Finding a kernel element requires breaking discrete log |
| Additive homomorphism | Linearity of $\phi$ |
| Prime group order $q$ | Working over a field (every nonzero element is invertible) |

---

## The takeaway

The number-theory approach to ZK proofs — Fermat's little theorem, Bézout's identity, modular inverses — gets you to the same place, but it can feel like a bag of tricks. The linear algebra view reveals the underlying geometry:

- **Soundness** is a full-rank system (unique solution).
- **Zero-knowledge** is an underdetermined system (a free variable).
- **Commitments** are linear maps with hard-to-find kernels.
- **Homomorphism** is just linearity.

If you're comfortable with null spaces, rank, and solving $A\mathbf{x} = \mathbf{b}$, you already have the intuition for everything in the [ZK proofs post](https://kchu25.github.io/blockchains/blog/general/zkproofs/). The only new ingredient is that the field is $\mathbb{F}_q$ instead of $\mathbb{R}$, and the encoding map $\phi = g^{(\cdot)}$ is one-way.
