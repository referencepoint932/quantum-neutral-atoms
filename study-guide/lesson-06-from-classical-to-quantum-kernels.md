# Lesson 06 — From Classical Kernels to Quantum Kernels

**Goal:** make the leap explicit. Simulate a tiny quantum feature map, compute a quantum kernel as a state overlap, feed it into the *same* learners from Lesson 03, and watch exponential concentration appear with your own eyes.

**QML tie-in:** This is where the guide converges on Pasqal's QEK. A quantum kernel is `k(x,x') = |⟨ψ(x)|ψ(x')⟩|²` — an inner product of states produced by a data-dependent circuit (gate-based) or Hamiltonian evolution (Pasqal's analog approach). Everything classical downstream is unchanged.

Prereqs: Lesson 02 (kernels), Lesson 03 (KRR/SVM), Lesson 05 (concentration).

---

## 1. A quantum feature map, simulated on a laptop

No quantum hardware needed: a state on `n` qubits is just a complex vector of length `2ⁿ`. We encode a scalar `x` by rotating each qubit, then entangle. The amplitudes are a nonlinear feature map of `x`.

```python
import torch

def single_qubit_state(angle):
    # |psi> = cos(angle/2)|0> + sin(angle/2)|1>
    return torch.tensor([torch.cos(angle/2), torch.sin(angle/2)], dtype=torch.cfloat)

def feature_state(x, n_qubits=4, scale=1.0):
    """Tensor-product encoding: each qubit rotated by scale*x, then a phase per qubit."""
    psi = torch.tensor([1.0+0j])
    for q in range(n_qubits):
        ang = torch.tensor(scale * x * (q + 1))      # different frequency per qubit
        qubit = single_qubit_state(ang)
        # add a data-dependent phase to make the map richer
        qubit = qubit * torch.tensor([1.0, torch.exp(1j*torch.tensor(scale*x))])
        psi = torch.kron(psi, qubit)                  # tensor product -> length doubles
    return psi / psi.norm()

s = feature_state(0.5, n_qubits=4)
print("state length:", s.shape[0], "(= 2^4)   norm:", s.norm().item())
```

`torch.kron` is the tensor product — the operation that makes the feature space grow *exponentially* (`2ⁿ`) with qubit count. That exponential lift is the source of both quantum kernels' promise and their concentration peril.

---

## 2. The quantum kernel = squared state overlap

```python
import torch

def quantum_kernel_matrix(xs, n_qubits=4, scale=1.0):
    states = [feature_state(float(x), n_qubits, scale) for x in xs]
    n = len(states)
    K = torch.zeros(n, n)
    for i in range(n):
        for j in range(n):
            overlap = torch.vdot(states[i], states[j])   # <psi_i|psi_j>
            K[i, j] = (overlap.conj() * overlap).real     # |<.|.>|^2 (fidelity)
    return K

xs = torch.linspace(-1, 1, 6)
K = quantum_kernel_matrix(xs, n_qubits=4, scale=2.0)
print(K.round(decimals=3))
print("symmetric:", torch.allclose(K, K.T), "| diag~1:", torch.allclose(torch.diag(K), torch.ones(6)))
```

It's a valid kernel: symmetric, unit diagonal, PSD. So it drops straight into Lesson 03.

---

## 3. Quantum kernel → same KRR/SVM, unchanged

```python
import torch

# regression target
torch.manual_seed(0)
xtr = torch.linspace(-1, 1, 40)
ytr = torch.sin(4*xtr) + 0.05*torch.randn(40)

K = quantum_kernel_matrix(xtr, n_qubits=5, scale=3.0)
lam = 1e-3
alpha = torch.linalg.solve(K + lam*torch.eye(K.shape[0]), ytr)
train_mse = ((K @ alpha) - ytr).pow(2).mean()
print(f"quantum-kernel KRR train MSE: {train_mse.item():.4f}")
```

This is identical to Lesson 03's KRR — only `K` now comes from quantum states. On Pasqal hardware, `K` would come from a Rydberg processor instead of `torch.kron`; the learning code wouldn't change.

---

## 4. Exponential concentration, observed

Scale up qubits and a poorly chosen feature map makes every pair of distinct points look orthogonal — the off-diagonal kernel values crash toward zero. The Gram matrix → identity → no learnable signal. This is the quantum face of Lesson 05's vanishing gradients.

```python
import torch

xs = torch.linspace(-1, 1, 8)
print("n_qubits | mean off-diagonal kernel value")
for n in [2, 4, 6, 8, 10]:
    K = quantum_kernel_matrix(xs, n_qubits=n, scale=3.0)
    off = K[~torch.eye(len(xs), dtype=bool)].mean()
    print(f"   {n:2d}    |   {off.item():.4e}")
```

Watch the mean off-diagonal value shrink as qubits increase — distinct inputs become indistinguishable to the kernel. Push `n` high enough and KRR/SVM on this kernel degrade to guessing.

---

## 5. Mitigations — the research frontier

The fix, as in Lesson 05, is **structure**:

- **Bandwidth / encoding scale.** Tuning `scale` (the data-encoding strength) is the quantum analogue of the RBF `sigma`; it directly controls concentration.
- **Concentration-free constructions.** Designing the interaction regime to avoid collapse — e.g. kernels built in the **Rydberg blockade** regime (`arXiv:2508.10819`), native to Pasqal's neutral-atom hardware.
- **Symmetry / equivariance.** Restricting the feature map to respect problem symmetries keeps the kernel informative — trainable kernels for symmetry-structured data (`arXiv:2509.14337`). This is Lesson 10.
- **Avoiding classical simulability.** A kernel that concentrates is useless; but a kernel that's *too* easy is also pointless because a classical computer could compute it. The sweet spot — hard-to-simulate yet non-concentrating — is exactly what QEK targets.

---

## 6. The real thing: Pasqal's QEK

The toy `feature_state` above stands in for what QEK does physically: encode a **graph** (Lesson 04) into a Rydberg Hamiltonian, evolve it, and turn the measured distribution into a kernel value. Then SVM/KRR (Lesson 03) finish the job — the pipeline Pasqal used for molecular toxicity prediction on neutral-atom hardware.

```
This lesson (simulated):  x ──► kron-encoded state |ψ(x)⟩ ──► |⟨ψ|ψ⟩|² ──► K ──► KRR/SVM
Pasqal QEK (hardware):    graph ──► Rydberg Hamiltonian ──► evolve+measure ──► K ──► KRR/SVM
```

References: `arXiv:2501.07433`, `arXiv:2508.10819`, `arXiv:2509.14337`, `arXiv:2107.03247`; Pasqal QEK library docs.

---

## Exercises (fill in)

1. Sweep `scale` at fixed `n_qubits=8` and find the value that maximizes mean off-diagonal kernel value — your "bandwidth" sweet spot.
2. Compare quantum-kernel KRR vs. RBF KRR (Lesson 03) on the same `sin(4x)` target. Which wins, and at what `n_qubits`?
3. Replace the product encoding with one that adds entangling structure between neighbors and observe the effect on concentration.

> Notes:
>
