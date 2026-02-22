@def title = "Chinese Remainder Theorem"
@def published = "21 February 2026"
@def tags = ["group-theory", "cryptography"]

# Chinese Remainder Theorem

## So, what's the big idea?

Imagine someone tells you: *"I have a number. When I divide it by 3, I get remainder 2. When I divide it by 5, I get remainder 3. What's my number?"*

That's exactly the kind of puzzle the **Chinese Remainder Theorem (CRT)** was built to solve.

---

## The Setup

You've got a system of congruences that looks like this:

$$x \equiv r_1 \pmod{m_1}$$
$$x \equiv r_2 \pmod{m_2}$$
$$\vdots$$
$$x \equiv r_k \pmod{m_k}$$

Where $r_i$ are your remainders and $m_i$ are your divisors (called **moduli**).

---

## The Key Condition

Here's the catch — the theorem only works when your moduli are **pairwise coprime**. That just means every pair shares no common factor other than 1:

$$\gcd(m_i, m_j) = 1 \quad \text{for all } i \neq j$$

So 3, 5, and 7 are fine. But 4 and 6? Nope — they share a factor of 2.

---

## The Guarantee

If that coprime condition holds, CRT tells you two beautiful things:

1. **A solution always exists.**
2. **It's unique** modulo $M = m_1 \cdot m_2 \cdots m_k$.

So there's exactly one answer in the range $[0, M)$, and infinitely many answers spaced $M$ apart after that.

---

## Back to Our Example

$$x \equiv 2 \pmod{3}, \quad x \equiv 3 \pmod{5}$$

Since $\gcd(3, 5) = 1$, we're good. The answer? $x = 8$. Check it:
- $8 = 2 \cdot 3 + 2$ ✓
- $8 = 1 \cdot 5 + 3$ ✓

And the next solution would be $8 + 15 = 23$, then $38$, and so on.

---

## Why Should You Care?

CRT pops up *everywhere*:

- **Cryptography** — RSA and other systems use it to speed up computations massively.
- **Computer arithmetic** — Breaking big number problems into smaller, easier ones.
- **Scheduling & puzzles** — Any time you're syncing periodic events.

---

## The Takeaway

CRT is essentially saying: *if your moduli don't "overlap" (i.e., they're coprime), you can always stitch together a bunch of remainder conditions into one unique solution.* Pretty neat for a theorem that's over 1,500 years old! 🎉