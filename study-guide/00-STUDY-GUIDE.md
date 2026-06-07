# PyTorch → QML Study Guide

*A hands-on curriculum that refreshes core ML in PyTorch, deliberately routed toward the concepts you'll need for QML on Pasqal's neutral-atom platform. Built to be filled out over several steps.*

Companion file: `../ML-topics-to-revisit-for-Pasqal-QML.md` (the topic map this curriculum implements).

---

## How this guide works

Each lesson is a standalone markdown file with explanation + runnable PyTorch code blocks. Code uses only `torch`, `numpy`, and optionally `matplotlib`/`scikit-learn`. Copy a block into a `.py` file or notebook and run it.

The thread running through every lesson: **the same machinery you use in classical ML is what you'll reach for in QML.** Autograd → variational-circuit training. Feature maps + kernels → the Quantum Evolution Kernel. Generalization theory → judging whether a quantum model actually beats a classical baseline.

**Status legend:** ✅ filled · 🟡 partial · ⬜ to be written in a later step.

---

## Curriculum

**Part I — Classical foundations → kernels (the QEK path)**

| # | Lesson | Core PyTorch concept | QML tie-in | Status |
|---|--------|----------------------|------------|--------|
| 01 | Tensors & autograd | Tensors, `autograd`, manual gradient descent | Parameter-shift / variational training loops in Qadence | ✅ |
| 02 | Feature maps & kernels | Building Gram matrices, the kernel trick in torch | Quantum feature maps; kernel = state overlap | ✅ |
| 03 | Kernel ridge & SVM | Closed-form KRR in torch; SVM via scikit-learn | Quantum Evolution Kernel plugs into KRR/SVM | ✅ |
| 04 | Graph data & graph kernels | Adjacency tensors; Weisfeiler–Lehman kernel | What QEK competes against on graph tasks | ✅ |

**Part II — Training dynamics & the classical→quantum kernel leap**

| # | Lesson | Core PyTorch concept | QML tie-in | Status |
|---|--------|----------------------|------------|--------|
| 05 | Generalization & training dynamics | Overfitting, regularization, vanishing gradients | Barren plateaus ↔ kernel concentration | ✅ |
| 06 | From classical to quantum kernels | Simulating a quantum feature map; concentration | QEK on Rydberg Hamiltonians; concentration-free kernels | ✅ |

**Part III — Neural & generative foundations**

| # | Lesson | Core PyTorch concept | QML tie-in | Status |
|---|--------|----------------------|------------|--------|
| 07 | Neural networks & classification | `nn.Module`, softmax + cross-entropy | VQC structure; data re-uploading | ✅ |
| 08 | Probabilistic & generative modeling | Sampling, NLL, KL, MMD | Born machines; measurement = sampling | ✅ |

**Part IV — Quantum models & mastery**

| # | Lesson | Core PyTorch concept | QML tie-in | Status |
|---|--------|----------------------|------------|--------|
| 09 | Variational quantum circuits (simulated) | Gates as tensors; the training loop | The core QML model + **parameter-shift rule** | ✅ |
| 10 | Expressivity & geometric/equivariant ML | Fourier view; symmetry/invariant features | Concentration-free, symmetry-structured kernels | ✅ |
| 11 | Physics-informed ML & differential equations | Higher derivatives via autograd; residual loss | Pasqal's differentiable quantum circuits (DQC) | ✅ |
| 12 | Capstone: hybrid pipelines & evaluating advantage | Quantum layer in `nn.Module`; baselining | Qadence/QEK deployment; simulability check | ✅ |

---

## Learning objectives

By the end of the guide you should be able to:

- Build and train a model in PyTorch by hand, understanding exactly where gradients come from.
- Construct feature maps and Gram matrices (vector *and* graph data), and explain the kernel trick.
- Fit KRR/SVM and swap in any kernel — including a simulated quantum kernel — without touching the learner, the precise interface QEK uses.
- Explain the expressivity–trainability trade-off, and recognize barren plateaus and kernel concentration as two faces of it.
- Build and train a variational quantum circuit in simulation, and derive/verify the **parameter-shift rule**.
- Use symmetry/equivariance and the Fourier view to reason about what quantum models can represent.
- Solve a differential equation with a physics-informed loss (the classical mirror of Pasqal's DQC).
- Assemble a hybrid quantum-classical model and judge quantum advantage honestly (baselines + simulability).

---

## Environment setup

```bash
# Python 3.10+ recommended
pip install torch numpy matplotlib scikit-learn
```

Quick sanity check:

```python
import torch
print("torch", torch.__version__)
print("cuda available:", torch.cuda.is_available())
x = torch.tensor([1.0, 2.0, 3.0])
print("mean:", x.mean().item())
```

---

## Progress tracker

- [X] Lesson 01 — Tensors & autograd
- [ ] Lesson 02 — Feature maps & kernels
- [ ] Lesson 03 — Kernel ridge & SVM
- [ ] Lesson 04 — Graph data & graph kernels
- [ ] Lesson 05 — Generalization & training dynamics
- [ ] Lesson 06 — From classical to quantum kernels
- [ ] Lesson 07 — Neural networks & classification
- [ ] Lesson 08 — Probabilistic & generative modeling
- [ ] Lesson 09 — Variational quantum circuits (simulated)
- [ ] Lesson 10 — Expressivity & geometric/equivariant ML
- [ ] Lesson 11 — Physics-informed ML & differential equations
- [ ] Lesson 12 — Capstone: hybrid pipelines & evaluating advantage

---

## Notes / scratchpad (fill in as you go)

> Use this section to jot questions, results, and links back to the QML papers in the companion topic file. Left intentionally open for subsequent steps.

-
-
-
