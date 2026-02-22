@def title = "Section 1, De-Jargoned (Nigel Smart's Cryptography book)"
@def published = "21 February 2026"
@def tags = ["group-theory", "cryptography"]

# Section 1, De-Jargoned

> **Source:** Nigel Smart, *Cryptography: An Introduction* (3rd Edition)
> [https://www.cs.umd.edu/~waa/414-F11/IntroToCrypto.pdf](https://www.cs.umd.edu/~waa/414-F11/IntroToCrypto.pdf)

## What's the point of Section 1?

Before you can understand crypto, you need one big idea: **math that wraps around**. That's basically it. Everything in Section 1 is scaffolding for that idea.

---

## Modular Arithmetic — "Clock Math"

You already know this. On a 12-hour clock, if it's 10 o'clock and you wait 5 hours, it's **3** o'clock — not 15. You wrapped around.

That's modular arithmetic. We write:

$$10 + 5 \equiv 3 \pmod{12}$$

Read it as: "10 + 5 lands on 3, when you're working mod 12."

More generally, $a \equiv b \pmod{N}$ just means $a$ and $b$ have the same remainder when divided by $N$. For example:

$$17 \equiv 2 \pmod{5} \quad \text{because } 17 = 3 \times 5 + 2$$

---

## What is $\mathbb{Z}/N\mathbb{Z}$?

This notation looks terrifying. It's not. It's just a fancy way to write "integers mod $N$."

**$\mathbb{Z}$** = all integers: ..., -2, -1, 0, 1, 2, ...

**$N\mathbb{Z}$** = all *multiples of $N$*. For $N=5$: {..., -10, -5, 0, 5, 10, ...}

**The "/" means "quotient by"** — but *not* ordinary division. It means: "collapse all the multiples of $N$ into a single equivalence class."

**$\mathbb{Z}/N\mathbb{Z}$** literally means: "Take all integers, and treat any two integers that differ by a multiple of $N$ as the *same thing*."

### Concrete example with $N = 5$:

- 0 and 5 differ by 5 (a multiple of 5) → *same thing*
- 1 and 6 differ by 5 → *same thing*
- 2 and 7 differ by 5 → *same thing*
- 3 and 8 differ by 5 → *same thing*
- 4 and 9 differ by 5 → *same thing*
- 7 and 12 differ by 5 → *same thing*

So the integers collapse into just 5 "buckets" — one for each remainder:

$$\mathbb{Z}/5\mathbb{Z} = \{\text{[all numbers with remainder 0]}, \text{[all with remainder 1]}, \text{[all with remainder 2]}, \text{[all with remainder 3]}, \text{[all with remainder 4]}\}$$

You represent each bucket by its smallest non-negative element:

$$\mathbb{Z}/5\mathbb{Z} = \{0, 1, 2, 3, 4\}$$

When you add two of these, if the result "overflows," you wrap around:

$$3 + 4 = 7 \equiv 2 \pmod 5$$

because 7 and 2 are in the same bucket (they differ by 5).

---

## Why "group"?

A **group** is just a fancy word for: *a set of things + an operation, where a few sanity rules hold*:

1. **Closure** — doing the operation stays inside the set. ($3 + 4 \pmod 5 = 2$ ✓ still in the set)
2. **Identity** — there's a "do nothing" element. (Adding 0 changes nothing ✓)
3. **Inverses** — every element has an "undo." (The inverse of 3 mod 5 is 2, since $3 + 2 = 5 \equiv 0$)
4. **Associativity** — order of grouping doesn't matter. ($(a+b)+c = a+(b+c)$ ✓)

So $(\mathbb{Z}/N\mathbb{Z}, +)$ — meaning "the set $\{0, 1, \ldots, N-1\}$ with addition mod $N$" — is a group. That's all the notation means.

---

## Why does crypto care?

Crypto needs math that's **easy to compute forward, hard to reverse**. Modular arithmetic is perfect for this:

- **Easy:** compute $3^{100} \pmod{17}$ with a computer in milliseconds
- **Hard:** given the result, figure out the exponent (this is the **discrete log problem** — the basis of many cryptosystems)

The "group" framing is just a clean way to describe the rules of the arithmetic you're playing in — so that theorems proved about abstract groups automatically apply to your specific crypto system.

---

## How do you actually compute $a^k \pmod{N}$ fast?

The naive approach — multiply $a$ by itself $k$ times — is hopeless. In crypto, $k$ might be a 2048-bit number. You'd be multiplying billions of billions of times.

The trick is **repeated squaring** (also called *fast exponentiation* or *exponentiation by squaring*). The key insight: instead of computing

$$a^{100} = a \times a \times \cdots \times a \quad (100 \text{ multiplications})$$

notice that you can **keep squaring**:

$$a^1 \to a^2 \to a^4 \to a^8 \to a^{16} \to a^{32} \to a^{64}$$

That's only 6 squarings to reach $a^{64}$. Then you build up 100 from powers of 2:

$$100 = 64 + 32 + 4 \quad \Rightarrow \quad a^{100} = a^{64} \cdot a^{32} \cdot a^{4}$$

And since you apply $\pmod{N}$ **at every step**, the numbers never blow up — they stay in $\{0, \ldots, N-1\}$ the whole time.

### The algorithm

1. Write the exponent $k$ in binary
2. Start with `result = 1`, `base = a mod N`
3. For each bit of $k$ from least to most significant:
   - If the bit is 1, multiply `result *= base`, then take mod $N$
   - Square `base = base² mod N`

### Runtime

Each squaring/multiplication involves numbers at most $N^2$ in size, and there are $\log_2(k)$ bits in $k$. So the total number of multiplications is $O(\log k)$ — astronomically better than the naive $O(k)$.

For a 2048-bit exponent, that's ~2048 multiplications instead of $2^{2048}$. That's the difference between "done instantly" and "heat death of the universe."

### Julia snippet

```julia
function modexp(base, exp, mod)
    result = 1
    base = base % mod

    while exp > 0
        if exp % 2 == 1          # if current bit is 1
            result = (result * base) % mod
        end
        exp = exp >> 1           # shift to next bit
        base = (base * base) % mod
    end

    return result
end

# Example: compute 3^100 mod 17
println(modexp(3, 100, 17))   # → 12

# Works on crypto-scale numbers too:
println(modexp(2, BigInt(2)^256, BigInt(10)^50 + 151))
```

> **Note:** Julia handles `BigInt` natively, so you can throw genuinely huge crypto-scale numbers at this and it just works. In practice, Julia's built-in `powermod(base, exp, mod)` does exactly this under the hood.