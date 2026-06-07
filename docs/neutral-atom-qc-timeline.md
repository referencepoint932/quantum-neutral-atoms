# Neutral Atom Quantum Computing: A Timeline of Major Achievements

*A field overview from theoretical foundations to the logical-qubit era, with emphasis on Pasqal's contributions (Qadence, PyTorch integration, QEC, circuit compilation, QML).*

Last updated: June 2026

---

## The through-line

Neutral atom quantum computing rests on two ideas that had to mature before anything else could happen: a way to hold and address individual atoms (**optical tweezers**), and a way to make them interact conditionally (the **Rydberg blockade**). Once both were in hand, the field moved unusually fast — from trapping a single atom, to defect-free arrays of hundreds, to programmable quantum simulators, to genuine error-corrected logical processors and arrays exceeding 6,000 qubits. The architecture's defining advantage is reconfigurability: atoms can be physically moved mid-computation, which gives flexible connectivity and cheap parallelism that fixed-layout hardware cannot match. That advantage is now the platform's strongest card in the race toward fault tolerance.

---

## 2000–2009 — Theoretical foundations and single-atom control

**Rydberg blockade proposed (2000).** Jaksch, Zoller, Lukin and collaborators proposed using highly excited Rydberg states to mediate interactions between atoms. The mechanism: when one atom is excited to a Rydberg state, it shifts its neighbors out of resonance so they cannot also be excited. Within a small cluster, only one excitation is allowed — exactly the conditional logic a two-qubit gate requires.

**Single atoms in optical tweezers (early 2000s).** The Institut d'Optique group in France — including Georges-Olivier Reymond, later Pasqal's CEO — demonstrated that a tightly focused laser beam ("optical tweezer") could trap and image a single neutral atom. This established individually addressable qubits, the second pillar of the modality.

*Why it matters:* These two threads — a gate mechanism and isolatable qubits — define the entire neutral-atom approach. Everything afterward is engineering and scale.

---

## 2009–2015 — First gates and entanglement

The Wisconsin group (Mark Saffman) and the Institut d'Optique group (Antoine Browaeys) independently observed the **Rydberg blockade between two atoms** around 2009, and went on to demonstrate the first Rydberg-mediated two-qubit gates. An early two-atom CNOT landed near ~58% fidelity — primitive by modern standards, but a proof that the proposed physics actually produced entanglement.

*Why it matters:* Confirmed the blockade gate works in the lab, moving the field from theory to demonstrated entanglement.

---

## 2016 — The scaling breakthrough: defect-free arrays

The central obstacle was loading. Optical tweezers fill randomly, roughly 50% of the time, so assembling a large, fully ordered array by chance is statistically hopeless. In 2016, groups at Harvard (Endres, Bernien, Lukin), the Institut d'Optique (Barredo, Browaeys), and others solved it by **rearranging atoms one at a time into defect-free patterns** after the initial random load, using movable tweezers.

*Why it matters:* This is arguably the single most important enabling step for the modality. It is why neutral atoms now scale in qubit count faster than almost any competing technology — and the lab work behind it is the direct lineage from which **Pasqal** was spun out.

---

## 2017–2021 — Programmable quantum simulators

With ordered arrays solved, the field's first scientific payoff came from **analog quantum simulation** — using the Rydberg interaction Hamiltonian directly rather than compiling into gates.

- **51-atom simulator (2017).** Lukin's group (Bernien et al.) studied quantum dynamics on a 51-atom chain.
- **256-atom programmable simulator (2021).** Ebadi et al. (*Nature*) scaled to 256 qubits arranged in reconfigurable 2D lattices (square, honeycomb, triangular), mapping out quantum phases of matter with tunable interactions.

*Why it matters:* Demonstrated real scientific utility well before universal gate-based machines were ready. This analog / Rydberg-Hamiltonian regime is also Pasqal's commercial entry point — its early machines target simulation and optimization problems.

---

## 2019–2023 — High-fidelity gates and the turn toward digital

Gate quality climbed sharply and the platform picked up architectural features that made it competitive for universal computing.

