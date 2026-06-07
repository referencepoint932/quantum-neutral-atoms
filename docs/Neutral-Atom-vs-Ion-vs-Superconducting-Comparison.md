# Quantum Computing Modalities Compared: Neutral Atom (incl. Pasqal), Trapped Ion & Superconducting

*Prepared: June 2026. Quantum hardware moves fast; the figures below are best-available public benchmarks as of mid-2026 and should be treated as snapshots, not fixed specs. Vendor "qubit counts" mix physical and "algorithmic/logical" definitions, so cross-vendor numbers are indicative rather than apples-to-apples.*

---

## 1. Executive Summary

Three platforms lead the race toward useful quantum computing, each occupying a different point on the same set of trade-offs:

- **Neutral atom** (Pasqal, QuEra, Atom Computing) wins on **scalability and connectivity flexibility**, operates at **room temperature**, and natively supports **analog** quantum simulation. It pays for this with **slower gates** and a **less mature digital/gate-model track record** than the other two.
- **Trapped ion** (Quantinuum, IonQ) wins on **raw fidelity and coherence** — the cleanest qubits in the business with **all-to-all connectivity** — but has the **slowest gates** and the **lowest qubit counts**.
- **Superconducting** (IBM, Google) wins on **gate speed** and has the most mature **engineering/fabrication** ecosystem, but is constrained by **cryogenics**, **limited on-chip connectivity**, and **shorter coherence**.

No modality is "best" overall. Neutral atom is the strongest scaling story; trapped ion is the strongest quality story; superconducting is the strongest speed-and-maturity story.

---

## 2. Neutral Atom Quantum Computing (General)

### 2.1 How it works
Individual neutral atoms (commonly rubidium or cesium) are held in optical tweezers — focused laser beams — arranged into 1D, 2D, or 3D arrays. Qubits are encoded in atomic energy levels. Two-qubit entangling gates exploit the **Rydberg blockade**: atoms briefly excited to high-energy Rydberg states interact strongly via van der Waals forces, preventing simultaneous excitation within a "blockade radius" and producing controlled interactions. Atoms can be physically **moved** mid-computation, giving reconfigurable connectivity.

### 2.2 Benefits

