@def title = "The Math Behind ZK Proofs: Groups, Modular Arithmetic, and Why Any of It Matters"
@def published = "21 February 2026"
@def tags = ["group-theory", "cryptography"]

# The Math Behind ZK Proofs: Groups, Modular Arithmetic, and Why Any of It Matters

## Is This the Bread and Butter of Cryptography?

Yes. Uncomfortably so.

Almost every piece of cryptography you've heard of reduces to two or three hard problems, all of which live in the structures this doc covers:

| Cryptosystem | What you're doing | Hard problem underneath |
|---|---|---|
| **RSA** | Encrypt/sign with a public key | Factoring $n = pq$ |
| **Diffie-Hellman** | Two strangers agree on a shared secret | Discrete log in $\mathbb{Z}_p^*$ |
| **Elliptic Curve Crypto** | Smaller keys, same security | Discrete log on an elliptic curve group |
| **Schnorr signatures** | Sign a message, prove identity | Discrete log in a prime-order group |
| **ZK proofs (Schnorr-style)** | Prove you know a secret without revealing it | Discrete log + group homomorphisms |
| **Pedersen commitments** | Lock in a value without showing it | Discrete log (two generators) |
| **SNARKs / STARKs** | Prove arbitrary computations | Polynomial commitments over finite fields |

The pattern is always the same: find a mathematical structure where *one direction is easy and the other is hard*, then build a protocol that exploits the gap. Groups give you the structure. Modular arithmetic gives you the concrete instantiation. Prime order gives you the security guarantee.

> **The real motivation:** You don't study this to appreciate abstract algebra. You study it because every time you see a lock icon in your browser, a blockchain transaction, or a password hash, someone made a bet that a specific hard problem in a specific group is still unsolved. Understanding the math means you understand *what that bet actually is* — and when it might go wrong.

If quantum computers ever break the discrete log problem (Shor's algorithm can do it), RSA, Diffie-Hellman, and Schnorr-style ZK proofs all fall simultaneously. The reason post-quantum cryptography is a scramble right now is that so many systems share the same mathematical foundation. That's how load-bearing this stuff is.

---

## The Real Reason We Care About "Symmetry"

When people say group theory is about *symmetry*, they mean something very specific that's easy to demonstrate. Rotate a square 90°. Then rotate it another 90°. You get a 180° rotation. Rotate it four times and you're back where you started. The collection of all these rotations forms a **group** — a set of operations that:

- **Close** on themselves: doing two rotations gives you another rotation (not some alien operation)
- Have an **identity**: "do nothing" is a valid operation
- Have **inverses**: every rotation can be undone
- Are **associative**: $(A \circ B) \circ C = A \circ (B \circ C)$ — the *grouping* of operations doesn't matter, only their left-to-right order. Rotate 90°, then 180°, then 90°: whether you mentally bundle the first two or the last two, you land in the same place. Concretely: associativity means operations are "pure sequencing" with no hidden context — each step just moves you to the next state, and those steps compose cleanly into chains. (Non-example: subtraction. $(10-3)-2 = 5$ but $10-(3-2) = 9$. Grouping changes the answer, so subtraction fails this, and integers under subtraction don't form a group.)

But *why should a cryptographer care about rotating squares?* They don't, directly. The point is that **numbers can behave the same way** — and when they do, you get powerful algebraic structure you can exploit.

---

## Modular Arithmetic: Clocks and Wrap-Around

You already know modular arithmetic. A clock wraps around: 11 + 3 = 2 (not 14). Formally, we write:

$$11 + 3 \equiv 2 \pmod{12}$$

The notation $a \equiv b \pmod{n}$ just means "$a - b$ is divisible by $n$", i.e., $a$ and $b$ have the same remainder when divided by $n$.

The set $\{0, 1, 2, \ldots, n-1\}$ with addition mod $n$ is denoted $\mathbb{Z}_n$. It forms a group — every element has an inverse, addition closes within the set, and there's an identity (0).