- **Fidelity climb.** Optimized Rydberg gates passed ~97% by 2020.
- **Coherent transport of entangled atoms (2022).** Bluvstein et al. showed atoms could be physically shuttled across the array while preserving entanglement, enabling connectivity to be reconfigured mid-circuit.
- **Alkaline-earth atoms (Sr, Yb).** Adoption of these species enabled optical-clock-quality qubits and **erasure conversion**, a noise-management technique that turns hard-to-correct errors into detectable atom-loss events.
- **99.5% CZ gates (2023).** Evered et al. (Harvard) reported two-qubit CZ gates at 99.5% fidelity on up to 60 atoms in parallel — crossing the surface-code error-correction threshold.

*Why it matters:* Gate fidelity above the error-correction threshold, combined with mid-circuit atom transport, made fault-tolerant operation a realistic target rather than an aspiration.

---

## 2023–2025 — The logical-qubit era (where the field is now)

This stretch is the headline of the field.

- **48 logical qubits (Dec 2023).** Harvard / QuEra / MIT / NIST-UMD (Bluvstein et al., *Nature*): the first programmable *logical* processor. Around 280 physical rubidium atoms encoded dozens of error-corrected logical qubits running algorithms with hundreds of logical operations. Critically, the team showed that **increasing the code distance lowered the error rate** — the signature behavior required for error correction to actually pay off.
- **Atom Computing past 1,000 qubits (late 2023).** A 1,225-site array populated with 1,180 physical qubits — the first universal platform to exceed the 1,000-qubit mark.
- **Logical magic-state distillation (July 2025).** QuEra / Harvard / MIT (*Nature*), on QuEra's Gemini system: the first magic-state distillation performed entirely on logical qubits, using distance-3 and distance-5 color codes and a 5-to-1 protocol whose output fidelity beat any input. Magic states are a required ingredient for *universal* fault tolerance, not just error detection.
- **Caltech 6,100-atom array (Sept 2025).** A record cesium array of ~6,100 atoms in ~12,000 tweezer sites, with ~13-second coherence times (roughly 10× prior tweezer systems) and 99.98% single-qubit fidelity, including atom transport across hundreds of micrometers with superposition preserved.
- **High-rate codes, late 2025.** Reporting toward ~90+ verified logical qubits on a few hundred physical atoms using high-rate codes — pointing at the clear trend of more logical qubits squeezed per physical atom. *(Treat exact late-2025 counts as provisional versus the peer-reviewed 48-logical-qubit and magic-state-distillation results.)*

*Why it matters:* The field crossed from "we can entangle qubits" to "we can run error-corrected logical algorithms," while simultaneously demonstrating raw scale in the thousands of qubits.

---

## Pasqal in focus

Pasqal is the European leader in the modality, spun directly from the Institut d'Optique tweezer/Rydberg lineage. Its distinctive bets map closely onto this project's themes.

### Hardware and scaling
The **Orion** platform line has delivered machines with over 100 qubits to customers, including the first deployments of neutral-atom QPUs inside HPC centers — France's GENCI and Germany's Forschungszentrum Jülich. Pasqal has **exceeded 1,000 atoms** in a processor, and its public roadmap targets roughly 1,000 physical qubits plus first logical qubits in 2025, scaling through later generations (Vela, Centaurus) toward early fault-tolerant systems by the end of the decade, with a stated ambition of 10,000 qubits.

### Digital-analog quantum computing (DAQC)
Rather than treating analog and gate-based computing as separate paradigms, Pasqal pushes a hybrid model that interleaves the native Rydberg Hamiltonian (analog) with digital gates. The goal is more precision than pure analog and less overhead than pure digital.

### Qadence and the PyTorch / QML stack
**Qadence** is Pasqal's open-source library for building digital-analog quantum programs, using a block-based interface (inspired by Yao). Its defining feature for quantum machine learning is **native integration with PyTorch automatic differentiation**, so quantum models slot directly into a standard PyTorch workflow.

