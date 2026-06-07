# Lesson 11 — Physics-Informed ML & Differential Equations

**Goal:** train a model to solve a differential equation by putting the equation itself in the loss — using PyTorch autograd to compute derivatives of the model. This is the classical version of Pasqal's **differentiable quantum circuit (DQC)** approach to PDEs.

**QML tie-in:** Pasqal has pushed neutral-atom computing for *physics-informed* machine learning and solving differential equations with differentiable quantum circuits. The recipe is identical to what you build here — a parameterized model, derivatives via differentiation, a residual loss — except the model is a quantum circuit and the derivatives come from parameter-shift (Lesson 09).

Prereqs: Lessons 01 (autograd, higher derivatives), 07 (nn models), 09 (differentiable circuits).

---

## 1. Derivatives of a model, for free

The enabling trick: autograd differentiates the model *with respect to its input*, not just its parameters. That gives you `du/dx`, `d²u/dx²`, etc.

```python
import torch

# u(x) = sin(x); check we can get du/dx = cos(x) via autograd
x = torch.linspace(0, 3, 5, requires_grad=True)
u = torch.sin(x)
du, = torch.autograd.grad(u.sum(), x, create_graph=True)   # create_graph -> differentiable again
print("du/dx :", du.round(decimals=3).tolist())
print("cos(x):", torch.cos(x).round(decimals=3).tolist())
```

`create_graph=True` keeps the derivative differentiable, so you can take *second* derivatives — essential for second-order PDEs.

---

## 2. Physics-informed loss: solve an ODE with no training data

Solve `u'(x) = u(x)`, `u(0) = 1` (whose solution is `eˣ`). We never supply solution values — only the equation and boundary condition, encoded as a loss.

```python
import torch
import torch.nn as nn

torch.manual_seed(0)

model = nn.Sequential(nn.Linear(1,32), nn.Tanh(), nn.Linear(32,32), nn.Tanh(), nn.Linear(32,1))
opt = torch.optim.Adam(model.parameters(), lr=0.01)

def u(x):
    return model(x)

for step in range(3000):
    opt.zero_grad()
    x = torch.rand(64, 1, requires_grad=True) * 2.0       # collocation points in [0,2]
    ux = u(x)
    dudx, = torch.autograd.grad(ux.sum(), x, create_graph=True)

    residual = dudx - ux                                  # ODE: u' - u = 0
    loss_ode = (residual**2).mean()

    x0 = torch.zeros(1,1)
    loss_bc = (u(x0) - 1.0)**2                            # boundary: u(0)=1

    loss = loss_ode + loss_bc
    loss.backward(); opt.step()
    if step % 1000 == 0:
        print(f"step {step:4d}  loss {loss.item():.2e}")

# compare to the exact solution e^x
xt = torch.linspace(0, 2, 6).unsqueeze(1)
with torch.no_grad():
    pred = u(xt).squeeze()
print("model :", pred.round(decimals=3).tolist())
print("exp(x):", torch.exp(xt.squeeze()).round(decimals=3).tolist())
```

The loss has two pieces: the **equation residual** (sampled at random collocation points) and the **boundary condition**. Minimizing it forces the model to *be* the solution. That's the entire physics-informed paradigm.

---

## 3. Second-order example (the structure scales)

A harmonic oscillator `u'' + u = 0`, `u(0)=0`, `u'(0)=1` (solution `sin(x)`). Same recipe, one more derivative.

```python
import torch
import torch.nn as nn

torch.manual_seed(0)
model = nn.Sequential(nn.Linear(1,64), nn.Tanh(), nn.Linear(64,64), nn.Tanh(), nn.Linear(64,1))
opt = torch.optim.Adam(model.parameters(), lr=0.005)

def deriv(y, x):
    return torch.autograd.grad(y.sum(), x, create_graph=True)[0]

for step in range(4000):
    opt.zero_grad()
    x = torch.rand(64,1, requires_grad=True)*6.0
    u = model(x)
    u_x = deriv(u, x)
    u_xx = deriv(u_x, x)
    loss = ((u_xx + u)**2).mean()                          # ODE residual
    x0 = torch.zeros(1,1, requires_grad=True)
    u0 = model(x0); u0_x = deriv(u0, x0)
    loss = loss + (u0)**2 + (u0_x - 1.0)**2                # u(0)=0, u'(0)=1
    loss.backward(); opt.step()

xt = torch.linspace(0, 6, 7).unsqueeze(1)
with torch.no_grad(): pred = model(xt).squeeze()
print("model :", pred.round(decimals=2).tolist())
print("sin(x):", torch.sin(xt.squeeze()).round(decimals=2).tolist())
```

---

## 4. Bridge to Pasqal's DQC

| Physics-informed NN (here) | Differentiable Quantum Circuit (Pasqal) |
|----------------------------|-----------------------------------------|
| `model(x)` = neural net | `u(x)` = quantum circuit expectation |
| `du/dx` via autograd | `du/dx` via **parameter-shift** / autodiff (Lesson 09) |
| Residual + BC loss | Same loss |
| Adam over weights | Adam over circuit angles |

Because parameter-shift gives *exact* derivatives of a circuit's output (Lesson 09), you can build the same residual loss with a quantum model — that's the heart of Pasqal's DQC for solving differential equations, and a key reason their neutral-atom platform is pitched for scientific computing, not just classification.

Reference: Pasqal, "Neutral Atom Quantum Computing for Physics-Informed Machine Learning."

---

## Exercises (fill in)

1. Solve `u' = -2x·u`, `u(0)=1` (solution `e^{-x²}`) and check the fit.
2. Add a source term: `u'' + u = cos(2x)` and compare to the analytic particular solution.
3. Replace the neural net with the simulated circuit from Lesson 09 and compute `du/dx` via parameter-shift — a fully "quantum" PINN in simulation.

> Notes:
>
