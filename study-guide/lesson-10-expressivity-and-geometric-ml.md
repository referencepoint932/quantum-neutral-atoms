# Lesson 10 — Expressivity & Geometric/Equivariant ML

**Goal:** understand what determines a model's expressive power (the Fourier view of quantum models) and how building in **symmetry** buys both trainability and generalization.

**QML tie-in:** The expressivity–trainability trade-off (Lessons 05/06) is resolved by *structure*. Geometric/equivariant QML — encoding problem symmetries into the circuit or kernel — is a leading route to quantum advantage without exponential concentration (`arXiv:2509.14337`). Graph permutation symmetry (Lesson 04) is the canonical example, directly relevant to QEK.

Prereqs: Lessons 04 (graph symmetry), 06 (feature maps), 09 (variational circuits).

---

## 1. Expressivity = the function class a model can represent

More parameters/depth → richer functions, but also harder training and worse generalization on small data. Demonstrate with polynomial degree as an expressivity dial (revisiting Lesson 05 from the representation angle).

```python
import torch

torch.manual_seed(0)
x = torch.linspace(-1, 1, 200)
target = torch.sin(5*x)   # needs some expressivity to fit

for degree in [1, 3, 7, 15]:
    Phi = torch.stack([x**d for d in range(degree+1)], dim=1)
    coef = torch.linalg.lstsq(Phi, target.unsqueeze(1)).solution
    fit_err = ((Phi @ coef).squeeze() - target).pow(2).mean()
    print(f"degree {degree:2d}: fit error {fit_err.item():.4f}")
```

Higher degree fits the wiggly target better — that's raw expressivity. The trick in QML is getting expressivity *where you need it* without making the whole model untrainable.

---

## 2. Quantum models are Fourier series

A key theoretical handle: a variational circuit whose data-encoding gates are `Ry(x)`-type rotations outputs a **truncated Fourier series in x**, with frequencies set by the encoding and coefficients set by the trainable parameters. Simulate this fact.

```python
import torch

# A simple data-encoding model's output is sum_k a_k cos(k x) + b_k sin(k x).
# Demonstrate by fitting arbitrary Fourier coefficients to data.
torch.manual_seed(0)
x = torch.linspace(-torch.pi, torch.pi, 300)
target = torch.sign(torch.sin(x))     # square wave: rich in odd harmonics

def fourier_features(x, K):
    feats = [torch.ones_like(x)]
    for k in range(1, K+1):
        feats += [torch.cos(k*x), torch.sin(k*x)]
    return torch.stack(feats, dim=1)

for K in [1, 3, 9]:                    # K = number of accessible frequencies
    Phi = fourier_features(x, K)
    coef = torch.linalg.lstsq(Phi, target.unsqueeze(1)).solution
    err = ((Phi @ coef).squeeze() - target).pow(2).mean()
    print(f"max frequency K={K}: fit error {err.item():.4f}")
```

The number of qubits and the encoding scheme set `K` (the available frequencies); the variational parameters set the coefficients. This is *the* mental model for "what can a quantum circuit learn" — and it tells you that encoding design, not just depth, governs expressivity.

---

## 3. Symmetry: the antidote to over-expressivity

If you know your problem is invariant under some transformation (e.g., relabeling graph nodes, rotating an image), build that invariance into the model. The model then *can't* waste capacity on symmetry-violating functions — smaller effective hypothesis space, better generalization, fewer barren plateaus.

**Invariant feature** demo: make a representation that ignores node ordering.

```python
import torch

# two graphs that are the SAME graph with nodes permuted
A = torch.tensor([[0.,1,1],[1,0,0],[1,0,0]])
P = torch.tensor([[0.,0,1],[1,0,0],[0,1,0]])     # a permutation matrix
A_perm = P @ A @ P.T

# NON-invariant feature: flattened adjacency -> differs under permutation
print("flattened equal? ", torch.allclose(A.flatten(), A_perm.flatten()))   # False

# INVARIANT features: sorted degree sequence, trace of A^2, etc.
def invariants(A):
    deg = torch.sort(A.sum(1)).values
    return torch.cat([deg, torch.tensor([torch.trace(A@A), A.sum()])])
print("invariants equal?", torch.allclose(invariants(A), invariants(A_perm)))  # True
```

The invariant features are identical for the two orderings — exactly the permutation invariance a graph kernel (WL, QEK) must have. **Equivariant** models generalize this: intermediate layers transform *consistently* with the symmetry, and only the final readout is invariant.

---

## 4. Why this matters for Pasqal QML

- **Graph tasks (QEK):** permutation symmetry is built into the kernel by construction — that's a large part of why QEK is a sensible graph method.
- **Avoiding concentration:** symmetry-structured kernels can retain a quantum advantage *without* the exponential concentration that kills generic quantum kernels (`arXiv:2509.14337`). Structure beats brute-force expressivity.
- **Hardware-native symmetry:** neutral-atom registers have geometric symmetries (atom arrangements) that can be matched to problem symmetries.

The recurring theme of the whole guide, stated sharply: **expressivity is cheap; useful, trainable expressivity comes from encoding the right structure.**

---

## Exercises (fill in)

1. In §2, relate `K` to qubit count for a product `Ry(x)` encoding (hint: frequencies add). Confirm empirically that more "qubits" → higher `K`.
2. Build a permutation-*equivariant* aggregation (sum over neighbors) and show its output permutes consistently with node relabeling.
3. Compare a symmetry-agnostic vs. symmetry-aware classifier on the graph data from Lesson 04; measure the generalization gap on held-out graphs.

> Notes:
>
