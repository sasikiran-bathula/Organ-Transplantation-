 * Backend:
```bash
cd QuantaOrgan\backend
python -m venv venv
venv\Scripts\activate
pip install -r requirements.txt
uvicorn main:app --reload --port 8000
```

* Frontend:
```bash
cd QuantaOrgan\frontend
npm install
npm run dev
```

Access at http://localhost:3000

Also access through live that I have deployed globally:
https://quantaorgan.web.app


### Key Highlights from the Specification

1. **Layered System Architecture:**
   * Detailed **Mermaid flowcharts** and **sequence diagrams** mapping interactions
   * **Next.js 15 / React 19** frontend
   * **FastAPI / SQLAlchemy REST gateway**
   * **Persistence tiers**

2. **Quantum vs. Classical Engine:**
   * **QUBO Hamiltonian mappings**
   * **p-layer QAOA** parameterized ansatz circuits on **IBM Qiskit Aer**
   * Baseline comparisons against **SciPy Hungarian/MILP solvers**

3. **Deterministic Clinical Rule Engine:**
   * Algorithmic evaluation of **ABO compatibility**
   * **Virtual CDC**
   * **DSA screening**
   * Organ-specific **cold ischemia decay limits**

4. **Security & Compliance Architecture:**
   * **6-role RBAC matrix**
   * **RFC 7519 JWT** session security
   * **SHA-256 chained** tamper-evident audit trails

5. **Deployment Topologies:**
   * Cloud **CDN architecture** on **Firebase Hosting** (`quantaorgan.web.app`)
   * Local standalone execution from your `OT_QUANTA.zip` package




    # 🏥 QuantaOrgan Platform
> Real-time donor coordination, deterministic crossmatch, and quantum allocation research.

## 🖥️ Platform Preview

<img width="1901" height="906" alt="Screenshot 2026-10-12 164413" src="https://github.com/user-attachments/assets/fe25d56b-03a6-484f-80f0-4e51211614b8" />




# ⚛️ QuantumMatch: Optimizing Organ Allocation with Quantum Computing

> A hybrid quantum-classical decision-support prototype that turns organ transplant matching into a QUBO problem and solves it with QAOA (Qiskit).

---