**Multiplication mod $n$** is trickier. Not every element has a multiplicative inverse. For example, $2 \times 3 = 6 \equiv 0 \pmod{6}$, so 2 and 3 "annihilate" each other. This is bad. But if $n$ is **prime**, then *every* nonzero element has a multiplicative inverse. This is why cryptography is obsessed with primes.

The set $\{1, 2, \ldots, p-1\}$ with multiplication mod a prime $p$ forms a group, written $\mathbb{Z}_p^*$. It has $p-1$ elements.

> **Why the star?** It means "exclude zero." $\mathbb{Z}_p = \{0, 1, \ldots, p-1\}$, but $0 \times \text{anything} = 0$ — you can never multiply your way back to 1, so 0 has no multiplicative inverse and breaks the group axioms. The star is shorthand for "units only" — elements that *do* have inverses. For prime $p$, that's everything except 0. For composite $n$, it's more selective: $\mathbb{Z}_6^* = \{1, 5\}$ because 2, 3, and 4 also lack inverses mod 6 (they share factors with 6). Another reason to love primes — $\mathbb{Z}_p^*$ loses exactly one element instead of many.

---

## The Integers Mod $p$ as a Group: Concretely

Let's use $p = 7$. Then $\mathbb{Z}_7^* = \{1, 2, 3, 4, 5, 6\}$.

Pick $g = 3$ and start computing powers mod 7:

$$3^1 = 3, \quad 3^2 = 9 \equiv 2, \quad 3^3 = 6, \quad 3^4 = 18 \equiv 4, \quad 3^5 = 12 \equiv 5, \quad 3^6 = 15 \equiv 1$$

Notice: every element of $\{1,2,3,4,5,6\}$ appeared exactly once before we cycled back to 1. So $g = 3$ is a **generator** — starting from it, you can reach every element by repeated multiplication. A group with a generator like this is called **cyclic**.

This is not always the case. For $p = 7$, try $g = 2$:

$$2^1 = 2, \quad 2^2 = 4, \quad 2^3 = 1, \quad 2^4 = 2, \ldots$$

You only hit $\{1, 2, 4\}$ — a **subgroup** of order 3 (it has 3 elements). This subgroup is itself a cyclic group.

> **What's a subgroup, really?** It's just a subset that's also a group under the same operation. $\{1, 2, 4\}$ closes under multiplication mod 7 (check: $2 \times 4 = 8 \equiv 1$, $4 \times 4 = 16 \equiv 2$, etc.), has an identity (1), and every element has an inverse — so it qualifies. Think of it like a clique inside the group that never needs to leave: any operation between two members lands back inside the clique. The full group $\mathbb{Z}_7^*$ is the whole party; the subgroup $\{1, 2, 4\}$ is a table that only talks to itself.
>
> And yes — the largest possible subgroup is the full group itself, $\{1, 2, 3, 4, 5, 6\}$. The generator that produces it (like $g = 3$ above) is called a **generator of the group**. So "generator" just means: the element whose powers sweep out the biggest possible set — the whole thing. Smaller generators produce smaller cliques; a true generator leaves no one out.

**Key insight:** The subgroup generated by $g$ contains exactly $\text{ord}(g)$ elements, where $\text{ord}(g)$ is the smallest positive integer such that $g^{\text{ord}(g)} \equiv 1 \pmod{p}$. Lagrange's theorem says the order of any subgroup must **divide** $|G| = p-1$. So if $p-1 = 6$, subgroup orders can only be 1, 2, 3, or 6.

> **The bridge from group to subgroup:** $\mathbb{Z}_p^*$ is the full group — all nonzero elements under multiplication mod $p$. But the moment you pick a generator $g$, you're carving out a *smaller* world: just the powers of $g$. That smaller world is a subgroup, and it inherits all the group properties (closure, inverses, identity) automatically — it's a self-contained group in its own right, just living inside the bigger one. The reason cryptographers care about subgroups specifically is that you want to work in the *smallest* structure that still makes discrete log hard. The full $\mathbb{Z}_p^*$ has order $p-1$, which is always even and often has many small factors — a liability, as we'll see. A carefully chosen subgroup of prime order $q$ is cleaner, tighter, and harder to attack.

