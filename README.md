# Quantum Neutral Atoms

Research library for exploring neutral-atom quantum computing with
[Qadence](https://pasqal-io.github.io/qadence/), running on real hardware and
simulators via [Azure Quantum](https://azure.microsoft.com/products/quantum) and
[Pasqal](https://www.pasqal.com/).

## Overview

Neutral-atom devices encode qubits in individual atoms held by optical tweezers,
with entanglement mediated by the Rydberg interaction. This project uses Qadence
to build and simulate analog and digital-analog programs, and targets Pasqal's
hardware through the Azure Quantum backend.

Alongside the hardware-facing code, the repository hosts a **hands-on learning
track** that rebuilds the core ideas of quantum machine learning from PyTorch
first principles — culminating in Pasqal's Quantum Evolution Kernel (QEK),
variational circuits (Qadence), differentiable quantum circuits (DQC), and an
honest framework for judging quantum advantage.

## Repository structure

| Path | What's inside |
|------|---------------|
| [`notebooks/`](notebooks/) | Runnable Jupyter notebooks — the QML kickstart series plus Qadence/Pasqal application tutorials |
| [`study-guide/`](study-guide/) | The PyTorch → QML curriculum (12 lessons in markdown); the notebooks implement it |
| [`docs/`](docs/) | Background reference on neutral-atom hardware, Pasqal's platform, key publications, and the project plan |
| `main.py` | Placeholder entry point |
| `pyproject.toml` | Project metadata and dependencies (managed with `uv`) |

## The PyTorch → QML kickstart notebook series

A six-notebook arc that takes you from a hand-written gradient-descent loop to a
hybrid quantum-classical pipeline — implementing all 12 lessons of the
[study guide](study-guide/00-STUDY-GUIDE.md). Every notebook is **fully executed
with figures and printed results embedded**, so you can read it like an article
or run it cell-by-cell. The recurring thread: *the same machinery you use in
classical ML is what you reach for in QML.*

Work through them in order:

| # | Notebook | Lessons | What you build |
|---|----------|---------|----------------|
| 1 | [`qadence-kickstart-qml.ipynb`](notebooks/qadence-kickstart-100-qml-basics.ipynb) | 01 | Tensors, autograd, and a training loop by hand → the same loop variational QML uses |
| 2 | [`qadence-kickstart-kernels.ipynb`](notebooks/qadence-kickstart-101-kernels.ipynb) | 02–03 | Feature maps, the kernel trick, Gram matrices, kernel ridge regression & SVM |
| 3 | [`qadence-kickstart-graphs-generalization.ipynb`](notebooks/qadence-kickstart-102-graphs-generalization.ipynb) | 04–05 | Graph kernels (Weisfeiler–Lehman) and generalization / barren plateaus |
| 4 | [`qadence-kickstart-quantum-kernels-nn-generative.ipynb`](notebooks/qadence-kickstart-103-quantum-kernels-nn-generative.ipynb) | 06–08 | Simulated quantum kernels, neural-net classifiers, and generative models |
| 5 | [`qadence-kickstart-variational-geometric-pinn.ipynb`](notebooks/qadence-kickstart-104-variational-geometric-pinn.ipynb) | 09–11 | Variational circuits + parameter-shift, expressivity/symmetry, physics-informed ML |
| 6 | [`qadence-kickstart-capstone-hybrid-advantage.ipynb`](notebooks/qadence-kickstart-105-capstone-hybrid-advantage.ipynb) | 12 | A hybrid `nn.Module` pipeline and a rigorous quantum-advantage evaluation |

### What each notebook covers

**1 — Tensors & autograd** *(Lesson 01)*
The seed notebook: gradients for free via `requires_grad`, a manual
forward→loss→backward→update loop, the same loop with `torch.optim`/`nn`, and an
extension to fitting a plane in 2-D feature space. Establishes the training
rhythm reused in every later notebook.

**2 — Feature maps & kernels** *(Lessons 02–03)*
Lifting data to gain linear separability; a proof-by-computation of the kernel
trick `⟨φ(x), φ(x′)⟩ = (1 + x·x′)²`; linear vs RBF Gram matrices; the
**kernel-concentration** failure mode (σ-sweep showing collapse to identity /
all-ones); a kernel nearest-centroid classifier; closed-form kernel ridge
regression on a noisy sine; the role of λ; and an SVM with a *precomputed* Gram
matrix — the exact interface QEK plugs into. *(8 figures.)*

**3 — Graph kernels & generalization** *(Lessons 04–05)*
Adjacency matrices and permutation invariance; the **Weisfeiler–Lehman** kernel
with a clear block-structured Gram matrix feeding the same SVM; an `n_iter` sweep
showing discriminative power grow; WL's blind spot (a 6-cycle vs two triangles —
non-isomorphic yet WL-identical, the opening for a quantum kernel); the
bias–variance U-curve; weight-decay regularization; honest k-fold
cross-validation; and **barren plateaus** shown collapsing side-by-side with
kernel concentration. *(10 figures.)*

**4 — Quantum kernels, neural nets & generative models** *(Lessons 06–08)*
A simulated quantum feature map (state as a `2ⁿ` vector via `kron`) visualized as
a Born-rule distribution; the quantum kernel `|⟨ψ|ψ′⟩|²` dropped into Lesson 03's
KRR unchanged; **exponential concentration** vs qubit count, with a 2-D
landscape over (encoding scale, qubits). *Enhancements:* a correction to the
"bandwidth sweet-spot" (maximizing mean overlap is a trap — maximize
*informativeness*); a data-dependent **ZZ/Ising coupling** closer to Rydberg
physics; and an honest quantum-vs-tuned-RBF bake-off. Then a multi-class neural
classifier with full decision regions, the activation-matters demo on concentric
circles (50% → 95%), categorical sampling / NLL / KL, an MMD generator
recovering a bimodal density, and a **categorical Born-machine** simulation.
*(16 figures.)*

**5 — Variational circuits, geometry & physics-informed ML** *(Lessons 09–11)*
Gates and CNOT as tensors; an encode→entangle→variational→measure classifier; the
encoding's **even-symmetry inductive bias** and how **data re-uploading** breaks
it to fit an odd target; the **parameter-shift rule** verified against autodiff
and *visualized* (the output is a sinusoid in each angle); **shot-noise**
gradients shrinking as 1/√shots. Then expressivity as a Fourier series (with an
empirical qubits→max-frequency spectrum), invariant/equivariant features, and a
symmetry-aware-vs-agnostic **generalization gap** (61% vs 100%). Finally
physics-informed solvers for `u′=u`, the harmonic oscillator, and a Gaussian —
capped by a **fully simulated quantum PINN** (single qubit, exact parameter-shift
derivatives) solving `u″+u=0` to within 6×10⁻⁴ of `sin(x)`. *(11 figures.)*

**6 — Capstone: hybrid pipelines & evaluating advantage** *(Lesson 12)*
*Part 1* implements the lesson faithfully: a hybrid `nn.Module`
(classical → quantum layer → classical head) trained on an XOR-like task, plus a
classical baseline. *Part 2* does it **properly** — a vectorized quantum layer;
held-out test sets with multi-seed confidence intervals and paired t-tests; a
**baseline ladder** showing the apparent quantum edge become statistically
insignificant against a strong classical net (p ≈ 0.09); the **exponential cost
of classical simulation** (extrapolated to 50 qubits ≈ 18 PB of memory);
**entanglement-entropy** and **concentration** simulability gates; a fair
quantum-kernel-vs-RBF learning-curve bake-off; and a final
**quantum-advantage scorecard**. The deliverable is the discipline to state,
honestly, that a laptop-simulable model is *not* a quantum advantage. *(4 figures.)*

> **Dependencies.** The kickstart notebooks use only `torch`, `numpy`,
> `matplotlib`, `scikit-learn`, `networkx`, and `scipy` — all provided by
> `uv sync`. No quantum hardware is required; everything runs on a laptop.

## Application notebooks (Qadence / Pasqal)

- [`notebooks/01_qadence_circuit_compilation.ipynb`](notebooks/01_qadence_circuit_compilation.ipynb)
  — the machinery underneath Qadence's `QuantumModel`: how an abstract,
  backend-agnostic `QuantumCircuit` is compiled into something a simulator or QPU
  can execute, and where differentiation hooks in.
- [`notebooks/portfolio_qaoa_pasqal.ipynb`](notebooks/portfolio_qaoa_pasqal.ipynb)
  — a cardinality-constrained portfolio-optimization QUBO solved as a
  proof-of-concept with Qadence digital-analog programs and PyTorch
  differentiable optimization.

## Study guide

The [`study-guide/`](study-guide/) directory contains the 12-lesson PyTorch → QML
curriculum the kickstart notebooks implement. Start with
[`00-STUDY-GUIDE.md`](study-guide/00-STUDY-GUIDE.md) for the full curriculum map,
learning objectives, and progress tracker. Each `lesson-NN-*.md` is a standalone
explanation with runnable code blocks.

## Reference docs

Background material in [`docs/`](docs/):

- [`pasqal-neutral-atom-master-reference.md`](docs/pasqal-neutral-atom-master-reference.md) — master reference on Pasqal's neutral-atom platform and toolchain.
- [`pasqal-plan.md`](personal/pasqal-plan.md) — project plan and roadmap.
- [`neutral-atom-qc-key-publications.md`](docs/neutral-atom-qc-key-publications.md) — annotated key publications.
- [`neutral-atom-qc-timeline.md`](docs/neutral-atom-qc-timeline.md) — milestone timeline for the field.
- [`Neutral-Atom-vs-Ion-vs-Superconducting-Comparison.md`](docs/Neutral-Atom-vs-Ion-vs-Superconducting-Comparison.md) — comparison of the leading qubit modalities.

## Requirements

- Python ≥ 3.12
- [uv](https://docs.astral.sh/uv/) for dependency management

## Setup

```bash
uv sync
```

This creates a virtual environment and installs Qadence and its dependencies
(including everything the kickstart notebooks need).

## Running the notebooks

Launch Jupyter inside the project environment:

```bash
uv run jupyter lab        # or: uv run jupyter notebook
```

then open any notebook under `notebooks/`. The kickstart notebooks ship with
their outputs already executed, so they also read well directly on disk or on
GitHub without running anything.

## Credentials

Azure Quantum and Pasqal credentials are read from the environment and are never
committed (see `.gitignore`). Copy `.env.example` to `.env` and configure your
Azure Quantum workspace and Pasqal access tokens locally before submitting jobs
to hardware. The kickstart notebooks do **not** require any credentials.

## License

Research use. © Dr. Boris Milanovic.
