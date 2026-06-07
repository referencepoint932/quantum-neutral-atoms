# Lesson 08 — Probabilistic & Generative Modeling

**Goal:** refresh distributions, sampling, maximum likelihood, and KL divergence, then train a tiny generative model — the classical scaffolding for **quantum generative models** (Born machines).

**QML tie-in:** Measurement on a quantum device *is* sampling from a probability distribution defined by the state. A **Born machine** trains a parameterized circuit so its measurement distribution matches data — pure generative modeling. The loss functions (negative log-likelihood, KL, MMD) and the sampling mindset here are exactly what you use there. This also underpins reading QEK output, which is a distribution over measurement samples.

Prereqs: Lesson 01 (autograd), Lesson 07 (cross-entropy = NLL).

---

## 1. Distributions and sampling

A generative model is a distribution you can sample from. Start with the basics in torch.

```python
import torch

torch.manual_seed(0)

# sampling from a categorical (a "measurement" over 4 outcomes)
probs = torch.tensor([0.1, 0.2, 0.3, 0.4])
samples = torch.multinomial(probs, num_samples=10000, replacement=True)
empirical = torch.bincount(samples, minlength=4).float() / 10000
print("target  :", probs.tolist())
print("sampled :", empirical.round(decimals=3).tolist())   # should match closely
```

Measurement on `n` qubits is exactly this: a categorical over `2ⁿ` bitstrings, with probabilities given by the squared amplitudes (Born rule, Lesson 06).

---

## 2. Maximum likelihood = minimizing negative log-likelihood

Fitting a model means making the observed data probable under it. For a categorical, MLE just matches empirical frequencies — and the loss is the **negative log-likelihood**, the same quantity inside cross-entropy.

```python
import torch

torch.manual_seed(0)
# observed counts from some unknown source
data = torch.multinomial(torch.tensor([0.5,0.1,0.1,0.3]), 2000, replacement=True)

# model: unnormalized logits -> softmax probs; train by NLL
logits = torch.zeros(4, requires_grad=True)
opt = torch.optim.Adam([logits], lr=0.1)
for _ in range(500):
    opt.zero_grad()
    logp = torch.log_softmax(logits, dim=0)
    nll = -logp[data].mean()            # negative log-likelihood of the data
    nll.backward(); opt.step()

print("learned probs:", torch.softmax(logits,0).round(decimals=3).tolist())
print("(true source :  [0.5, 0.1, 0.1, 0.3])")
```

A Born machine trains the *same way*: the model probabilities come from a circuit's measurement statistics instead of a softmax, but the NLL objective is identical.

---

## 3. KL divergence: how far apart are two distributions?

KL(p‖q) measures the cost of using q to model true p. It's the natural "distance" for generative training (and appears in the analysis of kernel/circuit expressivity).

```python
import torch

def kl(p, q, eps=1e-12):
    p = p + eps; q = q + eps
    return (p * (p / q).log()).sum()

p = torch.tensor([0.5, 0.1, 0.1, 0.3])
q1 = torch.tensor([0.45, 0.15, 0.1, 0.3])     # close to p
q2 = torch.tensor([0.25, 0.25, 0.25, 0.25])   # uniform, far from p
print("KL(p||close)  :", kl(p, q1).item())
print("KL(p||uniform):", kl(p, q2).item())    # larger
print("KL(p||p)      :", kl(p, p).item())      # ~0
```

KL is zero only when the distributions match, and it's asymmetric — both facts matter when choosing a generative loss.

---

## 4. A minimal generative model: 1-D density via a normalizing-flow-style map

To generate *continuous* data, learn an invertible map from a simple base (Gaussian) to the data. Here's the simplest trainable version: fit a monotonic affine+tanh warp by matching moments (a stand-in for full flow training).

```python
import torch
import torch.nn as nn

torch.manual_seed(0)
# target data: bimodal
data = torch.cat([torch.randn(500)*0.3 + 1.5, torch.randn(500)*0.3 - 1.5]).unsqueeze(1)

gen = nn.Sequential(nn.Linear(1,32), nn.Tanh(), nn.Linear(32,32), nn.Tanh(), nn.Linear(32,1))
opt = torch.optim.Adam(gen.parameters(), lr=0.01)

def mmd(a, b, sigma=0.5):
    # maximum mean discrepancy with an RBF kernel: a sample-based distribution distance
    def k(x,y): return torch.exp(-(torch.cdist(x,y)**2)/(2*sigma**2))
    return k(a,a).mean() + k(b,b).mean() - 2*k(a,b).mean()

for step in range(1500):
    z = torch.randn(256, 1)            # base samples
    fake = gen(z)                      # generated samples
    real = data[torch.randint(0, len(data), (256,))]
    opt.zero_grad()
    loss = mmd(fake, real)            # match the two distributions
    loss.backward(); opt.step()

with torch.no_grad():
    out = gen(torch.randn(2000,1))
print(f"generated mean {out.mean():.2f} std {out.std():.2f}  (target ~mean 0, std ~1.55)")
```

**MMD** is a kernel-based distribution distance — note it reuses the RBF kernel from Lesson 02. Quantum generative models are frequently trained with exactly this MMD objective, because it only needs *samples*, which is all a quantum device gives you.

---

## 5. Bridge to quantum generative models

| Classical generative model | Quantum (Born machine / DQGM) |
|----------------------------|-------------------------------|
| Sample from softmax / flow | Sample from measurement (Born rule) |
| NLL / KL / **MMD** loss | Same losses (MMD especially — sample-only) |
| Neural net generator | Parameterized quantum circuit |
| Backprop | Autodiff / parameter-shift (Lesson 09) |

Neutral-atom hardware is a natural sampler, so generative modeling is a leading near-term QML application. The key transferable skill is thinking in terms of *distributions and samples*, not closed-form densities.

---

## Exercises (fill in)

1. Verify KL is asymmetric: compute `kl(q2, p)` vs `kl(p, q2)`.
2. Swap MMD for an energy-distance loss and compare the generated distribution.
3. Make the base distribution categorical over bitstrings (length-`2ⁿ`) and train to match a target — this is a direct classical simulation of a Born machine.

> Notes:
>