| Benefit | Why it matters |
|---|---|
| **Scalability** | Identical atoms, no fabrication variability. Roadmaps target 100,000 atoms in a single vacuum chamber; Caltech demonstrated ~6,100 trapped atomic qubits (Sept 2025) and continuous 3,000-atom operation has been shown. |
| **Reconfigurable connectivity** | Any two atoms can be shuttled adjacent to each other — far more flexible than fixed superconducting chip layouts; approaches effective all-to-all. |
| **Room-temperature operation** | No dilution refrigerator. Systems run at ambient temperature with modest power (Pasqal's rack-mounted units ~3 kW), compatible with standard data-center infrastructure. |
| **Native analog mode** | Excellent for quantum simulation of many-body physics and combinatorial optimization *today*, before full gate-based fault tolerance arrives. |
| **Identical qubits** | Atoms are perfectly uniform by nature, eliminating the qubit-to-qubit variation that plagues fabricated devices. |
| **Strong error-correction path** | High parallelism and atom movement suit large surface codes; 2025 saw below-threshold error correction and dozens of logical qubits demonstrated on neutral-atom platforms. |

### 2.3 Drawbacks

| Drawback | Why it matters |
|---|---|
| **Slower gates** | Rydberg two-qubit gates run at ~1 µs — ~100–1,000× slower than superconducting (ns-scale). Lower circuit throughput. |
| **Atom loss & reloading** | Atoms can escape traps; arrays must be re-prepared, adding overhead and limiting continuous run time (being addressed by continuous-reload designs). |
| **Digital/gate maturity** | The gate-model (digital) track record is younger than superconducting/ion; much commercial value to date has come from analog mode. |
| **Mid-circuit measurement** | Fast, selective mid-circuit readout and feed-forward are harder than in some rivals (improving rapidly). |
| **Control complexity at scale** | Driving thousands of independent tweezers/laser channels with high fidelity is a hard optical-engineering problem. |
| **Heating/decoherence from lasers** | Laser-based control introduces its own noise channels that must be managed. |

---

## 3. Pasqal (Neutral Atom, Specific)

Pasqal is a French neutral-atom company (spun out of Institut d'Optique, co-founded with Alain Aspect's lineage). It emphasizes an **upgradable, room-temperature, analog-to-digital** platform and a strong open-source software stack.

### 3.1 Hardware & roadmap

| Generation / system | Approx. qubits | Timing | Note |
|---|---|---|---|
| Fresnel 2 | ~100 physical | Available | Analog/digital-analog processor |
| Orion Beta / Gamma | >140 physical | Late 2025 | Third-gen "Orion" line, rack-mountable, room temperature (~3 kW) |
| ~1,000 physical qubits | ~1,000 | End 2025 target | Scaling milestone |
| 250-qubit advantage demo | 250 | Early 2026 target | Industry-problem quantum-advantage demonstration |
| Vela | 256+ physical | ~2026–2027 | Higher repetition rate & fidelity |
| Centaurus | — | 2028 | Architected for early fault tolerance |
| Lyra | — | 2029 | Targeting impactful FTQC |

A **506-atom defect-free register** was demonstrated in 2024, underscoring array-preparation capability.

**Logical-qubit roadmap:** 2 logical qubits (2025) → 20 (2027) → ~100 high-fidelity (2029) → 200 (2030).

### 3.2 Software stack (key project focus areas)

Pasqal's open-source stack is one of its differentiators, especially the **PyTorch integration** for quantum machine learning and variational algorithms:

- **Pulser** — pulse-level interface for programming neutral-atom devices directly, with built-in qubit-interaction modeling. (`PulserDiff` adds pulse-level differentiability.)
- **Qadence** — a higher-level **digital-analog** quantum programming library. Block-based composition (inspired by Yao.jl), built for variational circuits and QML.
- **PyQTorch** — a PyTorch-based statevector emulator (digital + digital-analog) for ~25 qubits, with built-in **automatic differentiation**.
- **PyTorch / JAX integration** — out-of-the-box autodiff of quantum programs; supports **automatic differentiation (AD)** in simulators and **parameter-shift rules (PSR)** (incl. higher-order generalized PSR) for hardware-compatible gradients.

This makes Pasqal's stack particularly well-suited to **QML**, variational optimization, and **circuit compilation** research that blends classical deep-learning tooling with quantum hardware.

### 3.3 QEC approach
Pasqal leverages neutral-atom strengths (parallelism, atom movement, large arrays) for surface-code-style error correction, with an **upgradable platform** thesis: the *same* hardware running analog workloads today is engineered to evolve into digital, error-corrected operation, compressing the timeline to fault tolerance.

### 3.4 Pasqal-specific pros & cons

| Strengths | Limitations |
|---|---|
| Room-temperature, rack-mountable, data-center-friendly (~3 kW) | Digital gate fidelities still maturing vs. ion leaders |
| Best-in-class **open-source/PyTorch software** (Qadence, Pulser, PyQTorch) | Logical-qubit count today is small (~2) |
| Strong **analog** quantum simulation & optimization today | Slower gates inherent to Rydberg approach |
| Clear modular, **upgradable** path to FTQC | Roadmap milestones (1,000 qubits, 250-qubit advantage) still being proven at scale |
| European sovereignty / on-prem deployments | Smaller revenue/installed base than IBM-class incumbents |

---

## 4. Cross-Modality Benchmark Comparison

### 4.1 Headline technical benchmarks (flagship public systems, mid-2026)

| Metric | Neutral Atom (Pasqal / QuEra / Atom Computing) | Trapped Ion (Quantinuum / IonQ) | Superconducting (IBM / Google) |
|---|---|---|---|
| **Physical qubits (flagship)** | 100s today; 1,000s demonstrated (≈6,100 atoms trapped at Caltech) | ~56–98 (Quantinuum Helios 98; H2 56) | ~105–156+ (Google Willow 105; IBM Heron R2 156) |
| **Best 2-qubit gate fidelity** | ~99.5% (Atom Computing ~99.6%) | **~99.9%+** (Helios 99.92%; IonQ/Oxford Ionics >99.99%) | ~99.5–99.9% (IBM Heron R2) |
| **Coherence time** | Long (seconds-scale achievable) | **Longest** (seconds to minutes) | Shortest (~100–300 µs) |
| **2-qubit gate speed** | ~1 µs (medium) | ~10–100 µs (slowest) | **~20–100 ns (fastest)** |
| **Connectivity** | Reconfigurable, near all-to-all (atoms move) | **All-to-all** | Limited (nearest-neighbor on chip) |
| **Operating temperature** | Room temperature | Room temp (ions); cryo support optics | **~10–15 mK (dilution fridge)** |
| **Native analog mode** | **Yes** | Limited | No |
| **Error-correction milestone** | Below-threshold QEC; dozens of logical qubits (2025) | **48 logical qubits** on Helios (Nov 2025); DARPA-funded 100 logical by 2027 | Google below-threshold (Dec 2024); IBM qLDPC roadmap |

*Bold = category leader for that row.*

### 4.2 Qualitative scorecard (0–10, higher is better)

Scores are an opinionated synthesis of the benchmarks above — useful for relative comparison, not precise measurement.

| Criterion | Neutral Atom | Trapped Ion | Superconducting |
|---|---:|---:|---:|
| Scalability (path to 1M qubits) | **9** | 5 | 7 |
| 2-qubit gate fidelity | 7 | **10** | 7 |
| Coherence time | 8 | **10** | 4 |
| Gate speed / clock rate | 5 | 3 | **10** |
| Qubit connectivity | 9 | **10** | 4 |
| Operational simplicity (no cryo) | **9** | 7 | 3 |
| Ecosystem / software maturity | 7 | 7 | **9** |
| Manufacturing maturity | 6 | 6 | **9** |
| Error-correction readiness | 8 | 8 | **8** |
| Analog/simulation capability | **10** | 4 | 3 |
| **Unweighted total (/100)** | **78** | **70** | **64** |

### 4.3 Weighted score (illustrative "near-term utility" weighting)

Weights reflect what tends to matter most for reaching practical, error-corrected machines. Adjust to taste — the ranking is sensitive to weighting.

| Criterion | Weight | Neutral Atom | Trapped Ion | Superconducting |
|---|---:|---:|---:|---:|
| Scalability | 20% | 9 | 5 | 7 |
| Gate fidelity | 20% | 7 | 10 | 7 |
| Coherence | 10% | 8 | 10 | 4 |
| Gate speed | 10% | 5 | 3 | 10 |
| Connectivity | 10% | 9 | 10 | 4 |
| Operational simplicity | 10% | 9 | 7 | 3 |
| Software/ecosystem | 10% | 7 | 7 | 9 |
| Manufacturing maturity | 10% | 6 | 6 | 9 |
| **Weighted total (/10)** | 100% | **7.6** | **7.3** | **6.7** |

*Read with care: under a "fidelity-first" weighting, trapped ion leads; under a "speed/maturity-first" weighting, superconducting closes the gap. The platforms are genuinely close.*

---

## 5. Pros & Cons at a Glance

### Neutral Atom
**Pros:** best scalability, reconfigurable/near all-to-all connectivity, room-temperature operation, native analog simulation, identical qubits.
**Cons:** slower gates than superconducting, atom loss/reloading overhead, younger digital-gate track record, optical control complexity at scale.

### Trapped Ion
**Pros:** highest gate fidelities (>99.9%), longest coherence (seconds–minutes), full all-to-all connectivity, mature high-quality operations.
**Cons:** slowest gates (10–100 µs), lowest qubit counts, scaling beyond ~100 qubits requires complex ion-shuttling/photonic interconnects.

### Superconducting
**Pros:** fastest gates (ns), most mature fabrication/ecosystem, largest installed base (IBM cloud), strong QEC demonstrations (Google Willow, IBM qLDPC).
**Cons:** requires millikelvin cryogenics, limited on-chip connectivity, shorter coherence, fabrication variability between qubits.

---

## 6. Bottom Line

If the deciding factor is **scaling to large, connectivity-rich, room-temperature machines** — and especially if **analog simulation, QML, and a PyTorch-native software stack** matter — **neutral atom (Pasqal)** is the standout. If the deciding factor is **raw qubit quality today**, **trapped ion** leads. If it is **speed and engineering maturity**, **superconducting** leads. The most likely near-term outcome is **coexistence**, with workloads routed to the modality whose trade-offs fit best, while all three converge on fault tolerance over 2027–2030.

---

## 7. Caveats & Methodology
- Benchmark numbers are public, vendor-reported or peer-reviewed figures as of mid-2026 and change frequently. "Best" fidelities are often demonstrated under optimal conditions on selected qubit pairs, not guaranteed system-wide.
- Qubit counts are not directly comparable across vendors (physical vs. algorithmic vs. logical definitions differ).
- Scores in Sections 4.2–4.3 are an editorial synthesis for relative comparison, not measured quantities. The weighted ranking is sensitive to the chosen weights.

---

## 8. Sources
- [Key Advantages of Neutral-Atom Architectures — QuEra](https://www.quera.com/blog-posts/key-advantages-of-neutral-atom-quantum-computer-architectures)
- [Neutral Atom Quantum Computing: 2026's Big Leap — IEEE Spectrum](https://spectrum.ieee.org/neutral-atom-quantum-computing)
- [Neutral Atom Quantum Computing: The Room-Temperature QC — PostQuantum](https://postquantum.com/quantum-modalities/neutral-atom-quantum/)
- [Neutral-Atom Quantum Computing Market 2026–2036 — Businesswire/ResearchAndMarkets](https://www.businesswire.com/news/home/20260112950379/en/)
- [Pasqal Announces New Roadmap (beyond 1,000 qubits) — Pasqal](https://www.pasqal.com/newsroom/pasqal-announces-new-roadmap-focused-on-business-utility-and-scaling-beyond-1000-qubits-towards-fault-tolerance-era/)
- [Pasqal 2025 Roadmap — The Quantum Insider](https://thequantuminsider.com/2025/06/12/pasqal-releases-2025-roadmap-showcasing-upgradable-platform-from-todays-quantum-solutions-to-tomorrows-fault-tolerant-systems/)
- [Pasqal Roadmap Signals Strategic Acceleration — Quantum Computing Report](https://quantumcomputingreport.com/pasqals-2025-roadmap-signals-strategic-acceleration-toward-fault-tolerant-quantum-computing/)
- [Pasqal Error Correction](https://www.pasqal.com/error-correction/)
- [Qadence — Pasqal blog](https://www.pasqal.com/blog/qadence-a-user-friendly-library-for-designing-digital-analog-quantum-programs/)
- [Qadence GitHub](https://github.com/pasqal-io/qadence)
- [Qadence: a differentiable interface for digital-analog programs — arXiv](https://arxiv.org/pdf/2401.09915)
- [PulserDiff: a pulse differentiable extension for Pulser — arXiv](https://arxiv.org/pdf/2505.16744)
- [High-fidelity parallel entangling gates on a neutral-atom quantum computer — arXiv](https://arxiv.org/abs/2304.05420)
- [Quantinuum Helios / 56-qubit record — Quantinuum](https://www.quantinuum.com/press-releases/quantinuum-launches-industry-first-trapped-ion-56-qubit-quantum-computer-that-challenges-the-worlds-best-supercomputers)
- [Oxford 10⁻⁷ qubit gate error — PostQuantum](https://postquantum.com/quantum-research/oxford-qubit-gate-error/)
- [Quantum Hardware 2026: IBM vs Google vs IonQ vs Quantinuum — Quantum AI Report](https://quantumaireport.info/articles/quantum-hardware-comparison-2026)
- [The Race Toward FTQC: Willow, Heron, Ocelot, Majorana — PostQuantum](https://postquantum.com/quantum-computing/fault-tolerant-quantum-race/)
- [Google Willow chip — BlueQubit](https://www.bluequbit.io/blog/googles-quantum-computing-chip-willow)
- [Choosing a Quantum Stack in 2026 — upQubit](https://upqubit.com/choosing-a-quantum-stack-in-2026-trapped-ion-superconducting)
- [Comparing Neutral-Atom QC With Other Modalities — The Quantum Insider](https://thequantuminsider.com/2024/02/22/harnessing-the-power-of-neutrality-comparing-neutral-atom-quantum-computing-with-other-modalities/)
