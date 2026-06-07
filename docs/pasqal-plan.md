# Pasqal Interview Prep — Mid-Senior Quantum Developer (Quantum Circuit Team)

## *Prep sheet for Dr. Boris Milanovic. Stage 1: HR / recruiter screen. Full technical depth below for rounds 2+.*

---

## 1. The role, in one paragraph

The Quantum Circuit team is an **R&D team on the digital side** of Pasqal's roadmap. They match emerging *digital* neutral-atom hardware capabilities to quantum protocols, working in the **early fault-tolerant regime** (post-NISQ, before full FTQC). The core technical pillars are: **quantum error correction**, **circuit compilation** (encoding/compiling across the stack, from hardware-native ops up to applications), and **quantum machine learning** — plus publications, IP/patents, and customer/academic collaboration. This is *not* the analog-optimization (MIS/annealing) side of Pasqal.

---

## 2. Your fit map

**Lean hard on these — they're real, not checkbox:**

- Physics PhD (they require CS/Physics/Math PhD).
- Genuine publication + research-output record (IEEE, Elsevier, NIM-A). They explicitly value "publication or code repository records" and an IP/patent track.
- Production-grade Python and software discipline most pure-academics lack.
- TensorFlow/Keras + physics PhD → a credible angle on **physics-informed ML**, which they list under appreciated QML topics.
- Variational-algorithm experience (QAOA) → bridges to variational QML.

**Close these gaps before round 2 (and be honest about them):**

- **QEC** — nothing on your CV signals it. Highest-priority study area.
- **Fault-tolerant algorithms / resource estimation** — they call this out specifically.
- **qadence** — their own SDK, listed first in the JD.
- **Quantum (not just classical) ML** — variational circuits, quantum kernels, data re-uploading.

**Narrative to carry:** *"Physics PhD with a real publication and shipping-code record, already hands-on with your stack, actively closing the QEC/FTQC gap."* Don't pretend the gap isn't there — show momentum on it instead.

---

## 3. Technical study plan

### Track A — qadence (PRIMARY)

Pasqal's open-source **digital-analog** quantum programming library, built on **PyTorch**. This is the closest tool to what the team does day-to-day, and the PyTorch base ties straight to your TensorFlow/Keras experience.

- Install locally (`pip install qadence`), run the getting-started notebooks.
- Build a small **variational / QML** model: parameterized circuit, feature map, observable, train it with the PyTorch optimizer loop.
- Understand its three modes: **digital**, **analog**, **digital-analog (DAQC)** — and how it can compile down to a Pulser backend for analog execution.
- Goal: be able to say "I built and trained a QML model in qadence, here's what I noticed about the digital-analog interface."

### Track B — Neutral-atom platform via Pulser + your Azure simulator

This is what your Azure Quantum Pasqal simulator actually runs (EMU_SV / EMU_MPS / EMU_FREE backends; submit format `pasqal.pulser.v1`).

- Learn the building blocks: **register**, **layout** (use pre-calibrated layouts), **Rydberg blockade**, **analog pulse sequences**.
- Run one small analog sequence on EMU_FREE (≤12 qubits) end to end on Azure; try EMU_MPS to see tensor-network emulation.
- Goal: speak credibly about *why neutral-atom hardware behaves as it does* — reconfigurable geometry, blockade interactions, analog vs digital gate execution.

### Track C — QEC & fault-tolerance fundamentals (the gap)

- **Stabilizer formalism** → **surface code** basics → **logical vs physical qubits**.
- **Early fault-tolerant regime**: modest logical-qubit counts, partial error correction, algorithms tolerant of limited overhead.
- **Magic-state distillation** and **resource estimation** (physical qubits/gates needed once a protocol is encoded — they name this explicitly).
- **Neutral-atom QEC advantage**: atom shuttling enables **transversal gates** and reconfigurable connectivity. Read about the Harvard/QuEra logical-qubit work (Bluvstein et al., *Nature* 2023) — the canonical neutral-atom QEC reference.
- Practice in Qiskit/Cirq (you already know them): build a small stabilizer circuit, simulate a toy code.

### Track D — QML talking point

Prepare one crisp, specific story at the **interface of scientific modeling and ML**:

- Classical side you know: physics-informed neural networks (PINNs), neural operators (Fourier Neural Operator, DeepONet).
- Quantum side: variational quantum circuits, quantum kernels, data re-uploading, differentiable quantum programs (qadence).
- Connect it to your physics PhD: "I've worked with both physical modeling and ML, and the physics-informed framing is where I think near-term quantum ML is most promising because..."

### Suggested one-week rhythm

