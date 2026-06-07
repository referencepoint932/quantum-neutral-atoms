# Lesson 03 — Kernel Ridge Regression & SVM

**Goal:** fit the two learners that consume a kernel — kernel ridge regression (KRR, closed-form in PyTorch) and the support vector machine (SVM) — and see that swapping the kernel is the *only* change needed to go quantum.

**QML tie-in:** Pasqal's Quantum Evolution Kernel "integrates quantum kernels into existing kernel-based algorithms like support vector machines and kernel ridge regression." This lesson builds exactly those two algorithms against a Gram matrix, so the QEK hand-off is: produce K with a quantum device → call the code below.

---

## 1. Kernel ridge regression in closed form

Ridge regression in feature space has a clean closed-form solution that touches the data *only* through the kernel matrix K:

> α = (K + λI)⁻¹ y, and predictions are ŷ(x) = Σᵢ αᵢ k(x, xᵢ)

λ (lambda) is the regularization strength. No gradient descent needed — just a linear solve. This makes KRR the cleanest way to test a new kernel.

```python
import torch

def rbf_kernel(A, B, sigma=1.0):
    return torch.exp(-(torch.cdist(A, B) ** 2) / (2 * sigma**2))

torch.manual_seed(0)

# regression target: a noisy sine
Xtr = torch.linspace(-3, 3, 60).unsqueeze(1)
ytr = torch.sin(Xtr).squeeze(1) + 0.1 * torch.randn(60)
Xte = torch.linspace(-3, 3, 200).unsqueeze(1)

# --- fit ---
sigma, lam = 0.7, 1e-2
K = rbf_kernel(Xtr, Xtr, sigma)                      # (60, 60)
I = torch.eye(K.shape[0])
alpha = torch.linalg.solve(K + lam * I, ytr)         # solve (K + lambda I) a = y

# --- predict ---
K_te = rbf_kernel(Xte, Xtr, sigma)                   # (200, 60)
y_pred = K_te @ alpha

# report fit quality on training points
train_pred = K @ alpha
mse = ((train_pred - ytr) ** 2).mean()
print(f"train MSE: {mse.item():.4f}")
print(f"sample predictions near x=0: {y_pred[95:105].round(decimals=3).tolist()}")
```

The entire learning step is one `torch.linalg.solve`. To make it a *quantum* KRR, you replace the two `rbf_kernel(...)` calls with quantum-kernel evaluations and change nothing else.

---

## 2. The role of λ (regularization preview)

λ controls the bias–variance trade-off: small λ fits the training data hard (risking overfitting), large λ smooths. This is your first taste of Lesson 05.

```python
import torch

def rbf_kernel(A, B, sigma=1.0):
    return torch.exp(-(torch.cdist(A, B) ** 2) / (2 * sigma**2))

torch.manual_seed(0)
Xtr = torch.linspace(-3, 3, 60).unsqueeze(1)
ytr = torch.sin(Xtr).squeeze(1) + 0.1 * torch.randn(60)
K = rbf_kernel(Xtr, Xtr, 0.7)
I = torch.eye(K.shape[0])

for lam in [1e-4, 1e-2, 1.0, 100.0]:
    alpha = torch.linalg.solve(K + lam * I, ytr)
    train_mse = (((K @ alpha) - ytr) ** 2).mean().item()
    print(f"lambda={lam:>7}:  train MSE={train_mse:.4f}  |alpha|max={alpha.abs().max():.2f}")
```

You'll see tiny λ drives train MSE → 0 but blows up the coefficients (overfitting); large λ underfits. The "right" λ is chosen by cross-validation, not by training error.

---

## 3. SVM with a precomputed kernel

The SVM is the classic kernel classifier (maximum-margin). Implementing the full quadratic-program solver from scratch is fiddly, so use scikit-learn with a **precomputed** Gram matrix — the realistic QEK workflow, where the kernel comes from elsewhere (a quantum device) and sklearn just does the classical optimization.

```python
import torch
import numpy as np
from sklearn.svm import SVC

def rbf_kernel(A, B, sigma=1.0):
    return torch.exp(-(torch.cdist(A, B) ** 2) / (2 * sigma**2))

torch.manual_seed(0)
# two interleaving blobs
Xtr = torch.cat([torch.randn(40, 2) + 1.5, torch.randn(40, 2) - 1.5])
ytr = torch.cat([torch.zeros(40), torch.ones(40)]).long()
Xte = torch.cat([torch.randn(20, 2) + 1.5, torch.randn(20, 2) - 1.5])
yte = torch.cat([torch.zeros(20), torch.ones(20)]).long()

sigma = 1.5
K_tr = rbf_kernel(Xtr, Xtr, sigma).numpy()       # (80, 80) train-train
K_te = rbf_kernel(Xte, Xtr, sigma).numpy()       # (40, 80) test-train

clf = SVC(kernel="precomputed", C=1.0)           # <-- key: precomputed Gram matrix
clf.fit(K_tr, ytr.numpy())
acc = (clf.predict(K_te) == yte.numpy()).mean()
print(f"SVM test accuracy: {acc:.2%}")
```

`kernel="precomputed"` is the entire integration point. QEK produces `K_tr` and `K_te` from a Rydberg processor; this code is unchanged. Pasqal used exactly this pattern (a quantum graph kernel → classical kernel classifier) to predict molecular toxicity on neutral-atom hardware.

---

## 4. The big picture

```
            classical                                   quantum (Pasqal)
  data ──► φ (RBF/poly) ──► Gram matrix K ──► KRR/SVM     data(graph) ──► Rydberg evolution ──► K ──► KRR/SVM
                                   └──────────────── same downstream code ───────────────┘
```

The learner is kernel-agnostic. That is *why* quantum kernels are an attractive near-term QML strategy: the quantum computer only has to produce a good K; decades of classical kernel machinery does the rest.

---

## Exercises (fill in)

1. Wrap the KRR fit in k-fold cross-validation to pick `(sigma, lambda)`. Report test MSE for the chosen pair vs. a deliberately bad pair.
2. For the SVM, sweep `C` (0.01 → 100) and plot train vs. test accuracy. Where does overfitting begin?
3. Replace the RBF Gram matrix with a **random PSD matrix** of the same size and watch accuracy collapse — a reminder that the kernel must encode real structure, which is the whole burden a quantum kernel must carry.

> Notes:
>
>
