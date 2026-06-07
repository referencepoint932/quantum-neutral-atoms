# Lesson 09 — Variational Quantum Circuits (Simulated)

**Goal:** build a parameterized quantum circuit from scratch in PyTorch, train it as a classifier, and derive the **parameter-shift rule** — the gradient method that makes QML trainable on real hardware.

**QML tie-in:** This *is* QML's central model. Pasqal's Qadence is built to construct these circuits (digital-analog) and differentiate them with autodiff + parameter-shift on neutral-atom devices. Here you simulate the whole thing on a laptop so the moving parts are transparent before you touch hardware.

Prereqs: Lesson 01 (autograd/optim), Lesson 06 (state vectors, `kron`), Lesson 07 (training a classifier).

---

## 1. Qubits, gates, and states as tensors

A state is a length-`2ⁿ` complex vector; a gate is a unitary matrix applied to it. We build the few gates we need.

```python
import torch

# single-qubit gates
def Ry(theta):
    c, s = torch.cos(theta/2), torch.sin(theta/2)
    return torch.stack([torch.stack([c, -s]), torch.stack([s, c])]).cfloat()

def Rz(theta):
    e = torch.exp(1j*theta/2)
    return torch.tensor([[1/e, 0],[0, e]], dtype=torch.cfloat)

I2 = torch.eye(2, dtype=torch.cfloat)

def on_qubit(gate, q, n):
    """Embed a single-qubit gate acting on qubit q into the full n-qubit space."""
    ops = [gate if i == q else I2 for i in range(n)]
    full = ops[0]
    for op in ops[1:]:
        full = torch.kron(full, op)
    return full

n = 2
psi0 = torch.zeros(2**n, dtype=torch.cfloat); psi0[0] = 1.0   # |00>
print("start:", psi0.tolist())
```

---

## 2. An entangling gate (CNOT)

Entanglement is what gives quantum models access to correlations a product state can't represent.

```python
import torch

def cnot(control, target, n):
    dim = 2**n
    U = torch.zeros(dim, dim, dtype=torch.cfloat)
    for b in range(dim):
        bits = [(b >> (n-1-k)) & 1 for k in range(n)]
        if bits[control] == 1:
            bits[target] ^= 1
        b2 = sum(bit << (n-1-k) for k, bit in enumerate(bits))
        U[b2, b] = 1.0
    return U

print(cnot(0, 1, 2).real.int().tolist())   # standard 4x4 CNOT
```

---

## 3. A variational classifier

Recipe: **encode** data into rotation angles → **variational** trainable rotations → **entangle** → **measure** an observable. The measured expectation is the model output; train it with cross-entropy-style loss like any classifier (Lesson 07).

```python
import torch

n = 2
I2 = torch.eye(2, dtype=torch.cfloat)
def Ry(t):
    c,s = torch.cos(t/2), torch.sin(t/2)
    return torch.stack([torch.stack([c,-s]),torch.stack([s,c])]).cfloat()
def on_qubit(g,q,n):
    ops=[g if i==q else I2 for i in range(n)]; f=ops[0]
    for o in ops[1:]: f=torch.kron(f,o)
    return f
def cnot(c,t,n):
    d=2**n; U=torch.zeros(d,d,dtype=torch.cfloat)
    for b in range(d):
        bits=[(b>>(n-1-k))&1 for k in range(n)]
        if bits[c]==1: bits[t]^=1
        U[sum(x<<(n-1-k) for k,x in enumerate(bits)),b]=1.0
    return U

# observable: Z on qubit 0  ->  eigenvalues +1/-1, expectation in [-1,1]
Z = torch.tensor([[1,0],[0,-1]],dtype=torch.cfloat)
OBS = on_qubit(Z, 0, n)
ENT = cnot(0, 1, n)

def circuit(x, theta):
    psi = torch.zeros(2**n, dtype=torch.cfloat); psi[0]=1.0
    # data encoding (feature map): rotate each qubit by the input
    for q in range(n):
        psi = on_qubit(Ry(x), q, n) @ psi
    psi = ENT @ psi
    # variational layer
    for q in range(n):
        psi = on_qubit(Ry(theta[q]), q, n) @ psi
    psi = ENT @ psi
    exp = torch.vdot(psi, OBS @ psi).real      # <Z_0>
    return exp

# train to classify INNER vs OUTER: label +1 if |x|<0.75 else -1
torch.manual_seed(0)
theta = (torch.rand(n)*6.28).requires_grad_(True)
opt = torch.optim.Adam([theta], lr=0.1)
xs = torch.linspace(-1.5, 1.5, 40)
ys = torch.where(xs.abs() < 0.75, 1.0, -1.0)

for step in range(200):
    opt.zero_grad()
    preds = torch.stack([circuit(x, theta) for x in xs])
    loss = ((preds - ys)**2).mean()
    loss.backward(); opt.step()

acc = ((torch.stack([circuit(x,theta) for x in xs]) > 0) == (ys>0)).float().mean()
print(f"final loss {loss.item():.3f}  accuracy {acc.item():.2%}")   # ~95%
```

