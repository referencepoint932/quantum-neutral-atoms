# Lesson 01 — Tensors & Autograd

**Goal:** understand tensors and automatic differentiation by training a model with a hand-written gradient-descent loop. This is the exact loop you reuse in variational QML — only the function being differentiated changes.

**QML tie-in:** Qadence trains quantum circuits with PyTorch's autodiff plus parameter-shift rules. The optimizer, the loss, the `.backward()` call, the `step()` — all identical to what you write below. If you're comfortable with this loop, variational QML training is familiar territory.

---

## 1. Tensors

A tensor is an n-dimensional array that can (a) live on GPU and (b) track operations for differentiation.

```python
import torch

# scalars, vectors, matrices are all tensors
a = torch.tensor(3.0)
v = torch.tensor([1.0, 2.0, 3.0])
M = torch.randn(3, 3)          # 3x3 random matrix

print(v @ v)                   # inner product -> tensor(14.)
print(M.shape, M.T.shape)      # transpose
print((M @ v).shape)           # matrix-vector product -> torch.Size([3])
```

Inner products matter here: a kernel value is an inner product (Lesson 02), and a *quantum* kernel is an inner product of state vectors. Same operation, fancier vectors.

---

## 2. Autograd: gradients for free

Set `requires_grad=True` and PyTorch records every operation into a graph. Calling `.backward()` walks that graph backward (the chain rule) and fills in `.grad`.

```python
import torch

x = torch.tensor(2.0, requires_grad=True)
y = x**3 + 2*x          # y = x^3 + 2x  ->  dy/dx = 3x^2 + 2
y.backward()
print(x.grad)           # 3*(2^2) + 2 = 14  -> tensor(14.)
```

That's the whole idea: you write the forward computation, PyTorch gives you the gradient. In QML the forward pass runs a quantum circuit (or its simulation); the gradient comes from autodiff or parameter-shift, but you call it the same way.

---

## 3. A full training loop by hand

Let's fit a line `y = w*x + b` to noisy data, writing the gradient step manually so nothing is hidden.

```python
import torch

torch.manual_seed(0)

# synthetic data: true relationship y = 2x - 1 + noise
X = torch.linspace(-1, 1, 100).unsqueeze(1)        # shape (100, 1)
y = 2 * X - 1 + 0.1 * torch.randn_like(X)

# parameters we will learn
w = torch.zeros(1, requires_grad=True)
b = torch.zeros(1, requires_grad=True)

lr = 0.1
for epoch in range(200):
    y_pred = X @ w + b                  # forward pass
    loss = ((y_pred - y)**2).mean()     # mean squared error

    loss.backward()                     # fills w.grad, b.grad

    with torch.no_grad():               # don't track the update itself
        w -= lr * w.grad
        b -= lr * b.grad
        w.grad.zero_()                  # gradients accumulate; reset them
        b.grad.zero_()

    if epoch % 50 == 0:
        print(f"epoch {epoch:3d}  loss {loss.item():.4f}  w {w.item():.3f}  b {b.item():.3f}")

print(f"learned: w={w.item():.3f}, b={b.item():.3f}  (true: 2.0, -1.0)")
```

Three things to internalize, because they recur in every variational algorithm:

1. **Forward → loss → backward → update** is the universal rhythm.
2. **`zero_()` matters:** gradients accumulate by default. Forgetting to zero them is the most common training bug.
3. **`no_grad()` for the update:** the parameter update is bookkeeping, not part of the differentiated computation.

---

## 4. The same loop with `torch.optim` and `nn`

In practice you don't hand-roll the update. This is the idiomatic version — and the form Qadence-style QML code takes.

```python
import torch
import torch.nn as nn

torch.manual_seed(0)
X = torch.linspace(-1, 1, 100).unsqueeze(1)
y = 2 * X - 1 + 0.1 * torch.randn_like(X)

model = nn.Linear(1, 1)                       # w and b live inside
opt = torch.optim.Adam(model.parameters(), lr=0.05)
loss_fn = nn.MSELoss()

for epoch in range(300):
    opt.zero_grad()
    loss = loss_fn(model(X), y)
    loss.backward()
    opt.step()

w, b = model.weight.item(), model.bias.item()
print(f"learned: w={w:.3f}, b={b:.3f}  (true: 2.0, -1.0)")
```

In a variational QML program, `model` would be a parameterized quantum circuit instead of `nn.Linear`, but `opt.zero_grad()` / `loss.backward()` / `opt.step()` stay exactly the same. That portability is the whole point of Qadence's PyTorch integration.

---

## Exercises (fill in)

1. Change the loss to mean **absolute** error and watch how the gradients and convergence differ.
2. Replace `Adam` with `SGD(lr=...)`; find a learning rate that diverges, and one that's too slow.
3. Add a second input feature and fit a plane. Confirm `model.weight` now has shape `(1, 2)`.

> Notes:
>
> DONE
