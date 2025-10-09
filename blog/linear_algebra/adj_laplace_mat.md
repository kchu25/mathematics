
@def title = "Adjacency and Laplacian Matrices: Intuition & Math"
@def published = "9 October 2025"
@def tags = ["linear-algebra"]

# Adjacency and Laplacian Matrices: Intuition & Math

## Adjacency Matrix × Vector

### The Intuition
When you multiply the adjacency matrix $A$ by a node feature vector $\mathbf{x}$, you're **aggregating information from neighbors**. Each node collects and sums up the values from all nodes connected to it.

### The Math
Let's say you have a graph with adjacency matrix $A$ where $A_{ij} = 1$ if there's an edge from $j$ to $i$, and 0 otherwise.

When you compute $\mathbf{y} = A\mathbf{x}$:

$$y_i = \sum_{j=1}^{n} A_{ij} x_j = \sum_{j \in \mathcal{N}(i)} x_j$$

where $\mathcal{N}(i)$ is the set of neighbors of node $i$.

### Simple Example
Consider a graph: `1 -- 2 -- 3`

$$A = \begin{bmatrix} 0 & 1 & 0 \\ 1 & 0 & 1 \\ 0 & 1 & 0 \end{bmatrix}, \quad \mathbf{x} = \begin{bmatrix} 5 \\ 3 \\ 7 \end{bmatrix}$$

Then:
$$A\mathbf{x} = \begin{bmatrix} 3 \\ 12 \\ 3 \end{bmatrix}$$

- Node 1 aggregates from node 2: gets 3
- Node 2 aggregates from nodes 1 & 3: gets 5 + 7 = 12
- Node 3 aggregates from node 2: gets 3

**Key insight**: Each position in $A\mathbf{x}$ tells you the sum of your neighbors' values!

---

## Graph Laplacian Matrix

### The Intuition
The Laplacian $L = D - A$ (where $D$ is the degree matrix) measures how much a node's value **differs from its neighbors**. It captures the "roughness" or variation across edges.

### The Math
The Laplacian is defined as:

$$L = D - A$$

where $D_{ii} = \deg(i)$ is the degree of node $i$.

When you compute $L\mathbf{x}$:

$$L\mathbf{x} = D\mathbf{x} - A\mathbf{x}$$

$$(L\mathbf{x})_i = d_i x_i - \sum_{j \in \mathcal{N}(i)} x_j = \sum_{j \in \mathcal{N}(i)} (x_i - x_j)$$

This gives you the **difference between a node's value and each neighbor's value**, summed up!

### Smoothness Measure
The **quadratic form** captures total smoothness:

$$\mathbf{x}^T L \mathbf{x} = \frac{1}{2} \sum_{i,j} A_{ij}(x_i - x_j)^2$$

- Small value → nodes and their neighbors have similar values (smooth signal)
- Large value → big differences across edges (rough signal)

### Same Example
With our graph `1 -- 2 -- 3` and $\mathbf{x} = [5, 3, 7]^T$:

$$D = \begin{bmatrix} 1 & 0 & 0 \\ 0 & 2 & 0 \\ 0 & 0 & 1 \end{bmatrix}, \quad L = \begin{bmatrix} 1 & -1 & 0 \\ -1 & 2 & -1 \\ 0 & -1 & 1 \end{bmatrix}$$

$$L\mathbf{x} = \begin{bmatrix} 2 \\ -2 \\ 4 \end{bmatrix}$$

- Node 1: $(5-3) = 2$ → it's 2 units higher than its neighbor
- Node 2: $(3-5) + (3-7) = -2 - 4 = -6$... wait, let me recalculate: $2(3) - 12 = -6$
- Actually: Node 2 has degree 2, value 3, neighbors sum to 12, so: $2(3) - 12 = -6$

Let me fix this:

$$L\mathbf{x} = \begin{bmatrix} 2 \\ -6 \\ 4 \end{bmatrix}$$

- Node 1: difference from average neighbor = positive (higher than neighbors)
- Node 2: difference = negative (lower than neighbors) 
- Node 3: difference = positive (higher than neighbors)

---

## Why This Matters

**GNNs**: Neural networks use $A\mathbf{x}$ (or normalized versions) to let nodes communicate with neighbors!

**Graph Signal Processing**: Laplacian eigenvalues/eigenvectors tell you about graph structure and smooth vs. oscillatory patterns on the graph.