Note the structure: this is Lesson 07's classifier with a quantum circuit as the model. `loss.backward()` works because every gate is a differentiable torch op — exactly how Qadence's simulator backend trains.

**Why inner/outer and not `sign(x)`?** With an `Ry(x)` encoding measured via `⟨Z₀⟩`, this circuit's output is an *even* function of `x` — `circuit(-x, θ) == circuit(x, θ)`. So it physically cannot represent an odd target like `sign(x)`, and training would stall near 50%. This is a concrete, hands-on encounter with **inductive bias from the encoding** (Lesson 10): the data-encoding scheme decides what functions are even representable, before any training. The fix for odd targets is a richer encoding — e.g. **data re-uploading** (Exercise 2).

---

## 4. The parameter-shift rule (gradients on real hardware)

On a *real* device you can't backprop through the physics — you only get expectation values. The parameter-shift rule recovers exact gradients from two extra circuit evaluations:

> ∂⟨O⟩/∂θ = ½ [ f(θ + π/2) − f(θ − π/2) ]

for gates of the form `exp(-i θ P / 2)` (like `Ry`, `Rz`). Verify it matches autodiff:

```python
import torch

def f(theta_vec):                       # scalar circuit output as a function of theta
    return circuit(torch.tensor(0.7), theta_vec)

theta = torch.tensor([1.0, 2.0], requires_grad=True)
out = f(theta); out.backward()
autodiff_grad = theta.grad.clone()

# parameter-shift estimate, component by component
shift = torch.pi/2
ps_grad = torch.zeros(2)
for i in range(2):
    tp = theta.detach().clone(); tp[i]+=shift
    tm = theta.detach().clone(); tm[i]-=shift
    ps_grad[i] = 0.5*(f(tp) - f(tm))

print("autodiff       :", autodiff_grad.round(decimals=4).tolist())
print("parameter-shift:", ps_grad.round(decimals=4).tolist())
print("match:", torch.allclose(autodiff_grad, ps_grad, atol=1e-4))
```

They agree. This is the crux of trainable QML: the *exact* gradient is obtainable from device measurements alone — no backprop through hardware required. Qadence implements this (including higher-order shifts) so the same `loss.backward()` works whether you target a simulator or a neutral-atom QPU.

---

## 5. Connecting to the rest of the guide

- The circuit's data-encoding layer is a **quantum feature map** (Lesson 06) — push it further and you're back at quantum kernels.
- Add depth/width and you hit **barren plateaus** (Lesson 05): gradients from §4 shrink exponentially. Structure and shallow circuits are the defense.
- Swap the loss for an MMD over measurement samples and this becomes a **Born machine** (Lesson 08).
- Wrap it in a PyTorch `nn.Module` alongside classical layers → a **hybrid model** (Lesson 12).

References: Qadence paper `arXiv:2401.09915`; Qadence QML tutorial (docs.pasqal.com/qadence).

---

## Exercises (fill in)

1. Add a second variational layer and more qubits; learn a harder *even* target like a triple-bump (`+1` on three separated intervals).
2. Implement **data re-uploading**: alternate encode/variational layers 3×. Show it can now fit an *odd* target like `sign(x)` that the single-encode circuit above provably cannot — the encoding's inductive bias in action.
3. Add shot noise to §4 (estimate `f` from finite samples) and watch parameter-shift gradients get noisy — the real-hardware reality.

> Notes:
>
