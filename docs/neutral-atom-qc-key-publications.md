# Neutral Atom Quantum Computing: Key Publications & the Optical-Tweezers Lineage

*An annotated bibliography of the field's most credible and significant papers, each tied to what it enabled for quantum computing. Includes a dedicated deep-dive on optical tweezers, the foundational trapping technology. Pasqal-weighted.*

Last updated: June 2026

---

## How to read this

Entries are grouped by role in the field's development. "Credible and significant" here means peer-reviewed papers in top venues (*Nature*, *Science*, *Physical Review Letters*, *Reviews of Modern Physics*, *Quantum*) that are either the original demonstration of a capability or the canonical review of an area. Each entry states **what it enabled** — the concrete capability quantum computing gained from it. The two best starting points for a newcomer are the review articles flagged in the final section.

---

## 1. Theoretical foundations — the two ideas the whole field rests on

**Jaksch, Cirac, Zoller, Rolston, Côté, Lukin (2000). "Fast Quantum Gates for Neutral Atoms." *Physical Review Letters* 85, 2208.**
[APS](https://link.aps.org/doi/10.1103/PhysRevLett.85.2208) · [arXiv:quant-ph/0004038](https://arxiv.org/abs/quant-ph/0004038)
The proposal that started gate-based neutral-atom computing. It showed that the strong dipole–dipole interaction between atoms briefly excited to high-lying **Rydberg states** could drive a fast two-qubit gate, even though ground-state neutral atoms barely interact at all. *Enabled:* a concrete, fast entangling-gate mechanism for a qubit type that had previously seemed too weakly interacting to compute with.

**Lukin, Fleischhauer, Côté, Duan, Jaksch, Cirac, Zoller (2001). "Dipole Blockade and Quantum Information Processing in Mesoscopic Atomic Ensembles." *Physical Review Letters* 87, 037901.**
[APS](https://link.aps.org/doi/10.1103/PhysRevLett.87.037901)
Introduced the **Rydberg/dipole blockade** as a resource for deterministic entanglement: once one atom is excited to a Rydberg state, it detunes its neighbors so they cannot be excited too. *Enabled:* the central control primitive of the platform — collective, conditional logic across a small group of atoms — which every later gate and many simulators rely on.

---

## 2. Optical tweezers — the trapping technology (deep dive)

You noticed optical tweezers everywhere because they are the physical substrate of the entire modality: each qubit is a single atom held in a focused laser beam. This lineage runs from 1970s optics through a 2018 Nobel Prize to today's thousand-atom arrays.

**Ashkin (1970). "Acceleration and Trapping of Particles by Radiation Pressure." *Physical Review Letters* 24, 156.**
[APS](https://link.aps.org/doi/10.1103/PhysRevLett.24.156)
The first demonstration that laser light alone could push and trap micron-sized particles. *Enabled:* the founding idea that light can exert usable mechanical force on matter — the seed of all optical trapping.

**Ashkin, Dziedzic, Bjorkholm, Chu (1986). "Observation of a single-beam gradient force optical trap for dielectric particles." *Optics Letters* 11, 288.**
[Optica](https://opg.optica.org/ol/abstract.cfm?uri=ol-11-5-288)
The invention of the modern **optical tweezer**: a single tightly focused beam whose intensity gradient traps a particle at its focus. *Enabled:* the exact trap geometry later used to hold one atom per beam. This work earned **Arthur Ashkin the 2018 Nobel Prize in Physics** ([Nobel citation](https://www.nobelprize.org/prizes/physics/2018/ashkin/facts/)).

**Schlosser, Reymond, Protsenko, Grangier (2001). "Sub-poissonian loading of single atoms in a microscopic dipole trap." *Nature* 411, 1024.**
[Nature](https://www.nature.com/articles/35082512)
The Institut d'Optique group — including **Georges-Olivier Reymond, later Pasqal's CEO** — showed that a sufficiently small dipole trap loads exactly *one* atom at a time via the **collisional blockade** effect (a second atom always knocks the pair out). *Enabled:* deterministic single-atom qubits with clean, sub-Poissonian statistics — the direct experimental ancestor of Pasqal's hardware.

**Kaufman & Ni (2021). "Quantum science with optical tweezer arrays of ultracold atoms and molecules." *Nature Physics* 17, 1324.** *(review)*
[Nature Physics](https://www.nature.com/articles/s41567-021-01357-2)
The definitive modern review of tweezer-array science across computing, simulation, and metrology. *Best single source* for understanding how tweezers became a scalable quantum platform rather than a single-trap curiosity. *Enabled (as a reference):* a unified picture of the field for newcomers and a map of where tweezer arrays are heading.

*The thread:* radiation pressure (1970) → the single-beam gradient trap (1986) → trapping one atom per beam (2001) → arrays of hundreds-to-thousands of tweezers, each a qubit (Section 4 onward).

---

## 3. First experimental gates and entanglement

**Urban, Johnson, Henage, Isenhower, Yavuz, Walker, Saffman (2009). "Observation of Rydberg blockade between two atoms." *Nature Physics* 5, 110.**
[Nature Physics](https://www.nature.com/articles/nphys1178)
**Gaëtan, Miroshnychenko, Wilk, Chotia, Viteau, Comparat, Pillet, Browaeys, Grangier (2009). "Observation of collective excitation of two individual atoms in the Rydberg blockade regime." *Nature Physics* 5, 115.**
[Nature Physics](https://www.nature.com/articles/nphys1183)
Two independent groups (Wisconsin and Institut d'Optique) gave the first direct experimental confirmation of the blockade between two individually trapped atoms. *Enabled:* proof that the 2000–2001 theory worked in real apparatus — the green light for building gates on it.

**Isenhower, Urban, Zhang, Gill, Henage, Johnson, Walker, Saffman (2010). "Demonstration of a Neutral Atom Controlled-NOT Quantum Gate." *Physical Review Letters* 104, 010503.**
[PubMed](https://pubmed.ncbi.nlm.nih.gov/20366355/)
The first **CNOT gate** between two individually addressed neutral atoms (with a companion paper, Wilk et al., PRL 104, 010502, demonstrating entanglement). Fidelities were modest (~0.73 truth-table; Bell-state ~0.48–0.58 after loss correction). *Enabled:* a working universal-gate primitive on neutral atoms — crude, but the proof of principle the field built on for the next decade.

---

## 4. The scaling breakthrough — defect-free arrays (2016)

**Endres, Bernien, Keesling, Levine, Anschuetz, Krajenbrink, Senko, Vuletić, Greiner, Lukin (2016). "Atom-by-atom assembly of defect-free one-dimensional cold atom arrays." *Science* 354, 1024.**
[Science](https://www.science.org/doi/10.1126/science.aah3752)
**Barredo, de Léséleuc, Lienhard, Lahaye, Browaeys (2016). "An atom-by-atom assembler of defect-free arbitrary two-dimensional atomic arrays." *Science* 354, 1021.**
[Science](https://www.science.org/doi/10.1126/science.aah3778)
Tweezers load randomly (~50% per site), so a large ordered array is statistically near-impossible by chance. These two papers solved it: image the random load, then **rearrange atoms one at a time with movable tweezers** into a perfect target pattern. *Enabled:* arbitrary, defect-free arrays of dozens-to-hundreds of qubits — arguably the single most important enabling step for the modality, and the lab work from which **Pasqal** was spun out.

---

## 5. Programmable quantum simulators (2017–2021)

**Bernien, Schwartz, Keesling, Levine, Omran, Pichler, Choi, Zibrov, Endres, Greiner, Vuletić, Lukin (2017). "Probing many-body dynamics on a 51-atom quantum simulator." *Nature* 551, 579.**
[Nature](https://www.nature.com/articles/nature24622)
A programmable Ising-type spin model on 51 atoms, which also revealed unexpected coherent "quantum scar" dynamics. *Enabled:* the platform's first major scientific result — analog quantum simulation at a scale beyond easy classical reach.

**Ebadi, Wang, Levine, Keesling, Semeghini, Omran, Bluvstein, Samajdar, Pichler, Ho, Choi, Sachdev, Greiner, Vuletić, Lukin (2021). "Quantum phases of matter on a 256-atom programmable quantum simulator." *Nature* 595, 227.**
[Nature](https://www.nature.com/articles/s41586-021-03582-4)
Scaled programmable simulation to 256 qubits in reconfigurable 2D lattices. *Enabled:* exploration of quantum phase transitions at a scale and tunability that made neutral atoms a leading simulation platform.

**Scholl, Schuler, Williams, Eberharter, Barredo, Schymik, Lienhard, Henry, Lang, Lahaye, Läuchli, Browaeys (2021). "Quantum simulation of 2D antiferromagnets with hundreds of Rydberg atoms." *Nature* 595, 233.** *(Institut d'Optique / Pasqal lineage)*
[Nature](https://www.nature.com/articles/s41586-021-03585-1)
The Browaeys-group companion result: a 2D transverse-field Ising antiferromagnet with up to **196 atoms** at high fidelity. *Enabled:* demonstration that the European/Pasqal hardware line could do frontier many-body simulation at the hundreds-of-atoms scale.

---

## 6. High-fidelity gates, transport, and the logical-qubit era

**Bluvstein, Levine, Semeghini, Wang, Ebadi, Kalinowski, Keesling, Maskara, Pichler, Greiner, Vuletić, Lukin (2022). "A quantum processor based on coherent transport of entangled atom arrays." *Nature* 604, 451.**
[Nature](https://www.nature.com/articles/s41586-022-04592-6)
Showed that entangled atoms can be **physically shuttled** across the array between gate layers, giving dynamic, non-local connectivity. *Enabled:* flexible qubit connectivity without wiring — a key reason neutral atoms are well-suited to efficient error-correcting codes and compilation.

**Evered, Bluvstein, Kalinowski, Ebadi, Manovitz, Zhou, Li, Geim, Wang, Maskara, Levine, Semeghini, Greiner, Vuletić, Lukin (2023). "High-fidelity parallel entangling gates on a neutral-atom quantum computer." *Nature* 622, 268.**
[Nature](https://www.nature.com/articles/s41586-023-06481-y)
Two-qubit CZ gates at **99.5% fidelity**, run on many atom pairs in parallel — above the surface-code error-correction threshold. *Enabled:* gate quality good enough that quantum error correction can actually reduce errors rather than add them.

**Bluvstein, Evered, Geim, Li, Zhou, Manovitz, et al. (2024). "Logical quantum processor based on reconfigurable atom arrays." *Nature* 626, 58.**
[Nature](https://www.nature.com/articles/s41586-023-06927-3) · [arXiv:2312.03982](https://arxiv.org/pdf/2312.03982)
The landmark result: up to **48 error-corrected logical qubits** from ~280 physical atoms, running algorithms with hundreds of logical operations, and demonstrating that **increasing code distance lowers the logical error rate**. *Enabled:* the field's transition from "entangling physical qubits" to "running error-corrected logical algorithms" — the clearest evidence yet that neutral atoms are a viable path to fault tolerance.

**Rodriguez et al. / QuEra–Harvard–MIT (2025). "Experimental demonstration of logical magic state distillation." *Nature* (2025).**
[Nature](https://www.nature.com/articles/s41586-025-09367-3)
First magic-state distillation performed entirely on logical qubits (distance-3 and -5 color codes, a 5-to-1 protocol whose output beat every input). *Enabled:* a required ingredient for *universal* fault tolerance — not just protecting qubits, but producing the non-Clifford resources a universal logical computer needs.

---

## 7. Pasqal-focused publications & software (project emphasis)

**Henriet, Beguin, Signoles, Lahaye, Browaeys, Reymond, Jurczak (2020). "Quantum computing with neutral atoms." *Quantum* 4, 327.**
[Quantum journal](https://quantum-journal.org/papers/q-2020-09-21-327/) · [arXiv:2006.12326](https://arxiv.org/abs/2006.12326)
Pasqal's foundational review/white paper: a full tour from atoms and qubits up to application interfaces, with a taxonomy of NISQ-era tasks the platform addresses efficiently. *Best Pasqal-specific starting point.* *Enabled (as a reference):* a coherent statement of the neutral-atom value proposition that frames the company's roadmap.

**Seitz, Heim, Moutinho, Guichard, Abramavicius, Wennersteen, Both, Quelle, de Groot, Velikova, Elfving, Dagrada (2024). "Qadence: a differentiable interface for digital-analog programs." *arXiv:2401.09915*; published in *IEEE Software* (2025).**
[arXiv](https://arxiv.org/abs/2401.09915) · [Pasqal](https://www.pasqal.com/newsroom/qadence-a-differentiable-interface-for-digital-analog-programs/)
The reference for **Qadence**, Pasqal's open-source library for digital-analog programs. Its defining feature for QML is **native PyTorch autodifferentiation**, so quantum models behave like trainable PyTorch layers, with parameter-shift rules (including higher-order) usable on real hardware. *Enabled:* a practical software bridge between neutral-atom hardware and standard machine-learning workflows — directly relevant to QML, circuit compilation, and digital-analog computing.

**PyQTorch (Pasqal, open source). PyTorch-based differentiable state-vector / density-matrix simulator.**
[GitHub](https://github.com/pasqal-io/pyqtorch)
The default Qadence backend. *Enabled:* gradient-based training and simulation of digital-analog circuits inside the PyTorch ecosystem (a JAX-based alternative, Horqrux, also exists).

---

## 8. Best review articles to start reading

If you want to go deep, start with these four — together they cover theory, hardware, the tweezer platform, and the Pasqal view:

**Saffman, Walker, Mølmer (2010). "Quantum information with Rydberg atoms." *Reviews of Modern Physics* 82, 2313.**
[APS](https://link.aps.org/doi/10.1103/RevModPhys.82.2313) · [arXiv:0909.4777](https://arxiv.org/abs/0909.4777)
The canonical, heavily cited review of Rydberg-based quantum information — gates, collective encoding, light–atom interfaces. The single most authoritative theoretical reference for the field.

**Browaeys & Lahaye (2020). "Many-body physics with individually controlled Rydberg atoms." *Nature Physics* 16, 132.**
[Nature Physics](https://www.nature.com/articles/s41567-019-0733-z)
The leading review of Rydberg-atom many-body physics and simulation, from the Institut d'Optique group behind much of the hardware.

**Kaufman & Ni (2021).** *(see Section 2)* — the tweezer-platform review.

**Henriet et al. (2020).** *(see Section 7)* — the Pasqal/neutral-atom-computing review.

---

## Quick reference index

| # | Paper | Year | Venue | What it enabled |
|---|-------|------|-------|-----------------|
| 1 | Jaksch et al., *Fast quantum gates* | 2000 | PRL | Rydberg-based two-qubit gate (theory) |
| 2 | Lukin et al., *Dipole blockade* | 2001 | PRL | The blockade control primitive |
| 3 | Ashkin, *Radiation pressure* | 1970 | PRL | Light can trap matter |
| 4 | Ashkin et al., *Gradient force trap* | 1986 | Opt. Lett. | The optical tweezer (Nobel 2018) |
| 5 | Schlosser et al., *Sub-Poissonian loading* | 2001 | Nature | One atom per trap (Pasqal lineage) |
| 6 | Urban / Gaëtan et al., *Blockade observed* | 2009 | Nat. Phys. | Blockade confirmed experimentally |
| 7 | Isenhower et al., *Neutral atom CNOT* | 2010 | PRL | First neutral-atom CNOT |
| 8 | Endres & Barredo et al., *Defect-free arrays* | 2016 | Science | Scalable ordered qubit arrays |
| 9 | Bernien et al., *51-atom simulator* | 2017 | Nature | Frontier analog simulation |
| 10 | Ebadi et al., *256-atom simulator* | 2021 | Nature | 256-qubit programmable simulation |
| 11 | Scholl et al., *196-atom antiferromagnet* | 2021 | Nature | Hundreds-scale sim (Pasqal lineage) |
| 12 | Bluvstein et al., *Coherent transport* | 2022 | Nature | Reconfigurable connectivity |
| 13 | Evered et al., *99.5% CZ gates* | 2023 | Nature | Above-threshold gate fidelity |
| 14 | Bluvstein et al., *Logical processor* | 2024 | Nature | 48 error-corrected logical qubits |
| 15 | QuEra/Harvard/MIT, *Magic state distillation* | 2025 | Nature | Ingredient for universal fault tolerance |
| 16 | Henriet et al., *QC with neutral atoms* | 2020 | Quantum | Pasqal field review |
| 17 | Seitz et al., *Qadence* | 2024 | arXiv/IEEE Sw. | PyTorch-differentiable DAQC software |
| R1 | Saffman, Walker, Mølmer | 2010 | RMP | Canonical theory review |
| R2 | Browaeys & Lahaye | 2020 | Nat. Phys. | Many-body Rydberg review |
| R3 | Kaufman & Ni | 2021 | Nat. Phys. | Tweezer-array review |
