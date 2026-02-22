@def title = "Modular Arithmetic from Scratch: Why Clocks Lead to Cryptography"
@def published = "21 February 2026"
@def tags = ["group-theory", "cryptography"]

# Modular Arithmetic from Scratch: Why Clocks Lead to Cryptography

*A companion to "The Math Behind ZK Proofs" — building intuition before the abstraction.*

---

## Start with a clock

You've done modular arithmetic your whole life. If it's 10 o'clock now, what time is it in 5 hours? Not 15 — it's 3. The number line *wraps around* at 12.

That wrapping is the heart of everything. We write it as:

$$10 + 5 \equiv 3 \pmod{12}$$

Let's unpack the notation carefully. The $\equiv$ sign (not $=$) signals that we're not saying $10 + 5$ *equals* $3$ as numbers — it doesn't. What we're saying is that $10 + 5$ and $3$ are **equivalent under the rule "divide by 12 and look at the remainder."** Both $15$ and $3$ leave a remainder of $3$ when divided by $12$, so they're considered the same under this rule.

The $\pmod{12}$ on the right isn't an operation applied to just one side — it's a **label for the rule** that governs the whole line. It says: "the $\equiv$ you're looking at is the one where two numbers are equivalent when they differ by a multiple of $12$." Think of it as setting the context: everything in this statement lives on a clock with 12 positions, and $\pmod{12}$ is how we announce that.

Nothing mysterious. Division just leaves a leftover, and we're tracking the leftover.

---

## What happens when you multiply, and keep wrapping?

Let's pick a small number to play with — say, work mod $7$. Start with $3$ and keep multiplying by $3$, wrapping each time:

$$3^1 = 3$$
$$3^2 = 9 \equiv 2 \pmod 7$$
$$3^3 = 27 \equiv 6 \pmod 7$$
$$3^4 = 81 \equiv 4 \pmod 7$$
$$3^5 = 243 \equiv 5 \pmod 7$$
$$3^6 = 729 \equiv 1 \pmod 7$$

Something interesting just happened: we hit every nonzero number in $\{1, 2, 3, 4, 5, 6\}$ before cycling back to 1. The powers of 3 *visit the entire set* before repeating. Try this with base $2$, and you get a different story:

$$2^1 = 2, \quad 2^2 = 4, \quad 2^3 = 8 \equiv 1 \pmod 7$$

Now it cycles with period 3, only visiting $\{1, 2, 4\}$. Same mod, different base, totally different behavior.

This isn't a coincidence or a curiosity. This pattern — *some bases sweep out everything, others only sweep out part* — is the engine behind modern cryptography.

---

## Why primes?

Try doing this mod $6$ instead. Pick base $2$:

$$2^1 = 2, \quad 2^2 = 4, \quad 2^3 = 8 \equiv 2 \pmod 6$$

We're stuck in a loop immediately: $\{2, 4, 2, 4, \ldots\}$. We never even hit 1. Why?

Let's back up. In ordinary arithmetic, if you multiply a number by $2$, you can undo it by multiplying by $\frac{1}{2}$. That's the **multiplicative inverse** of $2$ — the thing that, when multiplied by $2$, gives you back $1$.

Now we're working mod $6$, so fractions are off the table. The only numbers in our world are $\{1, 2, 3, 4, 5\}$. The question is: is there a whole number $b$ in that set such that

$$2 \times b \equiv 1 \pmod 6?$$

Let's just check all of them:

$$2 \times 1 = 2, \quad 2 \times 2 = 4, \quad 2 \times 3 = 6 \equiv 0, \quad 2 \times 4 = 8 \equiv 2, \quad 2 \times 5 = 10 \equiv 4$$

None of them give 1. There is no inverse. The problem is that $2$ and $6$ share a common factor — $\gcd(2, 6) = 2 \neq 1$ — so every multiple of 2 is even, and 1 is odd, so we can never reach it. More dramatically: $2 \times 3 = 6 \equiv 0 \pmod 6$. Two nonzero numbers multiplied together gave zero — they "annihilated" each other. Once that's possible, the arithmetic becomes treacherous.

Switch to mod $7$ (a prime), and every nonzero number *does* have an inverse. For example:

$$3 \times 5 = 15 \equiv 1 \pmod 7$$

So 5 is the inverse of 3 — multiplying by 5 undoes multiplying by 3. You can verify the others: $2^{-1} \equiv 4$, $6^{-1} \equiv 6$, and so on. Every element is reachable and every operation is reversible.

Why does primality guarantee this? If $p$ is prime and $a$ is not divisible by $p$, then $\gcd(a, p) = 1$ — they share no common factors. That's enough (by Bézout's theorem) to guarantee a whole-number solution to $a \times b \equiv 1 \pmod p$ always exists. Composites fail because they have factors, and any $a$ sharing a factor with $n$ can never multiply its way to 1.

