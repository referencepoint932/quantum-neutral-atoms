# Lesson 02 — Feature Maps & Kernels

**Goal:** understand feature maps, the kernel trick, and how to build a Gram (kernel) matrix in PyTorch. This is the conceptual core that the **Quantum Evolution Kernel (QEK)** slots into.

**QML tie-in:** A quantum kernel is *literally* a feature map into a Hilbert space (the quantum state space) followed by an inner product. Pasqal's QEK encodes a graph into a Rydberg Hamiltonian, evolves the state, and reads out a similarity — that similarity is a kernel value. Everything you learn here about classical kernels transfers directly; only the feature map becomes quantum.

---

## 1. The idea: make hard problems linear by lifting

Some data isn't linearly separable in its raw coordinates but becomes separable after mapping into a higher-dimensional space. The map φ is the **feature map**.

```python
import torch

# 1D data that is NOT linearly separable: inner points one class, outer another
x = torch.tensor([-2.0, -1.5, -0.5, 0.5, 1.5, 2.0])
label = torch.tensor([1, 1, 0, 0, 1, 1])     # 0 = inner, 1 = outer

# feature map phi(x) = [x, x^2]  -> now the classes ARE linearly separable in x^2
phi = torch.stack([x, x**2], dim=1)
print(phi)
# the x^2 coordinate alone (col 1) separates inner (small) from outer (large)
```

The catch: useful feature maps can be enormous (even infinite-dimensional). Forming φ(x) explicitly is often impossible.

---

## 2. The kernel trick

A **kernel** computes the inner product in feature space *without ever building the feature vectors*:

> k(x, x′) = ⟨ φ(x), φ(x′) ⟩

Many algorithms (SVM, ridge regression, PCA) only ever touch the data through inner products. Swap those inner products for k(x, x′) and you operate in feature space for free. This is the trick that makes a quantum kernel usable: you never materialize the exponentially large state vector, you just need overlaps.

---

## 3. Building Gram matrices in PyTorch

The **Gram matrix** K is all pairwise kernel values: `K[i,j] = k(x_i, x_j)`. It's the single object every kernel method consumes.

### Linear kernel

```python
import torch

torch.manual_seed(0)
X = torch.randn(5, 3)                 # 5 samples, 3 features

K_linear = X @ X.T                    # k(x,x') = <x, x'>
print(K_linear.shape)                 # (5, 5)
print(torch.allclose(K_linear, K_linear.T))   # kernels are symmetric -> True
```

### RBF (Gaussian) kernel — the workhorse

`k(x, x′) = exp(-‖x − x′‖² / (2σ²))`. Note it's an inner product in an *infinite*-dimensional feature space, yet trivial to compute.

```python
import torch

def rbf_kernel(A, B, sigma=1.0):
    # squared Euclidean distances between every row of A and every row of B
    sq = torch.cdist(A, B) ** 2          # shape (len(A), len(B))
    return torch.exp(-sq / (2 * sigma**2))

torch.manual_seed(0)
X = torch.randn(5, 3)
K = rbf_kernel(X, X, sigma=1.0)
print(K)
print("diagonal is 1 (a point is identical to itself):", torch.diag(K))
```

Read the two structural facts off the matrix, because they decide whether *any* kernel — classical or quantum — is useful:

- **Symmetric:** `K == K.T`.
- **Diagonal = self-similarity.** If *off*-diagonal entries all collapse toward zero (every point looks dissimilar to every other), the kernel is uninformative. This is exactly the **exponential concentration** failure mode of quantum kernels — the Gram matrix drifts toward the identity and learning fails. You'll revisit this in Lesson 06.

---

## 4. A kernel is an interface, not an algorithm

The payoff: define a feature map → get a kernel → hand the Gram matrix to a learner. The learner doesn't care where K came from. Here's a tiny **kernel nearest-centroid** classifier to make the interface concrete:

```python
import torch

def rbf_kernel(A, B, sigma=1.0):
    return torch.exp(-(torch.cdist(A, B) ** 2) / (2 * sigma**2))

torch.manual_seed(0)
# two gaussian blobs
Xtr = torch.cat([torch.randn(20, 2) + 2, torch.randn(20, 2) - 2])
ytr = torch.cat([torch.zeros(20), torch.ones(20)]).long()
Xte = torch.tensor([[2.0, 2.0], [-2.0, -2.0]])

# classify by which class's points are, on average, most similar (kernel) to the test point
K = rbf_kernel(Xte, Xtr, sigma=1.5)          # (2 test, 40 train)
score_class0 = K[:, ytr == 0].mean(dim=1)
score_class1 = K[:, ytr == 1].mean(dim=1)
pred = (score_class1 > score_class0).long()
print("predictions:", pred.tolist())          # expect [0, 1]
```

To switch to a *quantum* kernel you would replace `rbf_kernel` with a function that returns quantum-state overlaps (QEK's role) — and this classifier would not change a single line. That swap-ability is precisely why Pasqal ships QEK as a kernel that plugs into existing kernel-based algorithms.

---

## Exercises (fill in)

1. Sweep `sigma` in the RBF kernel from 0.1 to 10 and print the mean off-diagonal value of K. Watch it concentrate (toward 0 or toward 1) at the extremes — a classical preview of kernel concentration.
2. Implement a **polynomial kernel** `k(x,x') = (<x,x'> + c)^d` and verify for `d=2` it matches an explicit feature map you build by hand.
3. Confirm any Gram matrix you build is positive semi-definite (`torch.linalg.eigvalsh(K).min() >= -1e-6`). Why must a valid kernel be PSD?

> Notes:
>
>
