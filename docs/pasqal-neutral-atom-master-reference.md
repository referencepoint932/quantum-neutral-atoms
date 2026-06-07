# Pasqal & Neutral Atom Quantum Computing — Master Reference

*A single merged reference combining (I) how Pasqal's machines operate, step-by-step; (II) the field's historical timeline; and (III) the key publications and optical-tweezers lineage. Pasqal-weighted throughout.*

Last updated: June 2026

---

## Contents

1. [The one-paragraph picture](#0-the-one-paragraph-picture)
2. [Part I — How Pasqal operates, step-by-step](#part-i--how-pasqal-operates-step-by-step)
3. [Part II — Field timeline of major achievements](#part-ii--field-timeline-of-major-achievements)
4. [Part III — Key publications & the optical-tweezers lineage](#part-iii--key-publications--the-optical-tweezers-lineage)
5. [Master reference index](#master-reference-index)

---

## 0. The one-paragraph picture

A Pasqal quantum computer is, at heart, an optical machine. It laser-cools a cloud of neutral atoms, then uses **optical tweezers** — tightly focused laser beams — to pick out individual atoms and arrange them into a programmable pattern, with one atom per qubit. Computation happens by shining precisely shaped laser pulses on those atoms: briefly exciting them to high-energy **Rydberg states** makes neighboring atoms interact strongly and conditionally (the **Rydberg blockade**), which is the resource for both analog simulation and digital entangling gates. The answer is read out by photographing which atoms glow. The whole field rests on two inventions — the optical tweezer (1986, Nobel 2018) and the Rydberg-blockade gate (proposed 2000–2001) — and Pasqal descends directly from the French lab that first trapped single atoms this way.

---

# Part I — How Pasqal operates, step-by-step

Pasqal builds **neutral-atom quantum processing units (QPUs)**: each qubit is a single neutral atom (commonly rubidium; ytterbium, strontium, and caesium are also used across the industry) held in light. A computation runs through the following cycle.

### Step 1 — Source and laser-cool the atoms (the MOT)
A small cloud of neutral atoms is captured and cooled to microkelvin temperatures in a **magneto-optical trap (MOT)**, using counter-propagating lasers and a magnetic field gradient. Cold atoms move slowly enough to be caught one at a time in the next step. *Note:* because the atoms are isolated in vacuum and controlled by light, the qubits themselves sit at ultracold temperature while the surrounding apparatus runs largely at room temperature — no dilution refrigerator is required.

### Step 2 — Create the tweezer array (the register)
A **spatial light modulator (SLM)** splits one trapping laser into hundreds of microscopic **optical tweezers**, each a separate trap, arranged in a user-defined 1D, 2D, or 3D geometry. This pattern of traps is the **register** — the physical layout of the qubits. Crucially, the geometry is programmable: the spacing and arrangement of atoms directly sets which qubits will interact and how strongly.

### Step 3 — Load atoms stochastically
Each tweezer captures **either zero or one atom**, roughly half the time, enforced by the **collisional blockade** effect (a second atom entering a tiny trap ejects the pair). The result is a randomly half-filled array — not yet usable.

### Step 4 — Image and rearrange into a defect-free pattern
A camera takes a fluorescence image to see which traps are filled. **Moving tweezers** (steered by acousto-optic deflectors, AODs) then physically shuttle atoms one at a time to fill the target pattern, producing a **defect-free register**. This atom-by-atom assembly (the 2016 breakthrough, Section II) is what makes large ordered arrays possible at all; Pasqal has assembled defect-free registers of several hundred atoms.

### Step 5 — Encode the qubit
The two qubit states |0⟩ and |1⟩ are stored in atomic electronic energy levels. Two encodings are used depending on mode:
- **Ground–Rydberg basis** (analog): |0⟩ = ground state, |1⟩ = a high-lying Rydberg state.
- **Two hyperfine ground states** (digital): long-lived, stable states used as the logical qubit, with the Rydberg state invoked only transiently to make atoms interact.

### Step 6 — Initialize
**Optical pumping** prepares every atom in a well-defined starting state (|0⟩), giving a clean, known input.

### Step 7 — Compute (the laser pulses do the work)
Pasqal hardware supports two complementary modes on the **same machine**:

- **Analog mode (Hamiltonian computing).** A global laser drives all atoms together. Because atoms in Rydberg states interact strongly, the system naturally evolves under a tunable **Ising-like Rydberg Hamiltonian**. The programmer shapes the answer through three knobs over time — laser **amplitude, detuning, and phase** — plus the **register geometry**, which fixes the interaction graph. This is the native way to run quantum simulation, combinatorial optimization (e.g. QUBO / Maximum Independent Set, where the graph is encoded directly in atom positions), and differential-equation solvers.
- **Digital / digital-analog mode (gate computing).** **Single-qubit gates** are applied with tightly focused addressing beams that hit individual atoms; **two-qubit entangling gates** use the Rydberg blockade between a pair. Atoms can also be **physically shuttled mid-circuit** to change which qubits are adjacent, giving flexible connectivity. **Digital-analog (DAQC)** interleaves these gates with native analog Hamiltonian blocks to get digital precision with analog efficiency.

### Step 8 — Read out (photograph the answer)
Qubits are measured by **state-selective fluorescence imaging**: a laser drives a cycling transition so that one qubit state scatters many photons and appears **bright** on the camera while the other stays **dark**. A single camera frame reads the entire register **in parallel**. Readout fidelity is currently ~98–99%, limited mainly by stray photon scattering and occasional atom loss.

### Step 9 — Repeat for statistics (shots)
Quantum measurement is probabilistic, so a single run gives one bitstring. The full cycle (Steps 3–8) is **repeated many times** to build up probability distributions and expectation values — the actual computational output.

### Step 10 — Close the hybrid loop (for variational / QML work)
For variational algorithms and quantum machine learning, a **classical optimizer** reads those statistics, updates the pulse or gate parameters, and the quantum cycle runs again. This loop is what trains a model or minimizes a cost function.

### Pasqal's software stack (how a user actually programs it)

The hardware is reached through a layered, open-source stack — this is where the project's themes of QML, compilation, and PyTorch integration live:

| Layer | Tool | Role |
|-------|------|------|
| Low-level pulses | **Pulser** | Open-source Python SDK to define the register and pulse sequences on hardware channels; covers analog and digital-analog control, with a built-in simulator. |
| No-code | **Pulser Studio** | Graphical, zero-code front end for building pulse sequences. |
| High-level / QML | **Qadence** | Block-based library for digital-analog programs; **natively PyTorch-differentiable**, so quantum models drop into ML workflows with parameter-shift gradients (incl. higher order) usable on real hardware. |
| Simulator backends | **PyQTorch** (PyTorch) / **Horqrux** (JAX) | Differentiable state-vector / density-matrix simulators that back Qadence for training and emulation. |
| Access | Cloud + HPC | Available via cloud providers and, distinctively, **deployed inside HPC centers** (France's GENCI, Germany's Jülich) alongside supercomputers. |

### Hardware lineup and roadmap (operational context)

Pasqal's QPU generations include the **Fresnel** line and the **Orion** platform (Alpha / Beta / Gamma). It has delivered **>100-qubit** machines to customers, assembled registers of several hundred atoms, and **exceeded 1,000 atoms** in a processor. The public roadmap targets ~1,000 physical qubits with first logical qubits in 2025, larger generations (**Vela**, ~200+ qubits; **Centaurus**, early fault-tolerant) later this decade, and an ambition toward 10,000 qubits — leaning on neutral atoms' native advantages (atom shuttling for flexible connectivity, near all-to-all reconfiguration, cheap parallelism) for error correction and circuit compilation.

---

# Part II — Field timeline of major achievements

The arc: hold and address a single atom (tweezers) → make atoms interact conditionally (Rydberg blockade) → assemble defect-free arrays → run programmable simulators → reach high-fidelity gates → run error-corrected *logical* processors and thousand-atom arrays. The defining advantage that emerged is **reconfigurability** — atoms can be moved during a computation — which is now the platform's strongest card in the fault-tolerance race.

### 2000–2009 — Theoretical foundations and single-atom control
- **Rydberg blockade proposed (2000–2001).** Jaksch et al. and Lukin et al. showed that briefly exciting atoms to Rydberg states gives the strong, conditional interaction a two-qubit gate needs.
- **Single atoms in optical tweezers (2001).** The Institut d'Optique group (incl. Pasqal's future CEO Georges-Olivier Reymond) trapped and imaged one atom in a focused beam — the second pillar of the modality.

### 2009–2015 — First gates and entanglement
The Wisconsin (Saffman) and Institut d'Optique (Browaeys) groups observed the **blockade between two atoms** (2009) and built the first Rydberg gates, including a **two-atom CNOT** (~0.73 fidelity, 2010). Primitive, but proof the physics computed.

### 2016 — The scaling breakthrough: defect-free arrays
Tweezers load randomly (~50%), so large ordered arrays are statistically hopeless by chance. Harvard (Endres) and Institut d'Optique (Barredo) solved it by **rearranging atoms one at a time** into perfect patterns — the single most important enabling step, and the lab work from which **Pasqal** spun out.

### 2017–2021 — Programmable quantum simulators
- **51-atom simulator (2017, Bernien et al.).** First frontier analog-simulation result.
- **256-atom simulator (2021, Ebadi et al.).** Programmable 2D lattices probing quantum phases of matter.
- **196-atom 2D antiferromagnet (2021, Scholl et al.).** The Browaeys/Pasqal-lineage companion at hundreds-of-atoms scale.

### 2019–2023 — High-fidelity gates, transport, and the turn to digital
- **Coherent transport of entangled atoms (2022, Bluvstein et al.).** Shuttling qubits mid-circuit gives dynamic connectivity.
- **Alkaline-earth atoms (Sr, Yb).** Enabled optical-clock qubits and **erasure conversion** (turning errors into detectable atom-loss events).
- **99.5% CZ gates (2023, Evered et al.).** Parallel two-qubit gates crossing the error-correction threshold.

### 2023–2025 — The logical-qubit era (where the field is now)
- **48 logical qubits (2024 in *Nature*, Bluvstein et al.).** First programmable *logical* processor; showed that larger code distance lowers logical error — the signature that error correction pays off.
- **>1,000 qubits (late 2023, Atom Computing).** First universal platform past 1,000.
- **Logical magic-state distillation (2025).** Required ingredient for *universal* fault tolerance, done entirely on logical qubits.
- **6,100-atom array (Sept 2025, Caltech).** ~13 s coherence, 99.98% single-qubit fidelity; record raw scale.
- *Late-2025 high-rate-code results pushing toward ~90+ logical qubits are still settling — treat exact counts as provisional vs. the peer-reviewed 48-logical-qubit and magic-state results.*

---

# Part III — Key publications & the optical-tweezers lineage

Peer-reviewed, high-impact papers, each tied to the capability it gave quantum computing. The two best newcomer entry points are the **review articles** (R1–R3) and Pasqal's own review (#16).

### Theoretical foundations
- **Jaksch et al. (2000), *Fast Quantum Gates for Neutral Atoms*, PRL 85, 2208.** [APS](https://link.aps.org/doi/10.1103/PhysRevLett.85.2208) · [arXiv](https://arxiv.org/abs/quant-ph/0004038) — *Enabled:* the Rydberg-based two-qubit gate (theory).
- **Lukin et al. (2001), *Dipole Blockade and QIP in Mesoscopic Atomic Ensembles*, PRL 87, 037901.** [APS](https://link.aps.org/doi/10.1103/PhysRevLett.87.037901) — *Enabled:* the blockade as the platform's core control primitive.

### Optical tweezers — the trapping technology (deep dive)
This lineage is why tweezers appear everywhere: every qubit is one atom in one focused beam.
- **Ashkin (1970), *Acceleration and Trapping of Particles by Radiation Pressure*, PRL 24, 156.** [APS](https://link.aps.org/doi/10.1103/PhysRevLett.24.156) — *Enabled:* the founding idea that light can mechanically trap matter.
- **Ashkin, Dziedzic, Bjorkholm, Chu (1986), *Observation of a single-beam gradient force optical trap*, Optics Letters 11, 288.** [Optica](https://opg.optica.org/ol/abstract.cfm?uri=ol-11-5-288) — *Enabled:* the modern optical tweezer itself. **(Arthur Ashkin, 2018 Nobel Prize in Physics — [Nobel](https://www.nobelprize.org/prizes/physics/2018/ashkin/facts/).)**
- **Schlosser, Reymond, Protsenko, Grangier (2001), *Sub-poissonian loading of single atoms in a microscopic dipole trap*, Nature 411, 1024.** [Nature](https://www.nature.com/articles/35082512) — *Enabled:* deterministic one-atom-per-trap loading via collisional blockade — the direct experimental ancestor of Pasqal's hardware (and its CEO is a co-author).
- **Kaufman & Ni (2021), *Quantum science with optical tweezer arrays…*, Nature Physics 17, 1324.** *(review)* [Nature Physics](https://www.nature.com/articles/s41567-021-01357-2) — *Best single source* on how tweezer arrays became a scalable quantum platform.

*The thread:* radiation pressure (1970) → the gradient-force trap (1986) → one atom per beam (2001) → arrays of hundreds-to-thousands of tweezers, each a qubit.

### First experimental gates and entanglement
- **Urban et al. (2009), Nature Physics 5, 110** [Nature Physics](https://www.nature.com/articles/nphys1178) and **Gaëtan et al. (2009), Nature Physics 5, 115** [Nature Physics](https://www.nature.com/articles/nphys1183) — *Enabled:* first experimental confirmation of the two-atom blockade.
- **Isenhower et al. (2010), *Demonstration of a Neutral Atom Controlled-NOT Quantum Gate*, PRL 104, 010503.** [PubMed](https://pubmed.ncbi.nlm.nih.gov/20366355/) — *Enabled:* the first working neutral-atom CNOT (with companion entanglement paper Wilk et al., PRL 104, 010502).

### The scaling breakthrough — defect-free arrays (2016)
- **Endres et al. (2016), *Atom-by-atom assembly… 1D*, Science 354, 1024.** [Science](https://www.science.org/doi/10.1126/science.aah3752)
- **Barredo et al. (2016), *…assembler of defect-free arbitrary 2D atomic arrays*, Science 354, 1021.** [Science](https://www.science.org/doi/10.1126/science.aah3778) — *Enabled:* arbitrary defect-free arrays — the prerequisite for everything that scaled afterward, and the basis of Pasqal's hardware.

### Programmable quantum simulators (2017–2021)
- **Bernien et al. (2017), *51-atom quantum simulator*, Nature 551, 579.** [Nature](https://www.nature.com/articles/nature24622) — *Enabled:* frontier analog simulation.
- **Ebadi et al. (2021), *256-atom programmable quantum simulator*, Nature 595, 227.** [Nature](https://www.nature.com/articles/s41586-021-03582-4) — *Enabled:* 256-qubit programmable simulation of quantum phases.
- **Scholl et al. (2021), *2D antiferromagnets with hundreds of Rydberg atoms*, Nature 595, 233.** [Nature](https://www.nature.com/articles/s41586-021-03585-1) — *Enabled:* hundreds-scale simulation on the Pasqal/Institut d'Optique line (196 atoms).

### High-fidelity gates, transport, and the logical era
- **Bluvstein et al. (2022), *Coherent transport of entangled atom arrays*, Nature 604, 451.** [Nature](https://www.nature.com/articles/s41586-022-04592-6) — *Enabled:* dynamic, non-local connectivity by moving qubits.
- **Evered et al. (2023), *High-fidelity parallel entangling gates*, Nature 622, 268.** [Nature](https://www.nature.com/articles/s41586-023-06481-y) — *Enabled:* gate fidelity (99.5%) above the QEC threshold.
- **Bluvstein et al. (2024), *Logical quantum processor based on reconfigurable atom arrays*, Nature 626, 58.** [Nature](https://www.nature.com/articles/s41586-023-06927-3) · [arXiv](https://arxiv.org/pdf/2312.03982) — *Enabled:* 48 error-corrected logical qubits and the demonstration that distance scaling lowers logical error.
- **QuEra/Harvard/MIT (2025), *Experimental demonstration of logical magic state distillation*, Nature.** [Nature](https://www.nature.com/articles/s41586-025-09367-3) — *Enabled:* a required ingredient for universal fault tolerance, on logical qubits.

### Pasqal-focused publications & software
- **Henriet et al. (2020), *Quantum computing with neutral atoms*, Quantum 4, 327.** [Quantum](https://quantum-journal.org/papers/q-2020-09-21-327/) · [arXiv](https://arxiv.org/abs/2006.12326) — Pasqal's foundational review; *best Pasqal-specific starting point.*
- **Seitz et al. (2024), *Qadence: a differentiable interface for digital-analog programs*, arXiv:2401.09915 (IEEE Software, 2025).** [arXiv](https://arxiv.org/abs/2401.09915) · [Pasqal](https://www.pasqal.com/newsroom/qadence-a-differentiable-interface-for-digital-analog-programs/) — *Enabled:* a PyTorch-differentiable software bridge between neutral-atom hardware and ML workflows (QML, compilation, DAQC).
- **PyQTorch (Pasqal, open source).** [GitHub](https://github.com/pasqal-io/pyqtorch) — *Enabled:* gradient-based training/simulation of digital-analog circuits in PyTorch.
- **Pulser (Pasqal, open source).** [Docs](https://docs.pasqal.com/pulser/) — pulse-level SDK for analog/digital-analog control (the operational entry point in Part I).

### Best review articles to start with
- **R1 — Saffman, Walker, Mølmer (2010), *Quantum information with Rydberg atoms*, Rev. Mod. Phys. 82, 2313.** [APS](https://link.aps.org/doi/10.1103/RevModPhys.82.2313) · [arXiv](https://arxiv.org/abs/0909.4777) — the canonical theory review.
- **R2 — Browaeys & Lahaye (2020), *Many-body physics with individually controlled Rydberg atoms*, Nature Physics 16, 132.** [Nature Physics](https://www.nature.com/articles/s41567-019-0733-z) — the Rydberg many-body/simulation review.
- **R3 — Kaufman & Ni (2021).** *(see optical-tweezers section)* — the tweezer-platform review.

---

## Master reference index

| # | Paper / tool | Year | Venue | Tie to quantum computing |
|---|--------------|------|-------|--------------------------|
| 1 | Jaksch et al., *Fast quantum gates* | 2000 | PRL | Rydberg two-qubit gate (theory) |
| 2 | Lukin et al., *Dipole blockade* | 2001 | PRL | The blockade control primitive |
| 3 | Ashkin, *Radiation pressure* | 1970 | PRL | Light can trap matter |
| 4 | Ashkin et al., *Gradient-force trap* | 1986 | Opt. Lett. | The optical tweezer (Nobel 2018) |
| 5 | Schlosser et al., *Sub-Poissonian loading* | 2001 | Nature | One atom per trap (Pasqal lineage) |
| 6 | Urban / Gaëtan et al., *Blockade observed* | 2009 | Nat. Phys. | Blockade confirmed in the lab |
| 7 | Isenhower et al., *Neutral-atom CNOT* | 2010 | PRL | First neutral-atom CNOT |
| 8 | Endres & Barredo et al., *Defect-free arrays* | 2016 | Science | Scalable ordered qubit arrays |
| 9 | Bernien et al., *51-atom simulator* | 2017 | Nature | Frontier analog simulation |
| 10 | Ebadi et al., *256-atom simulator* | 2021 | Nature | 256-qubit programmable simulation |
| 11 | Scholl et al., *196-atom antiferromagnet* | 2021 | Nature | Hundreds-scale sim (Pasqal lineage) |
| 12 | Bluvstein et al., *Coherent transport* | 2022 | Nature | Reconfigurable connectivity |
| 13 | Evered et al., *99.5% CZ gates* | 2023 | Nature | Above-threshold gate fidelity |
| 14 | Bluvstein et al., *Logical processor* | 2024 | Nature | 48 error-corrected logical qubits |
| 15 | QuEra/Harvard/MIT, *Magic-state distillation* | 2025 | Nature | Toward universal fault tolerance |
| 16 | Henriet et al., *QC with neutral atoms* | 2020 | Quantum | Pasqal field review |
| 17 | Seitz et al., *Qadence* | 2024 | arXiv/IEEE Sw. | PyTorch-differentiable DAQC software |
| 18 | Pulser | — | open source | Pulse-level operational SDK |
| R1 | Saffman, Walker, Mølmer | 2010 | RMP | Canonical theory review |
| R2 | Browaeys & Lahaye | 2020 | Nat. Phys. | Many-body Rydberg review |
| R3 | Kaufman & Ni | 2021 | Nat. Phys. | Tweezer-array review |

*Companion files in this project: `neutral-atom-qc-timeline.md` (timeline only) and `neutral-atom-qc-key-publications.md` (publications only). This document supersedes both by merging them with the operational walkthrough.*
