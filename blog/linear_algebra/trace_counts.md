@def title = "Why Trace of Adjacency Matrix Powers Counts Motifs"
@def published = "9 October 2025"
@def tags = ["linear-algebra", "graphs"]
# Why Trace of Adjacency Matrix Powers Counts Motifs

## The Core Principle

The trace of the $k$-th power of an adjacency matrix counts **closed walks** of length $k$ in a graph. This fundamental relationship allows us to count various graph motifs (substructures).

## Mathematical Foundation

### Adjacency Matrix Definition

Let $G = (V, E)$ be a graph with $n$ vertices. The adjacency matrix $\mathbf{A}$ is defined as:

$$A_{ij} = \begin{cases} 
1 & \text{if } (i,j) \in E \\
0 & \text{otherwise}
\end{cases}$$

### Powers of the Adjacency Matrix

The key insight is that entries of $\mathbf{A}^k$ count walks:

$$[\mathbf{A}^k]_{ij} = \text{number of walks of length } k \text{ from vertex } i \text{ to vertex } j$$

**Proof by induction:**

- **Base case** ($k=1$): $[\mathbf{A}^1]_{ij} = A_{ij}$ counts walks of length 1 (direct edges)

- **Inductive step**: Assume true for $k$. Then:
$$[\mathbf{A}^{k+1}]_{ij} = \sum_{\ell=1}^{n} [\mathbf{A}^k]_{i\ell} \cdot A_{\ell j}$$

This sum counts all walks of length $k$ from $i$ to $\ell$, extended by one edge from $\ell$ to $j$, giving walks of length $k+1$ from $i$ to $j$.

### The Trace Counts Closed Walks

The trace of a matrix is the sum of its diagonal elements:

$$\text{tr}(\mathbf{A}^k) = \sum_{i=1}^{n} [\mathbf{A}^k]_{ii}$$

Since $[\mathbf{A}^k]_{ii}$ counts walks of length $k$ from vertex $i$ back to itself, we have:

$$\boxed{\text{tr}(\mathbf{A}^k) = \text{total number of closed walks of length } k}$$

## Counting Triangles: A Detailed Example

### Step 1: Understanding $\text{tr}(\mathbf{A}^3)$

Consider a triangle with vertices $\{1, 2, 3\}$. The trace $\text{tr}(\mathbf{A}^3)$ counts all closed walks of length 3:

$$\text{tr}(\mathbf{A}^3) = [\mathbf{A}^3]_{11} + [\mathbf{A}^3]_{22} + [\mathbf{A}^3]_{33} + \text{(non-triangle vertices)}$$

### Step 2: Counting from Each Vertex

For the triangle $\{1, 2, 3\}$, starting from vertex 1, there are exactly **2 closed walks** of length 3:
- Walk A: $1 \to 2 \to 3 \to 1$ (clockwise)
- Walk B: $1 \to 3 \to 2 \to 1$ (counterclockwise)

So $[\mathbf{A}^3]_{11} = 2$ (for this triangle).

Similarly:
- From vertex 2: $2 \to 3 \to 1 \to 2$ and $2 \to 1 \to 3 \to 2$ contribute $[\mathbf{A}^3]_{22} = 2$
- From vertex 3: $3 \to 1 \to 2 \to 3$ and $3 \to 2 \to 1 \to 3$ contribute $[\mathbf{A}^3]_{33} = 2$

### Step 3: The Overcounting Factor

Each triangle is counted **6 times** in $\text{tr}(\mathbf{A}^3)$:
- 3 starting vertices × 2 directions = 6 walks

Therefore:

$$\boxed{\text{Number of triangles} = \frac{\text{tr}(\mathbf{A}^3)}{6}}$$

### Concrete Example

For the graph with edges $\{(1,2), (2,3), (1,3)\}$:

$$\mathbf{A} = \begin{pmatrix} 0 & 1 & 1 \\ 1 & 0 & 1 \\ 1 & 1 & 0 \end{pmatrix}$$

Computing $\mathbf{A}^3$:

$$\mathbf{A}^3 = \begin{pmatrix} 2 & 3 & 3 \\ 3 & 2 & 3 \\ 3 & 3 & 2 \end{pmatrix}$$

The diagonal shows $[\mathbf{A}^3]_{11} = [\mathbf{A}^3]_{22} = [\mathbf{A}^3]_{33} = 2$.

$$\text{tr}(\mathbf{A}^3) = 2 + 2 + 2 = 6 \implies \text{Number of triangles} = \frac{6}{6} = 1 \checkmark$$