This is why cryptography is obsessed with primes — they guarantee the arithmetic is well-behaved in a way composites aren't.

---

## The pattern worth naming

Let's be explicit about what we observed with base $3$ mod $7$. Repeated multiplication wraps around, and the resulting sequence has a *period* — the smallest $k$ such that $3^k \equiv 1 \pmod 7$. Here that's $k = 6$.

More remarkably, $6 = 7 - 1$. That's not a coincidence: **Fermat's Little Theorem** says that for any prime $p$ and any $a$ not divisible by $p$:

$$a^{p-1} \equiv 1 \pmod p$$

So the period of repeated multiplication always *divides* $p - 1$. For $p = 7$, that means periods can only be $1, 2, 3,$ or $6$. And bases like $3$ that achieve the maximum period ($p - 1$) are special — they visit every nonzero element. We'll call such a base a *generator*.

---

## A one-way street appears

Here's where things get interesting. Computing $3^{40} \pmod 7$ is fast — you can do it by repeatedly squaring (take $3^2$, square it to get $3^4$, square again for $3^8$, and so on). About $\log_2(40)$ multiplications.

But now ask the reverse question: given that $3^x \equiv 4 \pmod 7$, what is $x$?

For mod $7$ you can just try all 6 possibilities. But mod a 2048-bit prime $p$? There are roughly $2^{2048}$ possibilities. No known algorithm can invert this efficiently.

This asymmetry — fast forward, slow backward — is the **discrete logarithm problem**. We write $x = \log_g y$ to mean "the $x$ such that $g^x \equiv y$," and we've just stumbled upon why anyone cares: *you can publish $y = g^x \pmod p$ without revealing $x$*. That's a one-way function with a name.

---

## Why do we even care about the discrete log problem?

Imagine two people, Alice and Bob, who want to agree on a shared secret over a public channel — a channel where anyone can listen. Here's a trick:

1. They publicly agree on a prime $p$ and a generator $g$.
2. Alice picks a secret $a$, sends Bob $A = g^a \pmod p$.
3. Bob picks a secret $b$, sends Alice $B = g^b \pmod p$.
4. Alice computes $B^a = g^{ab} \pmod p$. Bob computes $A^b = g^{ab} \pmod p$.

They both arrive at $g^{ab}$ without ever transmitting it. An eavesdropper sees $g^a$ and $g^b$ — but to get $g^{ab}$, they'd need to recover either $a$ or $b$, which means solving a discrete log. This is **Diffie-Hellman key exchange**, and it works entirely because exponentiation mod a prime is a one-way function.

The discrete log problem isn't just a curiosity — it's the *specific computational assumption* that makes secret communication between strangers possible.

---

## So what's a "group" and why does anyone formalize it?

By now you've seen a pattern repeat across different settings:

- Integers mod $p$ under addition: everything wraps, you can always undo addition.
- Nonzero integers mod $p$ under multiplication: everything wraps, you can always undo multiplication (since $p$ is prime).

Both systems share the same skeleton:
- There's a *set* of elements.
- There's an *operation* that combines two elements and stays inside the set.
- There's an *identity* element (adding 0, or multiplying by 1).
- Every element has an *inverse* (you can undo any operation).
- The operation is *associative* (the order of grouping doesn't matter).

A **group** is just a name for any system with this skeleton. Formalizing it isn't pedantry — it means any theorem you prove about groups applies *everywhere the skeleton shows up*: integers mod $p$, elliptic curve points, permutations of a Rubik's cube, whatever.

In cryptography, the payoff is concrete: once you prove that the Schnorr proof system is secure in "any prime-order group," you get security for free on elliptic curves, classic modular arithmetic, and any future structure that fits the definition. One proof, many instantiations.

---

## The shape of the exponent map

There's one more structure worth noticing before moving to the full ZK story. The map $x \mapsto g^x \pmod p$ does something elegant to addition:

$$g^{x + y} = g^x \cdot g^y \pmod p$$

This means: if you know $g^x$ and $g^y$, you can compute $g^{x+y}$ just by multiplying them — without knowing $x$ or $y$ themselves. Addition on the left becomes multiplication on the right, through the map.

This is called a **homomorphism** — a map that preserves structure. And it's the key ingredient in ZK proofs: the prover can convince you of facts about their secret $x$ by working in $g^x$-space, where you can verify relationships without ever seeing $x$ directly.

Everything in the [main post](https://kchu25.github.io/mathematics/blog/group_theory/group_zk/) — Schnorr proofs, Pedersen commitments, the whole edifice — rests on exactly this one observation about how exponentiation talks to addition.

---

*Next: see how the group formalism and the discrete log assumption combine to make zero-knowledge proofs possible.*