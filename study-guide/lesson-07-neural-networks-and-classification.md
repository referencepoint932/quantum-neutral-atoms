# Lesson 07 — Neural Networks & Classification

**Goal:** build and train a real (multi-layer) classifier in PyTorch — layers, nonlinearities, softmax + cross-entropy — and see the structure that variational quantum models mimic.

**QML tie-in:** A variational quantum circuit is structurally a neural network: parameterized layers, a nonlinearity (measurement), a loss, gradient training. The "data re-uploading" QML technique is literally interleaving data-encoding and trainable layers — the same alternation you build here. Comfort with `nn.Module` and cross-entropy transfers straight to hybrid quantum-classical models (Lesson 12).

Prereqs: Lesson 01 (autograd, optim).

---

## 1. From linear to nonlinear: why layers + activations

A stack of linear layers with no nonlinearity collapses to a single linear layer. The **activation** (ReLU, tanh) is what lets a network represent curved decision boundaries — the classical counterpart of a quantum circuit's nonlinearity coming from measurement.

```python
import torch
import torch.nn as nn

# two stacked linears with NO activation == one linear (no added power)
a = nn.Linear(2, 8); b = nn.Linear(8, 2)
x = torch.randn(5, 2)
collapsed = b(a(x))                       # still an affine function of x
print("output shape:", collapsed.shape)   # (5, 2) -- but representationally linear
```

---

## 2. A multi-class classifier

The standard recipe: linear → activation → linear → logits, trained with cross-entropy (which folds in softmax). We classify three Gaussian blobs.

```python
import torch
import torch.nn as nn

torch.manual_seed(0)

# 3-class toy data
def blobs(n, center):
    return torch.randn(n, 2) + torch.tensor(center)
X = torch.cat([blobs(100,[0,3]), blobs(100,[-3,-3]), blobs(100,[3,-3])])
y = torch.cat([torch.zeros(100), torch.ones(100), 2*torch.ones(100)]).long()

model = nn.Sequential(
    nn.Linear(2, 32), nn.ReLU(),
    nn.Linear(32, 32), nn.ReLU(),
    nn.Linear(32, 3),                 # 3 logits, one per class
)
opt = torch.optim.Adam(model.parameters(), lr=0.01)
loss_fn = nn.CrossEntropyLoss()       # expects raw logits + integer labels

for epoch in range(300):
    opt.zero_grad()
    logits = model(X)
    loss = loss_fn(logits, y)
    loss.backward()
    opt.step()
    if epoch % 100 == 0:
        acc = (logits.argmax(1) == y).float().mean()
        print(f"epoch {epoch:3d}  loss {loss.item():.3f}  acc {acc.item():.2%}")

print("final train accuracy:", (model(X).argmax(1) == y).float().mean().item())
```

Key objects you'll reuse everywhere, including QML:

- **Logits → probabilities** via softmax (inside `CrossEntropyLoss`). In QML, measurement probabilities play the role of softmax outputs.
- **Cross-entropy** is the default classification loss; it's the negative log-likelihood of the correct class (ties to Lesson 08).
- **`argmax`** turns probabilities into a hard prediction.

---

## 3. Train/validation split — generalization in practice

Reusing Lesson 05's lesson: judge the model on held-out data.

```python
import torch

torch.manual_seed(0)
n = X.shape[0]
perm = torch.randperm(n)
cut = int(0.8 * n)
tr, va = perm[:cut], perm[cut:]

# (re-init + train on X[tr], y[tr] using the block above, then:)
# val_acc = (model(X[va]).argmax(1) == y[va]).float().mean()
print("hold out", len(va), "samples for validation")   # 60
```

---

## 4. The bridge to variational quantum models

| Neural network | Variational quantum circuit |
|----------------|-----------------------------|
| Input features `x` | Data-encoding gates (feature map) |
| Trainable weights `W, b` | Trainable rotation angles `θ` |
| ReLU/tanh nonlinearity | Measurement (Born rule) |
| Stacked layers | Alternating encode/entangle layers |
| Softmax + cross-entropy | Measurement probabilities + loss |
| Backprop | Autodiff / **parameter-shift** (Lesson 09) |
| Data re-input (rare) | **Data re-uploading** (common, boosts expressivity) |

The training loop is *identical*. That's why Pasqal's Qadence integrates with PyTorch: a quantum model is a drop-in `nn.Module`-like object.

---

## Exercises (fill in)

1. Remove all activations and retrain — confirm accuracy collapses toward a linear classifier's limit on non-linearly-separable data.
2. Replace `ReLU` with `Tanh` (the activation closest to quantum rotations) and compare convergence.
3. Add `weight_decay` and a validation split; find the setting with the best val accuracy (connects Lessons 05 + 07).

> Notes:
>