- **PyQTorch** — the default backend, a PyTorch-based differentiable state-vector and density-matrix simulator.
- **Horqrux** — a JAX-based backend alternative aimed at QML.
- Implements **parameter-shift rules** (including higher-order) for training variational models, including on real neutral-atom hardware.

This stack is the practical foundation for QML on the platform: it makes quantum circuits behave like differentiable layers that gradient-based optimizers can train.

### QEC and circuit compilation
Pasqal is racing the US groups on error correction, with a roadmap that moves explicitly from a couple of logical qubits toward scalable logical architectures. The platform leans on neutral atoms' native advantages — atom shuttling for flexible connectivity, near all-to-all reconfiguration, and cheap parallelism — which make certain QEC codes and compilation strategies cheaper to implement than on fixed-layout hardware.

---

## At a glance

| Year | Milestone | Who |
|------|-----------|-----|
| 2000 | Rydberg blockade proposed | Jaksch, Zoller, Lukin et al. |
| ~2001 | Single atom trapped in optical tweezer | Institut d'Optique (incl. G.-O. Reymond) |
| ~2009–2010 | Blockade between two atoms; first Rydberg gates (~58% CNOT) | Wisconsin (Saffman); Institut d'Optique (Browaeys) |
| 2016 | Defect-free atom-by-atom array assembly | Harvard; Institut d'Optique; others |
| 2017 | 51-atom programmable simulator | Lukin group |
| 2021 | 256-atom programmable simulator | Ebadi et al. (Harvard/MIT) |
| 2022 | Coherent transport of entangled atoms | Bluvstein et al. |
| 2023 | 99.5% parallel CZ gates (above QEC threshold) | Evered et al. (Harvard) |
| 2023 | 48 logical qubits, first logical processor | Harvard/QuEra/MIT/NIST |
| 2023 | First platform past 1,000 qubits (1,180) | Atom Computing |
| 2024–25 | >100-qubit Orion QPUs in HPC centers; >1,000 atoms | Pasqal |
| Jul 2025 | Logical magic-state distillation | QuEra/Harvard/MIT |
| Sep 2025 | 6,100-atom array, ~13 s coherence, 99.98% 1Q fidelity | Caltech |

---

## Sources

- [Pasqal — Quantum Computing History: Path to Pasqal](https://www.pasqal.com/quantum-computing-history-path-to-pasqal/)
- [Neutral atom quantum computer — Wikipedia](https://en.wikipedia.org/wiki/Neutral_atom_quantum_computer)
- [Harvard Gazette — first logical quantum processor (48 logical qubits)](https://news.harvard.edu/gazette/story/2023/12/researchers-create-first-logical-quantum-processor/)
- [QuEra — 48 logical qubits press release](https://www.quera.com/press-releases/harvard-quera-mit-and-the-nist-university-of-maryland-usher-in-new-era-of-quantum-computing-by-performing-complex-error-corrected-quantum-algorithms-on-48-logical-qubits0)
- [Nature / arXiv — Quantum phases on a 256-atom programmable simulator](https://arxiv.org/pdf/2012.12281)
- [Atom Computing — 1,180-qubit processor (HPCwire)](https://www.hpcwire.com/2023/10/24/atom-computing-wins-the-race-to-1000-qubits/)
- [Evered et al. — high-fidelity parallel entangling gates (PubMed)](https://pubmed.ncbi.nlm.nih.gov/37821591/)
- [Caltech — 6,100-qubit array record](https://www.caltech.edu/about/news/caltech-team-sets-record-with-6100-qubit-array)
- [Nature — experimental logical magic state distillation (2025)](https://www.nature.com/articles/s41586-025-09367-3)
- [Pasqal — 2025 roadmap toward fault tolerance](https://www.pasqal.com/newsroom/pasqal-releases-2025-roadmap/)
- [Pasqal — exceeds 1,000 atoms in processor](https://www.pasqal.com/newsroom/pasqal-exceeds-1000-atoms-in-quantum-processor/)
- [Pasqal — Qadence library](https://www.pasqal.com/newsroom/pasqal-unveils-qadence-a-quantum-programming-library/)
- [pyqtorch — GitHub](https://github.com/pasqal-io/pyqtorch)