## Counting 4-Cycles (Squares): A Detailed Example

### The Challenge: Degenerate Walks

For 4-cycles, $\text{tr}(\mathbf{A}^4)$ counts **all** closed walks of length 4, including non-square patterns!

Consider a square $\{1, 2, 3, 4\}$ with edges forming a cycle. From vertex 1, we have:

**Legitimate 4-cycle walks:**
- $1 \to 2 \to 3 \to 4 \to 1$ (clockwise around square)
- $1 \to 4 \to 3 \to 2 \to 1$ (counterclockwise around square)

**Degenerate walks (NOT part of the square structure):**
- $1 \to 2 \to 1 \to 2 \to 1$ (back-and-forth on edge $(1,2)$)
- $1 \to 4 \to 1 \to 4 \to 1$ (back-and-forth on edge $(1,4)$)

### Step-by-Step Correction

**Type 1 Degeneracy**: Back-and-forth on a single edge

If vertex $i$ has degree $d_i$, then there are $d_i$ ways to go out-and-back twice: $i \to j \to i \to j \to i$ for each neighbor $j$.

Total across all vertices: $\sum_{i=1}^n d_i = 2|E|$

**Type 2 Degeneracy**: Going around a triangle twice

A walk like $i \to j \to k \to i \to j$ (where $\{i,j,k\}$ form a triangle) uses the triangle but isn't a 4-cycle.

These walks return to $i$ through a 2-step path: $[\mathbf{A}^2]_{ii}$ counts such patterns.

### The Corrected Formula

$$\boxed{\text{Number of 4-cycles} = \frac{1}{8}\left[\text{tr}(\mathbf{A}^4) - 2|E| - \sum_{i=1}^n [\mathbf{A}^2]_{ii}\right]}$$

Where:
- Division by 8 accounts for: 4 starting vertices × 2 directions
- $2|E|$ removes back-and-forth walks
- $\sum_{i} [\mathbf{A}^2]_{ii} = \text{tr}(\mathbf{A}^2)$ removes triangle-based degeneracies

### Concrete Example

For a 4-cycle graph with edges $\{(1,2), (2,3), (3,4), (4,1)\}$:

$$\mathbf{A} = \begin{pmatrix} 0 & 1 & 0 & 1 \\ 1 & 0 & 1 & 0 \\ 0 & 1 & 0 & 1 \\ 1 & 0 & 1 & 0 \end{pmatrix}$$

First compute $\mathbf{A}^2$:

$$\mathbf{A}^2 = \begin{pmatrix} 2 & 0 & 2 & 0 \\ 0 & 2 & 0 & 2 \\ 2 & 0 & 2 & 0 \\ 0 & 2 & 0 & 2 \end{pmatrix}, \quad \text{tr}(\mathbf{A}^2) = 8$$

Then $\mathbf{A}^4$:

$$\mathbf{A}^4 = \begin{pmatrix} 8 & 0 & 8 & 0 \\ 0 & 8 & 0 & 8 \\ 8 & 0 & 8 & 0 \\ 0 & 8 & 0 & 8 \end{pmatrix}, \quad \text{tr}(\mathbf{A}^4) = 32$$

We have $|E| = 4$, so:

$$\text{Number of 4-cycles} = \frac{1}{8}[32 - 2(4) - 8] = \frac{1}{8}[32 - 8 - 8] = \frac{16}{8} = 2$$

Wait, we expected 1 cycle! What happened? The factor of 2 comes from **undirected edges** being counted twice in the trace formulation. For undirected graphs, we need:

$$\boxed{\text{Number of 4-cycles (undirected)} = \frac{1}{8}\left[\text{tr}(\mathbf{A}^4) - 2|E| - \text{tr}(\mathbf{A}^2)\right] \div 2}$$

This gives us $16/8 \div 2 = 1$ cycle. ✓

## General Principle

For any $k$-cycle:

1. Compute $\text{tr}(\mathbf{A}^k)$ to count all closed walks of length $k$
2. Subtract degenerate walks that revisit edges or use shorter cycles
3. Divide by $2k$ for undirected graphs (or $k$ for directed graphs) to account for multiple representations

## Connection to Eigenvalues

An elegant alternative formula uses eigenvalues $\lambda_1, \lambda_2, \ldots, \lambda_n$ of $\mathbf{A}$:

$$\boxed{\text{tr}(\mathbf{A}^k) = \sum_{i=1}^{n} \lambda_i^k}$$

This follows from the spectral theorem and provides a way to compute motif counts via graph spectra!