**Spectral Clustering**: Uses Laplacian eigenvectors to find communities!

---

## How These Operations Are Used in Practice

### In Optimization Problems (Objective Functions)

#### Graph Regularization
The Laplacian appears as a **smoothness penalty** in semi-supervised learning:

$\mathcal{L} = \underbrace{\sum_{i \in \text{labeled}} \ell(y_i, \hat{y}_i)}_{\text{supervised loss}} + \underbrace{\lambda \mathbf{x}^T L \mathbf{x}}_{\text{smoothness regularizer}}$

**What this does**: The first term fits labeled data, the second term encourages predictions to be smooth across edges. If two nodes are connected, we want their predicted values $x_i$ and $x_j$ to be similar!

**Interpretation**: Since $\mathbf{x}^T L \mathbf{x} = \sum_{(i,j) \in E} (x_i - x_j)^2$, minimizing this term penalizes large differences between connected nodes. You're assuming **connected nodes should have similar labels** (homophily assumption).

#### Graph Cuts & Clustering
In spectral clustering, we minimize:

$\text{RatioCut}(A_1, \ldots, A_k) = \sum_{i=1}^{k} \frac{\text{cut}(A_i, \bar{A}_i)}{|A_i|}$

This relaxes to solving:

$\min_{\mathbf{x}} \mathbf{x}^T L \mathbf{x} \quad \text{subject to constraints}$

**Interpretation**: Finding the eigenvectors with smallest eigenvalues of $L$ gives you the "smoothest" signals on the graph, which naturally separate communities!

### In Neural Networks (Forward Pass)

#### Graph Convolutional Networks (GCN)
The classic GCN layer does:

$H^{(l+1)} = \sigma\left(\tilde{D}^{-1/2}\tilde{A}\tilde{D}^{-1/2} H^{(l)} W^{(l)}\right)$

where $\tilde{A} = A + I$ (add self-loops), $\tilde{D}$ is the degree matrix of $\tilde{A}$.

**Breaking it down**:
1. $H^{(l)}W^{(l)}$: Transform features with learnable weights
2. $\tilde{A}(\cdot)$: Aggregate from neighbors (including self)
3. $\tilde{D}^{-1/2}(\cdot)\tilde{D}^{-1/2}$: Normalize by degree (symmetric normalization)

**Interpretation**: Each layer, every node collects information from its neighbors, transforms it, and passes it along. After $k$ layers, each node has aggregated information from its $k$-hop neighborhood!

#### Message Passing Neural Networks (MPNN)
More general framework:

$h_i^{(l+1)} = \text{UPDATE}\left(h_i^{(l)}, \sum_{j \in \mathcal{N}(i)} \text{MESSAGE}(h_i^{(l)}, h_j^{(l)}, e_{ij})\right)$

This is essentially:
- Compute messages from neighbors (function of their features)
- Aggregate messages ($\sum$ is the adjacency matrix operation!)
- Update your own representation

**Interpretation**: The adjacency matrix structure determines **who talks to whom**. The neural network learns **what to say** (MESSAGE) and **how to listen** (UPDATE).

#### Graph Attention Networks (GAT)
GAT learns the adjacency structure:

$h_i^{(l+1)} = \sigma\left(\sum_{j \in \mathcal{N}(i)} \alpha_{ij} W h_j^{(l)}\right)$

where $\alpha_{ij}$ are attention weights (learned, not fixed by $A$).

**Interpretation**: Instead of using fixed $A_{ij}$, the network learns **how much to weight each neighbor**. Some neighbors matter more than others for the prediction task!

#### Example: Node Classification Forward Pass

```
Input: Node features X ∈ ℝ^(n×d), Adjacency A
Layer 1: H₁ = ReLU(ÂXW₁)  ← aggregation via Â
Layer 2: H₂ = ReLU(ÂH₁W₂) ← aggregation again
Output: Ŷ = softmax(H₂)
```

At each layer, $\hat{A}$ (normalized adjacency) **mixes features from neighbors**, and $W$ **transforms** the mixed features. The network learns which transformations are useful for the task!

### Key Insight

- **Laplacian in loss**: Acts as a **regularizer** that enforces smoothness assumptions
- **Adjacency in forward pass**: Acts as the **information routing** mechanism that determines how node features flow through the network

Both leverage the same mathematical principle: **connected nodes should influence each other**!