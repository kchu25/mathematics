@def title = "The Math You Need for Zero-Knowledge Proofs"
@def published = "21 February 2026"
@def tags = ["group-theory", "cryptography"]

# The Math You Need for Zero-Knowledge Proofs

*Everything here feeds directly into the [ZK proofs post](https://kchu25.github.io/blockchains/blog/general/zkproofs/). The goal is one clean path: clock arithmetic → groups → the discrete log trick → how Schnorr proofs and Pedersen commitments actually work.*

---

## 1. Numbers that wrap around

You already do modular arithmetic every day. It's 10 o'clock; 5 hours later it's 3, not 15. The number line wraps at 12. We write:

$$10 + 5 \equiv 3 \pmod{12}$$

The $\equiv$ sign means "same remainder when you divide by 12." The $\pmod{12}$ is a label declaring the rule — everything lives on a 12-position clock. Nothing deeper than that.

Let's shrink the clock. Work mod 7, meaning every answer gets reduced to one of $\{0, 1, 2, 3, 4, 5, 6\}$.

**Addition mod 7** is a group:
- **Closure**: adding two elements always lands back in $\{0,\ldots,6\}$ after wrapping.
- **Identity**: $0$ (adding 0 changes nothing).
- **Inverses**: every number has a partner that sums to 0 mod 7. For instance $3 + 4 = 7 \equiv 0$.
- **Associativity**: grouping doesn't matter, $(a + b) + c = a + (b + c)$.

That's literally all a **group** is — a set with an operation satisfying those four properties. We'll see why this bare-bones definition is powerful in a moment.

---

## 2. Multiplication mod a prime — and why primes matter

Now switch from addition to multiplication. Take the nonzero elements $\{1,2,3,4,5,6\}$ under multiplication mod 7. Does this form a group?

Check inverses by hunting for the element $b$ such that $a \cdot b \equiv 1 \pmod{7}$:

| $a$ | $b$ | $a \cdot b \pmod{7}$ |
|-----|-----|----------------------|
| 2 | 4 | $8 \equiv 1$ |
| 3 | 5 | $15 \equiv 1$ |
| 6 | 6 | $36 \equiv 1$ |

Every nonzero element has an inverse. This set under multiplication is the group $\mathbb{Z}_7^*$, and it works because 7 is prime.

**Why must it be prime?** Try mod 6. Does 2 have a multiplicative inverse? Check every possibility:

$$2\times 1=2,\quad 2\times 2=4,\quad 2\times 3=6\equiv 0,\quad 2\times 4=8\equiv 2,\quad 2\times 5=10\equiv 4$$

None give 1. Worse, $2 \times 3 \equiv 0$ — two nonzero numbers multiplied to zero. The problem is $\gcd(2,6) = 2 \neq 1$; they share a factor. When the modulus is prime, $\gcd(a,p) = 1$ for every nonzero $a$, so inverses always exist (Bézout's theorem). That's why cryptography lives on primes.

---

## 3. Generators and cyclic groups

Take $g = 3$ in $\mathbb{Z}_7^*$ and compute successive powers:

$$3^1 = 3,\quad 3^2 = 2,\quad 3^3 = 6,\quad 3^4 = 4,\quad 3^5 = 5,\quad 3^6 = 1$$

Every element of $\{1,2,3,4,5,6\}$ appears exactly once before cycling back to 1. We say $g=3$ is a **generator** — its powers sweep out the entire group. A group where some element does this is called **cyclic**.

Not every element generates the whole group. Try $g = 2$:

$$2^1 = 2,\quad 2^2 = 4,\quad 2^3 = 1,\quad 2^4 = 2,\ldots$$

Only $\{1, 2, 4\}$ appears — a **subgroup** of order 3 (a self-contained group living inside the bigger one). Lagrange's theorem says subgroup orders must divide $|G| = p-1 = 6$, so the possibilities here are 1, 2, 3, or 6.

> **Fermat's Little Theorem** makes this precise: for any prime $p$ and any $a$ not divisible by $p$, $a^{p-1} \equiv 1 \pmod{p}$. So the "period" of repeated multiplication always divides $p-1$. In $\mathbb{Z}_7^*$, every element satisfies $a^6 \equiv 1$.

---

## 4. The one-way street: discrete logarithms

Here's where things get cryptographically interesting. Given $g$ and $p$, computing $g^x \bmod p$ is fast — repeated squaring does it in $O(\log x)$ multiplications.

Now reverse the question: given $y = g^x \bmod p$, find $x$.

For $p = 7$ you can just try all 6 values. For a 2048-bit prime? There are roughly $2^{2048}$ possibilities and no known shortcut. This asymmetry — **easy forward, hard backward** — is the **discrete logarithm problem** (DLP).

This is why you can publish $y = g^x \bmod p$ without revealing $x$. That's a one-way function, and it's the foundation of Diffie-Hellman key exchange, Schnorr signatures, and ZK proofs.

---

## 5. Why prime-order subgroups (and safe primes)

If the group order is composite, say $q = 15 = 3 \times 5$, an attacker can solve the discrete log separately in the order-3 and order-5 subgroups, then recombine via the Chinese Remainder Theorem. Each sub-problem is tiny. This is the **Pohlig-Hellman attack**.

**The fix:** work in a subgroup whose order $q$ is itself prime. Then there are no smaller subgroups to decompose into and Pohlig-Hellman gives you nothing.

**The standard construction** is a **safe prime**: pick $p = 2q + 1$ where both $p$ and $q$ are prime. Then $\mathbb{Z}_p^*$ has order $p-1 = 2q$. You choose a generator $g$ of the order-$q$ subgroup specifically, and now you're in a prime-order cyclic group where discrete log is as hard as it gets.

> In that subgroup, *every* non-identity element is a generator (because $q$ is prime). No "bad" choices — another perk of prime order.

---

## 6. The key insight: exponentiation is a homomorphism

The map $\phi(x) = g^x \bmod p$ does something elegant to addition:

$$g^{x + y} = g^x \cdot g^y \pmod{p}$$

Addition on the left becomes multiplication on the right, and the structure is preserved. A map between groups that respects the operation like this is called a **homomorphism**: $\phi(a + b) = \phi(a) \cdot \phi(b)$.

**This is the single most important fact for ZK proofs.** It means: if you know $g^x$ and $g^y$, you can compute $g^{x+y}$ — *without knowing $x$ or $y$*. You can verify algebraic relationships between secrets by working entirely in the "exponentiated" world. The secrets stay hidden; the structure is visible.

---

## 7. How Schnorr proofs use all of this

Now we have every ingredient. The Schnorr protocol proves "I know the secret $x$ behind the public key $y = g^x$" without revealing $x$. It's a three-move dance:

**Setup:** Public prime $p$, prime-order $q$ subgroup, generator $g$. Prover holds secret $x$, publishes $y = g^x \bmod p$.

**Step 1 — Commit.** Prover picks random $r$, sends $a = g^r \bmod p$.

**Step 2 — Challenge.** Verifier sends random $e \in \{1, \ldots, q-1\}$.

**Step 3 — Respond.** Prover sends $z = r + e \cdot x \pmod{q}$.

**Verification.** Verifier checks:

$$g^z \stackrel{?}{=} a \cdot y^e \pmod{p}$$

### Why does verification work?

Expand the right side using the homomorphism:

$$a \cdot y^e = g^r \cdot (g^x)^e = g^r \cdot g^{xe} = g^{r + xe} = g^z$$

It's just algebra on the exponents, made possible by the homomorphism $g^{a+b} = g^a \cdot g^b$.

### Why is it sound?

If a cheating prover $P^*$ (who doesn't know $x$) could answer *two* different challenges $e_1, e_2$ for the same commitment $a$:

$$z_1 = r + e_1 x \pmod{q}, \quad z_2 = r + e_2 x \pmod{q}$$

Subtract: $z_1 - z_2 = (e_1 - e_2) \cdot x \pmod{q}$. Since $q$ is prime, $(e_1 - e_2)$ is invertible, so:

$$x = (z_1 - z_2)(e_1 - e_2)^{-1} \pmod{q}$$

A cheater who can answer two challenges *must* know $x$. With a single random challenge from a set of size $q$, the probability of guessing correctly is $1/q$ — negligible.

> **Note:** this is exactly where prime order pays off. If $q$ were composite, $(e_1 - e_2)$ might share a factor with $q$ and not be invertible, breaking the extraction argument.

### Why is it zero-knowledge?

A **simulator** $S$ — who does *not* know $x$ — can produce transcripts indistinguishable from real ones by working backwards:

1. Pick random $z$ and $e$.
2. Compute $a = g^z \cdot y^{-e} \bmod p$.
3. Output $(a, e, z)$.

Check: $a \cdot y^e = g^z y^{-e} \cdot y^e = g^z$. ✓

The distribution of $(a, e, z)$ is identical whether it came from a real proof or the simulator. Since a fake transcript looks the same as a real one, the real transcript reveals nothing about $x$.

> **Intuition:** The random nonce $r$ acts like a one-time pad on the secret $x$. The response $z = r + ex$ mixes $x$ with fresh randomness, so every value of $z$ is equally likely regardless of $x$. The verifier sees a uniformly random number — no information leaks.

---

## 8. Pedersen commitments: hiding a value with two generators

A **commitment scheme** lets you lock in a value now and reveal it later — like sealing a number in an envelope. Pedersen commitments do this using *two* generators $g, h$ in the same prime-order group, where nobody knows $\log_g h$ (the discrete log of $h$ relative to $g$):

$$C(v, r) = g^v \cdot h^r \bmod p$$

- **Hiding:** Given $C$, you can't determine $v$ because $r$ is uniformly random. Different $(v, r)$ pairs produce the same $C$.
- **Binding:** You can't open $C$ to two different values. If $g^v h^r = g^{v'} h^{r'}$, then $g^{v - v'} = h^{r' - r}$, giving $\log_g h = (v - v')(r' - r)^{-1}$ — which would break the discrete log assumption.

### The magic: additive homomorphism

$$C(v_1, r_1) \cdot C(v_2, r_2) = g^{v_1} h^{r_1} \cdot g^{v_2} h^{r_2} = g^{v_1 + v_2} \cdot h^{r_1 + r_2} = C(v_1 + v_2,\; r_1 + r_2)$$

You can *add* committed values without opening them. This is the foundation of **confidential transactions** on blockchains: validators check that input commitments equal output commitments (inputs = outputs + fee) without ever seeing the amounts.

---

## 9. Making it non-interactive: the Fiat-Shamir transform

Interactive proofs are fine in theory, but inconvenient on a blockchain — there's no live verifier sending challenges. The **Fiat-Shamir heuristic** replaces the verifier's random challenge with a hash:

$$e = H(a \| y \| \text{context})$$

A cryptographic hash like SHA-256 behaves like a random oracle: its output is unpredictable unless you know the exact input. Since the prover commits to $a$ before computing $e$, and any change to $a$ completely changes $e$, the prover can't shop for a favorable challenge. It's as if the hash function *is* the verifier.

This turns the 3-move protocol into a single message $(a, z)$ that anyone can verify offline. The result is a **Non-Interactive Zero-Knowledge (NIZK) proof** — exactly what blockchains need.

---

## 10. The abstraction ladder (why bother with "groups" at all?)

You might wonder: why not just say "multiply numbers mod a prime" and skip the group theory? Because the abstraction buys you **portability**. Once you prove Schnorr is secure in "any prime-order cyclic group," you get security on:

- Classic modular arithmetic ($\mathbb{Z}_p^*$ subgroups)
- Elliptic curve groups (shorter keys, same security)
- Any future structure satisfying the group axioms

One proof, many instantiations. The group definition — closure, identity, inverses, associativity — is the minimal skeleton that makes all the algebra above work. Strip away anything more specific and the proofs still hold.

| Layer | What it is | Why you care |
|-------|-----------|-------------|
| Integers mod $p$ | Concrete numbers you compute with | The actual implementation |
| Group | Closure + identity + inverses + associativity | Proofs apply everywhere the axioms hold |
| Cyclic group | One generator reaches everything | Clean structure, every element is a power of $g$ |
| Prime-order cyclic group | No exploitable substructure | Discrete log is maximally hard, Pohlig-Hellman fails |
| Homomorphism ($x \mapsto g^x$) | Preserves structure across groups | Lets you verify secret relationships without seeing secrets |

---

## Where this connects

Everything in the [ZK proofs post](https://kchu25.github.io/blockchains/blog/general/zkproofs/) — the Schnorr protocol, Pedersen commitments, OR-proofs, Fiat-Shamir — rests on exactly what's above:

1. **Modular arithmetic** gives you numbers that wrap around.
2. **Primes** guarantee every element is invertible.
3. **Prime-order subgroups** make discrete log maximally hard.
4. **The homomorphism** $g^{x+y} = g^x \cdot g^y$ lets you compute on secrets without seeing them.
5. **The simulator argument** proves zero-knowledge: fake transcripts are indistinguishable from real ones because random nonces act as one-time pads.

Modern systems (SNARKs, STARKs, PLONK) go further — polynomial commitments, arithmetic circuits, bilinear pairings — but they all grow from this same soil. If you understand why $g^{r + ex} = g^r \cdot (g^x)^e$ and what that implies about information leakage, you understand the engine.