---

## Why Prime-Order Subgroups?

So we've just seen that not every element generates the whole group. $g = 2$ in $\mathbb{Z}_7^*$ only generates $\{1, 2, 4\}$ — a subgroup of order 3. That subgroup is a perfectly valid group in its own right. The question for cryptography is: **which subgroup do you want to work in, and why does it matter?**

Let's think about what an attacker is trying to do. Recall that the whole point of working in a cyclic group is that $g^x \bmod p$ is our one-way function — the prover's secret is $x$, and the public key is $y = g^x \bmod p$. So attacking the system means inverting that function: given $y$, find $x$. That's the **discrete logarithm problem**, and it lives entirely inside whichever cyclic subgroup $g$ generates. The attacker's battlefield *is* that subgroup — which is exactly why the choice of subgroup matters so much. If the subgroup has order $q$, that takes up to $q$ steps. For a large prime $q$, that's infeasible. So far so good.

But here's the catch: if the group order $q$ is *composite* — say $q = 15 = 3 \times 5$ — an attacker can do something much cleverer. They can solve the discrete log *separately* in the subgroup of order 3 and the subgroup of order 5, then combine the answers using the Chinese Remainder Theorem. Each sub-problem is tiny. This is the **Pohlig-Hellman attack**, and it completely bypasses the apparent size of the group.

The upshot: the hardness of discrete log depends not on the *size* of the group order, but on its **largest prime factor**. A group of order $q = 2^{64}$ is actually easy to attack — you just solve 64 tiny discrete logs in groups of order 2. A group of order $q = $ a 256-bit prime is genuinely hard.

> **The fix is simple:** make $q$ itself prime. Then there are no smaller subgroups to decompose into (other than the trivial group of order 1), Pohlig-Hellman gets you nothing, and you're forced to attack the full problem.

The standard construction is the **safe prime**: pick $p = 2q + 1$ where both $p$ and $q$ are prime. Then $\mathbb{Z}_p^*$ has order $p - 1 = 2q$. The only prime factors are 2 and $q$. Pohlig-Hellman can exploit the factor of 2 (trivially — order-2 means only two elements) but the order-$q$ piece is untouchable. So you pick $g$ to be a generator of the order-$q$ subgroup specifically, and you're done — you're working in a prime-order group where discrete log is as hard as it gets.

---

## The Discrete Logarithm: A One-Way Function With Structure

A **one-way function** is easy to compute in one direction, hard to reverse. SHA-256 is a one-way function, but it's "dumb" — it just scrambles bits. Modular exponentiation is a one-way function that **preserves algebraic structure**:

$$f(x) = g^x \bmod p$$

Forward: fast, using repeated squaring in $O(\log x)$ multiplications.

Backward (find $x$ given $g^x \bmod p$): no efficient algorithm is known for large primes. The best take roughly $e^{O(\sqrt{\log p \cdot \log \log p})}$ operations — sub-exponential but still astronomical for 2048-bit $p$.

The *structural* property you retain:

$$g^{x+y} = g^x \cdot g^y \pmod{p}$$

This means if you know $g^x$ and $g^y$, you can compute $g^{x+y}$ — without knowing $x$ or $y$. This is **homomorphism** — a structure-preserving map between algebraic objects. And it's what makes ZK proofs possible.

---

## What Does "Group Homomorphism" Actually Mean?

A **homomorphism** $\phi: G \to H$ is a map between groups that respects the group operation:

$$\phi(a \cdot b) = \phi(a) \cdot \phi(b)$$

The map $\phi(x) = g^x \bmod p$ is a homomorphism from $(\mathbb{Z}_q, +)$ to the order-$q$ subgroup $(\langle g \rangle, \times)$:

$$g^{x+y} = g^x \cdot g^y \pmod{p}$$

