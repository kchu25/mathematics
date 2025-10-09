@def title = "Common Closed Walks Across Multiple Graphs"
@def published = "9 October 2025"
@def tags = ["linear-algebra", "graphs"]

# Common Closed Walks Across Multiple Graphs

## Problem Setup

You have multiple graphs $G_1, G_2, \ldots, G_m$ on the **same vertex set** $V = \{1, 2, \ldots, n\}$. You want to find closed walks that exist in **all** graphs simultaneously—that is, walks where every edge in the walk is present in every graph.

## The Key Insight: Element-wise Product

A closed walk of length $k$ exists in **all** graphs if and only if every edge in the walk exists in every graph. This is captured by the **element-wise (Hadamard) product**:

$$\mathbf{A}_{\text{common}} = \mathbf{A}_1 \odot \mathbf{A}_2 \odot \cdots \odot \mathbf{A}_m$$

where:
$$[{\mathbf{A}_{\text{common}}}]_{ij} = \begin{cases} 
1 & \text{if edge } (i,j) \text{ exists in ALL graphs } G_1, \ldots, G_m \\
0 & \text{otherwise}
\end{cases}$$

Then common closed walks are counted by:

$$\boxed{\text{Common closed walks of length } k = \text{tr}(\mathbf{A}_{\text{common}}^k)}$$

## Wait... If Each Graph is Complete?

If **each individual graph is complete** (every $G_i$ is $K_n$), this becomes a special case. Then each adjacency matrix is:

$\mathbf{A}_i = \mathbf{J} - \mathbf{I}$

where $\mathbf{J}$ is the all-ones matrix and $\mathbf{I}$ is the identity.

The Hadamard product of complete graphs is:

$\mathbf{A}_{\text{common}} = (\mathbf{J} - \mathbf{I}) \odot (\mathbf{J} - \mathbf{I}) \odot \cdots \odot (\mathbf{J} - \mathbf{I}) = \mathbf{J} - \mathbf{I}$

So the **common graph is also complete**! But for general graphs, $\mathbf{A}_{\text{common}}$ can be any graph that is a subgraph of each $G_i$.

## General Properties

### The Common Graph Structure

The common graph $G_{\cap} = (V, E_{\cap})$ has edge set:

$E_{\cap} = E_1 \cap E_2 \cap \cdots \cap E_m$

This is the **most restrictive** graph—it only contains edges that appear in all layers.

### Interpreting Common Closed Walks

$[\mathbf{A}_{\cap}^k]_{ii} = \text{closed walks of length } k \text{ from } i \text{ using only common edges}$

These walks are "robust" in the sense that they exist regardless of which graph layer you're observing.

### Relationship to Individual Graphs

Since $E_{\cap} \subseteq E_i$ for all $i$:

$\mathbf{A}_{\cap} \leq \mathbf{A}_i \quad \text{(element-wise)}$

Therefore:
$\text{tr}(\mathbf{A}_{\cap}^k) \leq \text{tr}(\mathbf{A}_i^k) \quad \text{for all } i$

The number of common closed walks is **at most** the number in any individual graph.

## More Interesting Case: Different Vertex Sets

Perhaps you meant each graph is on a **different vertex set** but you want to find common **patterns**?

### Setup
- Graph $G_1$ on vertices $V_1$ with $|V_1| = n_1$
- Graph $G_2$ on vertices $V_2$ with $|V_2| = n_2$
- Each is complete within its vertex set

### Interpretation 1: Graph Union

If you consider the **union** of these graphs (disjoint vertex sets), you get disconnected complete components. Common closed walks would only exist within each component.

### Interpretation 2: Labeled Pattern Matching

You want to find **isomorphic closed walks** across graphs. For example, if both graphs contain triangles (which all complete graphs do), you're asking how many triangles exist in each.

For complete graph $K_n$, the number of each motif is:
- **Triangles**: $\binom{n}{3}$
- **4-cycles**: $\binom{n}{4} \times 3$ (choose 4 vertices, then 3 ways to form a 4-cycle)
- **$k$-cycles**: depends on $k$ and $n$

## Practical Example: Three General Graphs

Consider three graphs on vertices $\{1, 2, 3, 4\}$:

**Graph 1**: Triangle with tail
```
  1---2
  |   |
  3---4
```
$\mathbf{A}_1 = \begin{pmatrix} 
0 & 1 & 1 & 0 \\
1 & 0 & 0 & 1 \\
1 & 0 & 0 & 1 \\
0 & 1 & 1 & 0
\end{pmatrix}$

**Graph 2**: 4-cycle
```
  1---2
  |   |
  4---3
```
$\mathbf{A}_2 = \begin{pmatrix} 
0 & 1 & 0 & 1 \\
1 & 0 & 1 & 0 \\
0 & 1 & 0 & 1 \\
1 & 0 & 1 & 0
\end{pmatrix}$

