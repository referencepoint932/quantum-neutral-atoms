# Lesson 05 — Generalization & Training Dynamics

**Goal:** make the bias–variance trade-off, regularization, and vanishing gradients concrete in PyTorch — then connect them directly to the central QML failure mode.

**QML tie-in:** Trainability is *the* problem in QML. **Barren plateaus** (gradients that vanish exponentially with system size) and quantum-**kernel concentration** (Gram matrix collapsing to the identity, Lesson 02/06) are two faces of one trade-off: more expressive models are harder to train. A 2025 result proves these are formally equivalent (`arXiv:2501.07433`); for the analog/Pasqal setting, loss-landscape flatness is linked to phases of matter (`arXiv:2506.13865`).

Prereqs: Lesson 01 (training loops), Lesson 03 (λ regularization).

---

## 1. Overfitting vs. underfitting

Fit polynomials of increasing degree to a small noisy sample and watch train error fall while validation error makes a U-turn.

```python
import torch

torch.manual_seed(0)

def make_data(n):
    x = torch.rand(n, 1) * 2 - 1
    y = torch.sin(3 * x) + 0.1 * torch.randn(n, 1)
    return x, y

xtr, ytr = make_data(15)      # small -> easy to overfit
xva, yva = make_data(200)     # validation

def poly_features(x, degree):
    return torch.cat([x**d for d in range(degree + 1)], dim=1)

for degree in [1, 3, 9, 14]:
    Phi = poly_features(xtr, degree)
    # least squares fit
    coef = torch.linalg.lstsq(Phi, ytr).solution
    train_mse = ((Phi @ coef - ytr) ** 2).mean()
    val_mse = ((poly_features(xva, degree) @ coef - yva) ** 2).mean()
    print(f"degree {degree:2d}:  train {train_mse.item():.4f}   val {val_mse.item():.4f}")
```

Expect train error to keep dropping with degree while validation error bottoms out then climbs — the signature of overfitting. The best model is the one that *generalizes*, not the one that fits training data best.

---

## 2. Regularization tames variance

Adding an L2 penalty (ridge / weight decay) shrinks coefficients and curbs overfitting — the same λ from Lesson 03, now on a neural net via `weight_decay`.

```python
import torch
import torch.nn as nn

torch.manual_seed(0)
xtr, ytr = torch.rand(15,1)*2-1, None
ytr = torch.sin(3*xtr) + 0.1*torch.randn(15,1)
xva = torch.rand(200,1)*2-1
yva = torch.sin(3*xva) + 0.1*torch.randn(200,1)

def fit(weight_decay):
    torch.manual_seed(1)
    net = nn.Sequential(nn.Linear(1,64), nn.Tanh(), nn.Linear(64,1))
    opt = torch.optim.Adam(net.parameters(), lr=0.02, weight_decay=weight_decay)
    for _ in range(2000):
        opt.zero_grad(); loss = ((net(xtr)-ytr)**2).mean(); loss.backward(); opt.step()
    return ((net(xva)-yva)**2).mean().item()

for wd in [0.0, 1e-3, 1e-1]:
    print(f"weight_decay={wd:<6}: val MSE {fit(wd):.4f}")
```

Some regularization usually helps; too much underfits. There is no free lunch — you tune it with cross-validation.

---

## 3. Cross-validation done right

Never select hyperparameters on the test set. k-fold CV reuses data honestly.

```python
import torch

def kfold_indices(n, k=5):
    perm = torch.randperm(n)
    return [perm[i::k] for i in range(k)]   # k disjoint folds

torch.manual_seed(0)
x = torch.rand(100,1)*2-1
y = torch.sin(3*x) + 0.1*torch.randn(100,1)

def poly_features(x, d): return torch.cat([x**i for i in range(d+1)], 1)

for degree in [1,3,5,9]:
    folds = kfold_indices(100, 5)
    errs = []
    for f in range(5):
        val = folds[f]; tr = torch.cat([folds[i] for i in range(5) if i!=f])
        coef = torch.linalg.lstsq(poly_features(x[tr],degree), y[tr]).solution
        errs.append(((poly_features(x[val],degree)@coef - y[val])**2).mean().item())
    print(f"degree {degree}: CV MSE {sum(errs)/len(errs):.4f}")
```

Pick the degree with the lowest CV error — that's your principled model-selection rule, and it carries over verbatim to choosing quantum-kernel/circuit hyperparameters.

---

## 4. Barren plateaus: gradients that vanish with size

Here's the QML punchline made tangible. For a generic deep/wide parameterized model with random init, the *variance* of any single gradient component shrinks as the model grows. When gradients are exponentially small, training stalls — a barren plateau.

We simulate the phenomenon with a toy "expressive random model" whose output is an expectation of a random unit vector rotated by many parameters; gradient variance vs. width shows the trend.

```python
import torch

torch.manual_seed(0)

def grad_variance(width, n_samples=200):
    """Variance of d(output)/d(theta_0) over random parameter settings."""
    grads = []
    for _ in range(n_samples):
        theta = (torch.rand(width) * 2 * torch.pi).requires_grad_(True)
        # a highly oscillatory scalar 'loss' standing in for an expressive model output
        out = torch.prod(torch.cos(theta)) + torch.mean(torch.sin(width**0.5 * theta))
        out.backward()
        grads.append(theta.grad[0].item())
    return torch.tensor(grads).var().item()

for width in [2, 6, 12, 20, 40]:
    print(f"width {width:3d}: grad variance {grad_variance(width):.2e}")
```

You should see the variance trend downward as width grows — gradients concentrate around zero. In real variational quantum circuits this decay is *exponential* in qubit count, which is why naive deep ansätze are untrainable and why structure (shallow circuits, problem symmetry, smart initialization) is essential.

---

## 5. The unifying picture

| Classical | Quantum analogue |
|-----------|------------------|
| Overfitting from too much capacity | Over-expressive ansatz / feature map |
| Vanishing/exploding gradients in deep nets | **Barren plateaus** (exp. small gradients) |
| Kernel with near-identity Gram matrix is useless | **Exponential concentration** of quantum kernels |
| Regularization, architecture priors | Symmetry/equivariance, shallow & structured circuits |

The mitigations rhyme: in both worlds you fight expressivity-without-learnability by injecting **structure**. That sets up Lesson 10 (geometric/equivariant models) and Lesson 06 (concentration-free kernels).

References: `arXiv:2501.07433` (concentration ↔ barren plateaus), `arXiv:2506.13865` (analog loss landscapes & phases of matter).

---

## Exercises (fill in)

1. Plot the degree-vs-(train,val) curves from §1 to see the U-shape explicitly (matplotlib).
2. In §4, change the model to a non-expressive one (e.g., `out = theta.mean()`) and confirm gradient variance does *not* decay — structure prevents plateaus.
3. Combine with Lesson 02: sweep RBF `sigma` to large values and show the Gram matrix concentrating, mirroring the gradient-variance decay here.

> Notes:
>