- **Days 1–2:** qadence install + QML toy model (Track A).
- **Day 3:** Pulser + one Azure emulator run (Track B).
- **Days 4–5:** QEC fundamentals + neutral-atom QEC reading (Track C).
- **Day 6:** QML talking point + skim 1–2 recent Pasqal/neutral-atom arXiv papers ("technological watch").
- **Day 7:** consolidate; write your own 1-paragraph summary of each track in your words.

---

## 4. Concrete hands-on tasks (proof you used their stack)

1. **qadence:** train a small variational classifier or regressor; note the digital-analog compilation step.
2. **Pulser on Azure:** submit one analog sequence to EMU_FREE; inspect results format.
3. **Qiskit/Cirq:** simulate a distance-3 surface-code stabilizer measurement (toy version) and a simple resource-estimate calculation.
4. Push these to a public GitHub repo (they value code-repo evidence) — even small, clean examples count.

---

## 5. HR screen prep (this is the round that's actually next)

A recruiter screen is about **motivation, communication, and logistics** — not deep tech. Prepare these.

### Likely questions & how to frame answers

- **"Tell me about yourself."** → 60-second story (below). Tight, role-aimed, no full career replay.
- **"Why Pasqal?"** → Neutral-atom leadership, second-quantum-revolution scale-up, the specific pull of the digital/FTQC/QML problem space, and that you've already been hands-on with their stack. Specificity beats flattery.
- **"Why leave Microsoft / why now?"** → Move *toward* deep-tech quantum R&D and hands-on circuit/QEC work, not *away* from anything. Keep it positive and forward-looking.
- **"Walk me through your quantum experience."** → QAOA project (AI Tour Munich), D-Wave vs OR-Tools study, Azure Quantum, variational algorithms, physics PhD foundation. Honest about being NISQ/applied so far and deliberately moving into FTQC/QEC.
- **"What do you know about us?"** → Founded 2019, France (Palaiseau/Massy HQ), neutral-atom leader, ~275–300 people, Nobel laureate co-founder (Alain Aspect), going public via SPAC (Bleichroeder) in H2 2026, partners include IBM, NVIDIA, customers like CMA CGM, Thales, Sumitomo. (Don't overdo the finance talk — show you've done homework, move on.)
- **"Salary expectations?"** → Have a researched range for a France-based mid-senior quantum role ready; it's fine to say you're flexible and want to understand the full package first.
- **"Notice period / availability?"** → Know your Microsoft notice period and earliest start.

### Logistics to have a clear answer on (they WILL come up)

- **Relocation / work arrangement.** Role is **Massy/Palaiseau, France, hybrid** — you're in Dublin. Decide your position (relocate? hybrid cadence? timeline?) *before* the call. This is often a screen-stage filter.
- **Work authorization** — EU citizen, so straightforward; mention it, it's a plus.
- **Language** — your German C2 / French B2 is a nice signal for a French team; worth surfacing.

### Your 60-second story (adapt in your own voice)

> "I'm a physicist-turned-engineer — PhD in quantum/nuclear physics with a detector-readout and HPC background, and a publication record from that work. Since then I've spent ~20 years building production software and cloud systems, currently at Microsoft on Azure and Azure Quantum. Over the last while I've focused on quantum specifically — QAOA work I demoed at AI Tour Munich, a D-Wave vs classical optimization study, and hands-on time with quantum frameworks. What draws me to this role is the digital, fault-tolerant side of neutral-atom computing — I've started working in your stack directly, and I'm deliberately deepening on QEC and quantum ML, which is exactly where this team sits."
> 

### Questions to ask them (shows seriousness)

- How does the Quantum Circuit team split time across QEC, compilation, and QML right now?
- What does "early fault-tolerant" mean concretely on Pasqal's current/next hardware generation?
- How much of the role is publication/IP vs. internal stack development?
- What does the hybrid arrangement and team distribution look like in practice?

---

## 6. Quick fact pack on Pasqal (current as of mid-2026)

- French neutral-atom quantum computing company, founded 2019, HQ Palaiseau/Massy, ~275–300 employees, subsidiaries in several countries (incl. new US HQ in Illinois).
- Co-founders include Nobel laureate **Alain Aspect** and neutral-atom pioneer **Antoine Browaeys**.
- Technology: programmable arrays of neutral atoms via optical tweezers; **Rydberg blockade** for interactions; both **analog** and **digital** gate modes.
- Software: open-source **Pulser** (pulse-level) and **qadence** (digital-analog, PyTorch-based); cloud access via own platform plus Azure, Google Cloud, OVHcloud, Scaleway.
- Business: ~7 deployed systems, 25+ commercial use cases, 40+ clients/partners (IBM Quantum Network member, NVIDIA, CMA CGM, Thales, Sumitomo). Going public via SPAC merger with Bleichroeder Acquisition Corp. II (announced March 2026; F-4 filed May 2026; expected to close H2 2026).