Addition on the left, multiplication on the right — but the structure is preserved. The "symmetry" people keep talking about is literally this: the operation on one side *mirrors* an operation on the other, through the map $g^{(\cdot)}$.

**Why does this matter for ZK proofs?** Because in the Schnorr protocol, when you verify $g^z \equiv a \cdot y^e \pmod{p}$, you're checking a homomorphism equation:

$$g^{r + ex} = g^r \cdot (g^x)^e$$

The verifier can check this equation in the *exponent space* without ever seeing $r$ or $x$ directly. The homomorphism lets you "compute with secrets" without revealing them.

---

## Euler's Theorem and Fermat's Little Theorem

Two results you'll see used everywhere:

**Fermat's Little Theorem**: If $p$ is prime and $\gcd(a, p) = 1$, then:

$$a^{p-1} \equiv 1 \pmod{p}$$

So $a^{p-1} = 1$, $a^p = a$, $a^{p+1} = a^2$, etc. Exponents work modulo $p-1$ in $\mathbb{Z}_p^*$.

**Consequence**: To find $a^{-1} \bmod p$, compute $a^{p-2} \bmod p$. This is how modular inverses are computed in practice (using fast exponentiation).

**More general form**: In a group of order $q$, $g^q = 1$ (identity). So exponents work mod $q$. In Schnorr, the response is $z = r + ex \bmod q$ — the $\bmod q$ is essential because the exponent space wraps around at $q$.

---

## The Chinese Remainder Theorem (CRT)

If $n = p \cdot q$ where $\gcd(p, q) = 1$, then working mod $n$ is "equivalent" to working mod $p$ and mod $q$ simultaneously:

$$\mathbb{Z}_n \cong \mathbb{Z}_p \times \mathbb{Z}_q$$

The ring isomorphism is explicit: $x \bmod n$ corresponds to the pair $(x \bmod p, \; x \bmod q)$.

**Why it matters**: RSA lives in $\mathbb{Z}_{pq}^*$. The CRT lets you perform computations mod $p$ and mod $q$ separately (which is fast) and recombine. It also explains the factoring-based attacks on RSA — knowing $p$ and $q$ separately completely breaks the hardness assumption.

---

## The Quadratic Residue Setup (the Warm-Up Example)

The page's warm-up asks: prove you know $w$ such that $w^2 \equiv 4 \pmod{77}$. Let's unpack this.

$77 = 7 \times 11$. By CRT, $\mathbb{Z}_{77} \cong \mathbb{Z}_7 \times \mathbb{Z}_{11}$. An element is a **quadratic residue** mod 77 if it's a perfect square — and by CRT, that means it's a square mod 7 *and* a square mod 11.

The reason $w = 75$ also satisfies $w^2 \equiv 4 \pmod{77}$: in mod $n$ arithmetic, squaring is 2-to-1 (at best). There are actually 4 square roots of 4 mod 77: $\{2, 36, 41, 75\}$. The ZK proof works because you can commit to $r^2$ and then respond with either $r$ or $r \cdot w$, and the distribution of what you reveal looks the same regardless of which root you know.

---

## Generators and the Structure of Cyclic Groups

A **cyclic group** $G = \langle g \rangle$ of order $q$ means:

$$G = \{g^0, g^1, g^2, \ldots, g^{q-1}\}$$

If $q$ is prime, **every** non-identity element is a generator. If $q$ is composite, only some elements are. This is why working in prime-order subgroups is cleaner: no "bad" generators, every element works.

The **structure theorem for finite abelian groups** says every finite abelian group is a product of cyclic groups of prime-power order. So cyclic groups are the atomic building blocks. When cryptographers say "work in a group of prime order $q$," they mean: one atomic piece, no substructure to exploit.

---

## Putting It Together: Why Schnorr Works

The Schnorr protocol's correctness, soundness, and zero-knowledge all reduce to algebraic facts about the cyclic group:

**Correctness** is just the homomorphism $g^{r+ex} = g^r (g^x)^e$.

**Soundness** relies on the fact that if you could answer two different challenges $e_1 \neq e_2$ for the same commitment $a$, then:

