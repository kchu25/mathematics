@def title = "Linear map through vectorization"
@def published = "6 October 2025"
@def tags = ["linear-algebra"]

# Linear map through vectorization

## The Big Picture

Imagine you have a matrix $x$ of size $L \times M$ (think of it as a small image or data grid), and you want to transform it into another matrix of size $S \times P$. The most general way to do this linearly is to have a weight for *every possible connection* between input and output positions.

## The Transformation

Here's the formula that captures this idea:

$$\text{output}[s,p] = \sum_{l=1}^{L} \sum_{m=1}^{M} w[s,p,l,m] \cdot x[l,m]$$

What this says: *"To get the value at output position $(s,p)$, take a weighted sum of ALL input values, where the weight depends on both where you're reading from $(l,m)$ and where you're writing to $(s,p)$."*

The weight tensor $w$ has four indices because it needs to know:
- Where in the output? → $(s, p)$
- Where in the input? → $(l, m)$

## Why Is This Linear?

A transformation is linear if it plays nice with addition and scalar multiplication:

**Adding inputs:** If you add two inputs together first, then transform, you get the same result as transforming each separately and adding the outputs:
$$f(x + y) = f(x) + f(y)$$

**Scaling inputs:** If you multiply the input by some constant, the output gets multiplied by the same constant:
$$f(c \cdot x) = c \cdot f(x)$$

Our transformation satisfies both properties because we're just doing multiplication and addition—no nonlinear operations like squaring or taking maximums.

## The Matrix Formulation: Flattening Everything

Here's the key insight: **any linear transformation can be written as matrix multiplication**. To do this, we need to "flatten" our 2D matrices into 1D vectors.

### Vectorization: Stacking Columns

The standard way to flatten a matrix is **column-major vectorization**—we stack the columns on top of each other:

$$\text{vec}(x) = \begin{bmatrix} x[1,1] \\ x[2,1] \\ \vdots \\ x[L,1] \\ x[1,2] \\ \vdots \\ x[L,M] \end{bmatrix}$$

Think of it like reading a spreadsheet column by column, top to bottom.

### The Matrix Form

Once everything is vectorized, our transformation becomes simple matrix multiplication:

$$\text{vec}(\text{output}) = W \cdot \text{vec}(x)$$

where $W$ is an $(SP) \times (LM)$ matrix. Each entry of $W$ corresponds to one weight from our original 4D tensor:

$$W_{j,i} = w[s,p,l,m]$$

The trick is figuring out how the indices $(s,p,l,m)$ map to the flat indices $j$ and $i$.

## The Index Mapping: The Bookkeeping

### From Multi-dimensional to Flat (Forward)

For an element at position $(l, m)$ in an $L \times M$ matrix:
$$\text{flat index} = l + (m-1) \cdot L$$

This formula says: *"Skip $(m-1)$ complete columns of length $L$, then move down $l$ rows."*

Similarly, for the output position $(s, p)$ in an $S \times P$ matrix:
$$\text{flat index} = s + (p-1) \cdot S$$

**So the complete mapping is:**
$$W_{s + (p-1)S, \, l + (m-1)L} = w[s,p,l,m]$$

### From Flat Back to Multi-dimensional (Inverse)

If you have a flat index $i$ and want to recover the original position $(l, m)$ in an $L \times M$ matrix:

$$l = ((i-1) \bmod L) + 1$$
$$m = \left\lfloor \frac{i-1}{L} \right\rfloor + 1$$

The intuition: 
- The remainder when dividing by $L$ tells you the row
- The quotient tells you which column you're in

## Why Does This Matter?

This framework shows that seemingly complex tensor operations are just matrix multiplications in disguise. This is powerful because:

1. **Theoretical clarity**: We can use all the tools from linear algebra
2. **Computational efficiency**: Optimized matrix multiplication libraries (BLAS, etc.)
3. **Implementation simplicity**: Reshape, multiply, reshape back

The cost? We need to keep track of these index mappings, which is the tedious part—but it's just bookkeeping!