## 📌 Table of Contents
1. [Introduction](#-introduction)
2. [The Problem](#1-the-problem)
3. [Why Classical Computing Struggles](#2-why-classical-computing-struggles)
4. [Quantum Computing in 90 Seconds](#3-quantum-computing-in-90-seconds)
5. [Our Approach](#4-our-approach-from-transplant-matching-to-qubits)
6. [System Architecture](#5-system-architecture)
7. [How the Quantum Model Runs](#6-how-the-quantum-model-runs)
8. [Benchmark Results](#7-benchmark-results)
9. [Classical vs. Quantum](#8-classical-vs-quantum)
10. [Impact, Challenges and Future Scope](#9-impact-challenges-and-future-scope)
11. [Repository Structure and How to Run](#10-repository-structure-and-how-to-run)
12. [Key Terms](#11-key-terms)
13. [Discussion Questions](#12-discussion-questions)

---

## 🎤 Introduction

Imagine a donor heart becomes available. In about **4–6 hours** it must be matched, transported, and transplanted, while hundreds of patients wait. Every year, many patients die waiting for an organ. Part of the problem is donor shortage, and part is how we **match and allocate** the organs we have.

**QuantumMatch** reframes allocation as a combinatorial optimization problem and explores how quantum optimization can support it.

---

## 1. The Problem

Matching depends on **many factors at the same time**:

* **Blood group compatibility**
* **HLA tissue match**
* **Organ size and weight match**
* **Medical urgency and waiting time**
* **Distance and ischemic time** (heart ≈ 4–6 h, liver ≈ 8–12 h, kidney ≈ 24–36 h)
* **Paired kidney exchange:** Incompatible pairs swap in cycles (A → B → C → A)

The question is not *"is this one match good?"* but **"what is the best set of matches across all patients and donors at once?"**

---

## 2. Why Classical Computing Struggles

* **N donors and N recipients give N! possible assignments** (20 patients ≈ 2.4 × 10¹⁸).
* **Kidney exchange with cycles of length 3 or more is NP-hard.**
* **Classical solvers (ILP, heuristics)** are excellent at small and medium scale, but rely on approximations or slow down as constraints and network size grow.
* **Allocation is dynamic:** new donors arrive and conditions change.

---

## 3. Quantum Computing in 90 Seconds

| Concept | Meaning | Why it helps |
| :--- | :--- | :--- |
| **Superposition** | n qubits represent \(2^n\) states | Many candidate matchings represented together |
| **Entanglement** | Qubits are correlated | Encodes dependencies between choices |
| **Interference** | Amplitudes add or cancel | Good solutions amplified, bad ones suppressed |

---

## 4. Our Approach: From Transplant Matching to Qubits

### Step 1: Binary variables
`x_ij = 1` if donor \(i\) is matched to recipient \(j\). One variable is one qubit.

### Step 2: QUBO cost function
```text
Minimize:  - ∑ w_ij · x_ij
           + A · ∑_i (∑_j x_ij - 1)²      (each donor gives one organ)
           + A · ∑_j (∑_i x_ij - 1)²      (each recipient gets one organ)
```

* `w_ij` is the compatibility score, built from HLA match, urgency, waiting time, and size match.
* **Penalty terms** enforce the constraints.
* Blood-group-incompatible pairs receive a strong negative weight.

### Step 3: Solve with QAOA (Quantum Approximate Optimization Algorithm)
The best allocation becomes the **lowest-energy state** of a quantum system.

---

## 5. System Architecture

```text
Donor and recipient data (synthetic CSV)
              ↓
Preprocessing: filtering and compatibility scores
              ↓
QUBO formulation
              ↓
Quantum solver: QAOA (Qiskit)  ↔  Classical optimizer (COBYLA)
              ↓
Classical validity check
              ↓
Ranked suggestions → Clinician makes the final decision
```

* **Hybrid design:** classical code handles data and checks, the quantum part handles optimization.
* **Decision support only:** clinicians and ethics committees decide.

---

## 6. How the Quantum Model Runs

1. **Load and score:** read the CSV files and compute `w_ij`.
2. **Build QUBO:** one qubit per donor-recipient pair.
3. **Hadamard gates** put all qubits in superposition.
4. **Cost layer (\(\gamma\))** encodes scores and constraints as phases.
5. **Mixer layer (\(\beta\))** lets amplitudes interfere.
6. **Measure** over many shots, each giving one candidate matching.
7. **Classical optimizer** updates \(\gamma\) and \(\beta\), then repeats.
8. **Post-process:** pick the best valid bitstring and output the matching.

---

## 7. Benchmark Results

### Classical MILP / Hungarian vs. Qiskit QAOA / QUBO simulation:

| Metric | Result |
| :--- | :--- |
| Classical runtime (SciPy Hungarian / MILP) | **0.305 ms** |
| Quantum runtime (Qiskit QUBO, Aer statevector) | **24.192 ms** |
| Feasibility rate | **100%** (0 constraint violations) |
| Measured speedup | **0.013x** |
| Backend | `aer_simulator_statevector` (simulator, not real hardware) |

### What this tells us
* ✅ **The quantum formulation is correct:** every output was a valid matching.
* ⏱️ **Classical is faster at this small scale**, which is expected. The Hungarian algorithm is extremely efficient on small assignment problems, and a simulator pays an exponential cost to imitate qubits.
* 🔬 We do not claim quantum speedup. This benchmark is the validated foundation for testing larger instances and real hardware.

---

## 8. Classical vs. Quantum

| Aspect | Classical | Quantum (our approach) |
| :--- | :--- | :--- |
| **Information unit** | Bit | Qubit (superposition) |
| **Search** | One candidate at a time, or heuristics | Many candidates represented together, amplified by interference |
| **Output** | Exact, deterministic | Probabilistic distribution of answers |
| **Best today** | Small and medium problems | Potential for large, highly constrained problems |
| **Maturity** | Decades of tuned software | Early, noisy hardware (NISQ) |

> 💡 *Classical wins today at small scale. Quantum is the bet on tomorrow's larger, harder problems.*

---

## 9. Impact, Challenges and Future Scope

### Potential impact
* More organs matched, fewer wasted
* Better long-term graft survival through higher-quality matches
* Faster, fairer, more transparent allocation

### Honest challenges
* **NISQ hardware** has limited qubits and noise, so we use hybrid methods.
* Patient data is sensitive: compliance with **HIPAA (US)** and **DPDP Act 2023 (India)**, with anonymized or synthetic data.
* Clinical, ethical, and regulatory validation is required.
* Quantum advantage for this problem is **not yet proven**.

### Future scope
* Multi-organ and kidney-exchange cycle optimization
* Real-time organ transport and route optimization
* Testing on real IBM Quantum hardware
* Validation on anonymized registry data (OPTN, SRTR, NOTTO)

---

## 10. Repository Structure and How to Run

```text
QuantumMatch/
├── README.md
├── donors.csv          # 20 synthetic donors
├── recipients.csv      # 20 synthetic recipients
└── quantum_match.py    # scoring, QUBO, QAOA, classical benchmark
```

```bash
pip install pandas numpy qiskit qiskit-optimization qiskit-aer
python quantum_match.py
```

> ⚠️ *All data is randomly generated, not real patient data. Qiskit import names vary between versions, so check your installed version if an import fails.*

---

## 11. Key Terms

| Term | Full form | Meaning |
| :--- | :--- | :--- |
| **HLA** | Human Leukocyte Antigen | Tissue-matching markers |
| **QUBO** | Quadratic Unconstrained Binary Optimization | Problem format for quantum optimizers |
| **QAOA** | Quantum Approximate Optimization Algorithm | Hybrid quantum optimization algorithm |
| **NISQ** | Noisy Intermediate-Scale Quantum | Today's quantum hardware |
| **NP-hard** | Non-deterministic Polynomial-time hard | No known fast algorithm for every case |
| **ILP / MILP** | (Mixed) Integer Linear Programming | Classical optimization benchmark |
| **OPTN** | Organ Procurement and Transplantation Network | US organ allocation system |
| **NOTTO** | National Organ and Tissue Transplant Organisation | India's national body |
| **HIPAA** | Health Insurance Portability and Accountability Act | US health data privacy law |
| **DPDP** | Digital Personal Data Protection Act, 2023 | India's data privacy law |

---

## 12. Discussion Questions

1. **Which transplant decision most needs better data or optimization?**
   Paired kidney exchange and multi-organ allocation, where options grow explosively and time limits matter.
2. **What evidence would clinicians need before adopting a quantum-enabled tool?**
   Benchmarks against current solvers, retrospective validation on anonymized data, reproducibility and explainability, and prospective pilots in a decision-support role.
3. **How should privacy, fairness, and safety be evaluated?**
   * **Privacy** through anonymization, encryption, and legal compliance.
   * **Fairness** through outcome audits across age, sex, region, and socioeconomic groups.
   * **Safety** through human-in-the-loop review, classical validation of every output, and continuous monitoring.