$$z_1 - z_2 = (e_1 - e_2) \cdot x \pmod{q}$$

Since $q$ is prime, $(e_1 - e_2)$ has a multiplicative inverse mod $q$ (Fermat's little theorem), so:

$$x = (z_1 - z_2)(e_1 - e_2)^{-1} \pmod{q}$$

If the group had composite order, $(e_1 - e_2)$ might not be invertible, breaking the argument.

**Zero-knowledge** is the surprising one. The simulator doesn't know $x$ but still produces indistinguishable transcripts by working *backwards*: pick $z$ and $e$, then solve $a = g^z y^{-e}$. This only works because you can compute $y^{-e} = (g^x)^{-e} = g^{-ex}$ without knowing $x$ — you're using the group structure, not the secret. The discrete log hardness ensures the verifier can't tell "forward" proofs from "backward" simulations.

---

## The Pedersen Commitment: Hiding Behind a Second Generator

Pedersen commitments use *two* generators $g$ and $h$ in the same cyclic group:

$$C(v, r) = g^v \cdot h^r \bmod p$$

The **hiding** property: given $C$, you can't find $v$ because $r$ is uniformly random. Different values of $v$ are indistinguishable — you can always find an $r'$ that produces the same commitment for any $v'$ you want, *if you know $\log_g h$*. But since nobody knows $\log_g h$ (it's chosen "nothing up my sleeve"), you're computationally stuck.

The **binding** property: you can't open $C$ to two different values. If you could find $(v, r)$ and $(v', r')$ with the same $C$, then $g^v h^r = g^{v'} h^{r'}$, giving $g^{v-v'} = h^{r'-r}$, so $\log_g h = (v-v')(r'-r)^{-1}$ — contradicting hardness of discrete log.

The **homomorphic** property: $C(v_1, r_1) \cdot C(v_2, r_2) = C(v_1 + v_2, r_1 + r_2)$. This is just algebra on the exponents. It's the key to confidential transactions — you can verify commitments balance (inputs = outputs) without ever seeing the amounts.

---

## The Abstraction Ladder

To summarize why these abstractions stack the way they do:

1. **Integers mod $n$** — the concrete object. Easy to compute with.
2. **Groups** — the abstraction that captures "closure, inverse, identity, associativity." You abstract away *what* the elements are to reason about *structure* alone.
3. **Cyclic groups** — the cleanest groups. One generator, everything reachable.
4. **Prime-order cyclic groups** — no exploitable substructure. The discrete log is maximally hard.
5. **Homomorphisms** — structure-preserving maps. Let you "compute on secrets" without revealing them.
6. **One-way homomorphisms** ($x \mapsto g^x$) — easy to apply, hard to invert. The core of asymmetric cryptography.

Each layer strips away unnecessary detail to expose what actually drives the security. The discrete log assumption lives in step 4–5. If you just said "I'll work with some numbers," you'd need to re-prove security for every different instantiation. The group-theoretic language lets you write one proof and apply it everywhere — elliptic curves, classic mod-$p$, anything satisfying the group axioms.

That's the real reason for the abstraction: **one proof to rule them all**.

---

## What's Left (Briefly)

The blog post gestures toward modern ZK systems like SNARKs and STARKs. These go beyond discrete-log-based ZK into:

- **Polynomial commitments**: commit to a polynomial $f(x)$, prove evaluations without revealing $f$. Requires **bilinear pairings** — maps $e: G_1 \times G_2 \to G_T$ that are linear in both arguments. More group theory.
- **Arithmetic circuits**: express a computation as a circuit of additions and multiplications, then prove correct execution. The "language" the ZK system speaks.
- **Elliptic curve groups**: the same cyclic group structure, but the "elements" are points on a curve and the "operation" is point addition. Much shorter keys for the same security level.

All of it runs on the same underlying machinery: find a group where the right operations are easy and the wrong ones are hard, and exploit the algebraic structure to prove statements about hidden witnesses.