**Graph 3**: Star from vertex 1
```
    2
    |
  1-+-4
    |
    3
```
$\mathbf{A}_3 = \begin{pmatrix} 
0 & 1 & 1 & 1 \\
1 & 0 & 0 & 0 \\
1 & 0 & 0 & 0 \\
1 & 0 & 0 & 0
\end{pmatrix}$

**Common edges** (intersection):

Edge $(1,2)$: ✓ in all three
Edge $(1,3)$: ✓ in $G_1$ and $G_3$, ✗ in $G_2$
Edge $(1,4)$: ✗ in $G_1$, ✓ in $G_2$ and $G_3$

Only edge $(1,2)$ is common to all!

$\mathbf{A}_{\cap} = \begin{pmatrix} 
0 & 1 & 0 & 0 \\
1 & 0 & 0 & 0 \\
0 & 0 & 0 & 0 \\
0 & 0 & 0 & 0
\end{pmatrix}$

**Common closed walks**:

For $k=2$:
$\mathbf{A}_{\cap}^2 = \begin{pmatrix} 
1 & 0 & 0 & 0 \\
0 & 1 & 0 & 0 \\
0 & 0 & 0 & 0 \\
0 & 0 & 0 & 0
\end{pmatrix}$

$\text{tr}(\mathbf{A}_{\cap}^2) = 2$

These are the walks $1 \to 2 \to 1$ and $2 \to 1 \to 2$.

For $k=3$:
$\text{tr}(\mathbf{A}_{\cap}^3) = 0$

No triangles in the common graph (it's just a single edge).

For $k=4$:
$\text{tr}(\mathbf{A}_{\cap}^4) = 2$

Same as $k=2$ since we just go back and forth twice.

## Applications and Interpretations

### Multilayer Networks

In **multilayer network** analysis, each graph represents a different type of relationship:
- Layer 1: Co-authorship network
- Layer 2: Citation network  
- Layer 3: Collaboration network

Common closed walks represent **robust patterns** that exist across all relationship types.

### Temporal Networks

If graphs represent **snapshots over time**:
- $G_1$: Network at time $t_1$
- $G_2$: Network at time $t_2$
- $G_3$: Network at time $t_3$

Common closed walks are **persistent structures** that exist throughout the time period.

### Consensus/Aggregation

The common graph $G_{\cap}$ represents the **consensus structure**—edges that all observers agree exist. This is conservative compared to union $G_{\cup}$ which includes any edge reported by at least one observer.

## Computing Efficiency

### Direct Computation

1. Compute $\mathbf{A}_{\cap} = \mathbf{A}_1 \odot \mathbf{A}_2 \odot \cdots \odot \mathbf{A}_m$ (element-wise AND)
2. Compute $\mathbf{A}_{\cap}^k$ via repeated matrix multiplication
3. Sum diagonal: $\text{tr}(\mathbf{A}_{\cap}^k)$

**Complexity**: $O(m \cdot n^2)$ for intersection + $O(k \cdot n^3)$ for powering (or $O(n^3 \log k)$ via matrix exponentiation)

### Spectral Method

If $\mathbf{A}_{\cap}$ has eigenvalues $\lambda_1, \ldots, \lambda_n$:

$\text{tr}(\mathbf{A}_{\cap}^k) = \sum_{i=1}^n \lambda_i^k$

**Complexity**: $O(n^3)$ for eigendecomposition (one time), then $O(n)$ per query for different $k$

This is **much faster** if you need counts for many different walk lengths!

## Advanced: Weighted Common Walks

If edges have weights/probabilities in each graph, use:

$\mathbf{W}_{\cap} = \mathbf{W}_1 \odot \mathbf{W}_2 \odot \cdots \odot \mathbf{W}_m$

where $[{\mathbf{W}_{\cap}}]_{ij} = \prod_{\ell=1}^m [\mathbf{W}_{\ell}]_{ij}$

Then $[\mathbf{W}_{\cap}^k]_{ii}$ counts weighted closed walks with weight equal to the product of edge weights across all layers along the walk.

## Summary Table

| Concept | Formula | Interpretation |
|---------|---------|----------------|
| Common adjacency | $\mathbf{A}_{\cap} = \mathbf{A}_1 \odot \cdots \odot \mathbf{A}_m$ | Edges in ALL graphs |
| Common walks of length $k$ | $\text{tr}(\mathbf{A}_{\cap}^k)$ | Closed walks using only common edges |
| Spectral form | $\sum_{i=1}^n \lambda_i^k$ | Using eigenvalues of $\mathbf{A}_{\cap}$ |
| Common triangles | $\frac{\text{tr}(\mathbf{A}_{\cap}^3)}{6}$ | Triangles present in all graphs |
| Bound | $\text{tr}(\mathbf{A}_{\cap}^k) \leq \min_i \text{tr}(\mathbf{A}_i^k)$ | At most the minimum across graphs |

**Key insight**: The Hadamard product $\odot$ implements edge intersection, converting a multilayer/temporal problem into a single-graph motif counting problem!