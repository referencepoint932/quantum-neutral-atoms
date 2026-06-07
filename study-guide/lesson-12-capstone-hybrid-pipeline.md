# Lesson 12 — Capstone: Hybrid Pipelines & Evaluating Quantum Advantage

**Goal:** assemble the pieces into a hybrid quantum-classical model, and — just as important — learn to judge honestly whether a quantum approach actually helps.

**QML tie-in:** Real near-term QML is *hybrid*: a quantum component (kernel, circuit, or sampler) wrapped in classical optimization and classical pre/post-processing, all in PyTorch. This is the Qadence/QEK deployment model. The discipline that separates hype from results is rigorous baselining and the classical-simulability check.

Prereqs: all prior lessons.

---

## 1. A hybrid model: classical layers + quantum layer + classical head

The quantum circuit (Lesson 09) becomes one `nn.Module` among others. Autograd/parameter-shift makes the whole stack trainable end-to-end.

```python
import torch
import torch.nn as nn

# --- reuse the simulated circuit from Lesson 09 (abbreviated here) ---
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
Z = torch.tensor([[1,0],[0,-1]],dtype=torch.cfloat); OBS=on_qubit(Z,0,n); ENT=cnot(0,1,n)

class QuantumLayer(nn.Module):
    def __init__(self, n_qubits=2):
        super().__init__()
        self.theta = nn.Parameter(torch.rand(n_qubits)*6.28)
        self.n = n_qubits
    def forward(self, feats):                 # feats: (batch, n_qubits) encoding angles
        outs = []
        for row in feats:
            psi = torch.zeros(2**self.n, dtype=torch.cfloat); psi[0]=1.0
            for q in range(self.n): psi = on_qubit(Ry(row[q]), q, self.n) @ psi
            psi = ENT @ psi
            for q in range(self.n): psi = on_qubit(Ry(self.theta[q]), q, self.n) @ psi
            outs.append(torch.vdot(psi, OBS @ psi).real)
        return torch.stack(outs).unsqueeze(1)

class Hybrid(nn.Module):
    def __init__(self):
        super().__init__()
        self.pre = nn.Linear(2, 2)            # classical preprocessing -> encoding angles
        self.q = QuantumLayer(2)
        self.head = nn.Linear(1, 1)           # classical post-processing
    def forward(self, x):
        return self.head(self.q(torch.tanh(self.pre(x))))

torch.manual_seed(0)
X = torch.randn(60, 2); y = (X[:,0]*X[:,1] > 0).float().unsqueeze(1)   # XOR-like target
model = Hybrid()
opt = torch.optim.Adam(model.parameters(), lr=0.05)
lossfn = nn.BCEWithLogitsLoss()
for step in range(150):
    opt.zero_grad(); loss = lossfn(model(X), y); loss.backward(); opt.step()
acc = ((torch.sigmoid(model(X))>0.5).float()==y).float().mean()
print(f"hybrid train loss {loss.item():.3f}  acc {acc.item():.2%}")
```

`opt.step()` updates classical *and* quantum parameters together. On Pasqal hardware the `QuantumLayer.forward` would call a neutral-atom backend and gradients would use parameter-shift — the surrounding code is unchanged.

---

## 2. The discipline: always beat a classical baseline

A quantum result means nothing without a fair classical comparison on the same data and budget.

```python
import torch, torch.nn as nn

torch.manual_seed(0)
X = torch.randn(60,2); y = (X[:,0]*X[:,1] > 0).float().unsqueeze(1)

# classical baseline of similar size
clf = nn.Sequential(nn.Linear(2,8), nn.Tanh(), nn.Linear(8,1))
opt = torch.optim.Adam(clf.parameters(), lr=0.05); lossfn = nn.BCEWithLogitsLoss()
for _ in range(150):
    opt.zero_grad(); loss = lossfn(clf(X), y); loss.backward(); opt.step()
acc = ((torch.sigmoid(clf(X))>0.5).float()==y).float().mean()
print(f"classical baseline acc {acc.item():.2%}")
```

If the classical baseline matches or beats the quantum model on your task, there is no advantage — report that honestly. Most claimed advantages evaporate against a well-tuned classical baseline; the ones that survive are the interesting science.

---

## 3. The classical-simulability check

A quantum model is only *potentially* advantageous if a classical computer can't efficiently reproduce it. Two quick sanity questions before claiming anything:

- **Does the kernel/circuit concentrate?** (Lesson 06.) If the Gram matrix → identity, it's useless regardless of simulability.
- **Is it efficiently simulable?** Product states, Clifford circuits, low-entanglement / low-depth circuits, and many encodings are classically simulable. If your laptop reproduces the output in seconds (as in this whole guide!), that's a simulation, not an advantage — useful for learning, not for claiming supremacy.

The genuinely interesting regime is **hard-to-simulate AND non-concentrating AND beats classical baselines** — the target QEK and Pasqal's algorithms aim for.

---

## 4. Putting the whole guide together

```
 data ──► classical preprocessing ──► QUANTUM PIECE ──► classical postprocessing ──► prediction
                                       (one of:)
                                       • quantum kernel  (L02,03,06)  → SVM/KRR
                                       • variational circuit (L09)    → trainable model
                                       • Born machine (L08)           → generative samples
                                       • DQC (L11)                    → DE solver
          everything trained with the L01 loop; judged with L05 generalization + §2/§3 here
```

Pasqal's stack maps on cleanly: **QEK** is the quantum-kernel branch; **Qadence** builds the variational/DQC branches; neutral-atom hardware is the sampler underneath all of them.

---

## 5. Suggested next steps beyond this guide

1. Install Pasqal's open-source tools and re-run Lessons 03/06 with real **QEK** kernels, and Lesson 09 with **Qadence** on a simulator backend.
2. Reproduce the molecular-toxicity graph-classification workflow (Lessons 04 + 06).
3. Read the frontier papers cited throughout, now with the hands-on intuition to follow the math:
   - Concentration ↔ barren plateaus: `arXiv:2501.07433`
   - Analog loss landscapes: `arXiv:2506.13865`
   - Concentration-free Rydberg kernels: `arXiv:2508.10819`
   - Symmetry-structured trainable kernels: `arXiv:2509.14337`
   - QEK foundations: `arXiv:2107.03247`, `arXiv:2509.09421`
   - Qadence: `arXiv:2401.09915`

---

## Exercises (fill in)

1. Run §1 and §2 with several random seeds; report mean ± std accuracy for hybrid vs. classical. Is any gap statistically real?
2. Increase `QuantumLayer` qubits and measure wall-clock time — feel the exponential cost of classical simulation (and why hardware exists).
3. Pick one real dataset (e.g., a small molecular graph set) and run the full QEK → SVM pipeline against an RBF-SVM and a GNN baseline.

> Notes:
>
