# Lesson 04 — Graph Data & Graph Kernels

**Goal:** represent graph-structured data in PyTorch and build a classical graph kernel (Weisfeiler–Lehman), establishing the baseline that Pasqal's **Quantum Evolution Kernel (QEK)** must beat.

**QML tie-in:** QEK is a *graph* ML method — it encodes graph topology into a Rydberg Hamiltonian and reads out a similarity. To judge whether the quantum kernel is worth it, you need a strong classical graph kernel for comparison. The downstream learner is identical to Lesson 03 (`SVC(kernel="precomputed")`), so this lesson is really "produce a graph Gram matrix and reuse everything."

Prereqs: Lessons 02 (Gram matrices) and 03 (kernel learners).

---

## 1. Representing a graph

A graph is nodes + edges. In tensors: an **adjacency matrix** `A` (who connects to whom) and optionally **node features/labels**.

```python
import torch

# a 4-node "path" graph: 0-1-2-3
A = torch.tensor([
    [0,1,0,0],
    [1,0,1,0],
    [0,1,0,1],
    [0,0,1,0],
], dtype=torch.float)

print("degrees (row sums):", A.sum(1).tolist())   # [1, 2, 2, 1]
print("symmetric (undirected):", torch.allclose(A, A.T))
```

Molecules are the canonical example: atoms are nodes (with a chemical-element label), bonds are edges. This is exactly the data type behind Pasqal's toxicity-prediction work.

---

## 2. Why comparing graphs is hard

Two graphs can be identical but with nodes listed in a different order — there's no canonical ordering. A good graph similarity must be **permutation invariant**: relabeling nodes shouldn't change the answer. This is the same symmetry concern that motivates *equivariant* models (Lesson 10).

---

## 3. The Weisfeiler–Lehman (WL) kernel

WL is the standard, strong, cheap baseline. Intuition: repeatedly refine each node's label by hashing together its own label and its neighbors' labels. After a few rounds, two graphs that share many such "labels" are similar. The kernel = inner product of label-count histograms.

```python
import torch
from collections import Counter

def wl_histograms(A, node_labels, n_iter=2):
    """Return a Counter of WL labels seen across iterations for one graph."""
    labels = list(node_labels)
    hist = Counter(labels)
    n = A.shape[0]
    neighbors = [torch.nonzero(A[i]).flatten().tolist() for i in range(n)]
    for _ in range(n_iter):
        new = []
        for i in range(n):
            neigh = sorted(labels[j] for j in neighbors[i])
            new.append(f"{labels[i]}|{','.join(map(str, neigh))}")
        # compress long strings to integers for compactness
        labels = new
        hist.update(labels)
    return hist

def wl_kernel(g1, g2, n_iter=2):
    h1 = wl_histograms(*g1, n_iter)
    h2 = wl_histograms(*g2, n_iter)
    keys = set(h1) | set(h2)
    return sum(h1[k] * h2[k] for k in keys)   # histogram inner product
```

Build a Gram matrix over a small dataset of graphs:

```python
import torch

# three triangles and three paths; (adjacency, node_labels)
def triangle():
    A = torch.tensor([[0,1,1],[1,0,1],[1,1,0]], dtype=torch.float)
    return (A, ["C","C","C"])
def path():
    A = torch.tensor([[0,1,0],[1,0,1],[0,1,0]], dtype=torch.float)
    return (A, ["C","C","C"])

graphs = [triangle(), triangle(), triangle(), path(), path(), path()]
y = [0,0,0,1,1,1]                       # 0 = triangle, 1 = path

n = len(graphs)
K = torch.zeros(n, n)
for i in range(n):
    for j in range(n):
        K[i, j] = wl_kernel(graphs[i], graphs[j])
print(K)
# triangles should be more similar to triangles than to paths -> block structure
```

You'll see a clear 3×3 block structure: same-class graphs score higher. That block structure *is* a learnable signal.

---

## 4. Plug into the Lesson 03 learner — unchanged

```python
import numpy as np
from sklearn.svm import SVC

K_np = K.numpy()
# leave-one-out style: train on 4, test on 2 (one per class) for illustration
train_idx = [0,1,3,4]; test_idx = [2,5]
clf = SVC(kernel="precomputed", C=1.0)
clf.fit(K_np[np.ix_(train_idx, train_idx)], [y[i] for i in train_idx])
pred = clf.predict(K_np[np.ix_(test_idx, train_idx)])
print("graph SVM predictions:", pred.tolist(), "true:", [y[i] for i in test_idx])
```

The classifier code is byte-for-byte the same as Lesson 03. Only the kernel changed — from RBF on vectors to WL on graphs.

---

## 5. Bridge to QEK

QEK replaces the WL histogram with quantum dynamics:

```
WL kernel:   graph ──► label-refinement histograms ──► histogram inner product ──► K
QEK:         graph ──► encode topology in Rydberg Hamiltonian ──► time-evolve ──► sample distribution ──► distribution distance ──► K
```

Both are permutation-aware graph kernels feeding the same SVM/KRR. QEK's bet is that the Rydberg quantum dynamics capture graph structure that classical kernels miss — and the **local-detuning** extension (`arXiv:2509.09421`) lets *node attributes*, not just topology, enter the Hamiltonian, the quantum analogue of WL's node labels.

References: `arXiv:2107.03247` (original QEK), `arXiv:2509.09421` (attributed-graph kernels via local detuning), Pasqal's QEK library docs.

---

## Exercises (fill in)

1. Add graphs of varying size (4–6 nodes) and confirm WL still works (the kernel handles different node counts naturally).
2. Vary `n_iter` from 0 to 4. How does discriminative power change? (n_iter=0 is just a node-label-count kernel.)
3. Make two *non-isomorphic* graphs that WL cannot distinguish (a known WL limitation). This is precisely the gap a quantum kernel might exploit.

> Notes:
>
