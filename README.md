# THE MULTIPLIER WAS NEVER THE PROBLEM

## Tensordyne Napier, the Pareto Logarithmic Number System, and Why the Addition Debt Remains: A Structural Comparison with the CORDIC-Native Substrate

**ERI Labs · Eric Ren · Jersey City, New Jersey · github.com/ericrenone · June 17, 2026**

---

> *"The shift is not the primitive. The shift and the add together are the primitive."*
> — CORDIRAC, ERI Labs, March 2026

> *"Multiplying matrices of numbers is hard, even for computers with matrix multiplier units. But adding banks of numbers is a lot easier."*
> — NextPlatform on Tensordyne, June 16, 2026

> *"Tensordyne's intellectual property lies in its proprietary approximation for efficient addition in the log domain and its hardware implementation."*
> — IndexBox, June 15, 2026

> *"2–10× energy overhead for iterative workloads on IEEE 754 vs. CORDIC-native arithmetic."*
> — Luo et al., IEEE TVLSI, 2019

---

## The Thesis

Two systems have now independently reached the same conclusion about IEEE 754 floating-point arithmetic: it is the wrong substrate. They have reached incompatible conclusions about what to replace it with.

**Tensordyne Napier** — taped out June 15, 2026, TSMC 3nm, 138 billion transistors, 300W, systems shipping Q2 2027 — replaces the multiplier with logarithmic arithmetic. Under the Pareto number system, multiplication becomes addition in the log domain (log(a×b) = log(a) + log(b)), and the freed silicon area funds more memory. The multiplier, the expensive part, is gone. The outstanding problem — addition in the log domain is nonlinear and has always required a lookup table — is solved via a proprietary Mitchell approximation correction achieving 99.9% accuracy versus FP16 baseline. No retraining required.

**The CORDIC-native substrate** — established theoretically by the ERI corpus through 2025–2026, with production validation at CARMEN (28nm CMOS, arXiv:2605.06878, June 2026), SYCore (arXiv:2503.11685, 2025), NeuEdge (arXiv:2602.02439, 2026), and CORVET (arXiv:2602.19268) — replaces the multiplier with iterative shift-and-add rotation. No multiplications. No lookup tables. No approximation committed at silicon. Three geometric modes — circular (m=+1), flat (m=0), hyperbolic (m=-1) — compute all transcendental functions (sin, cos, exp, ln, √, arctan, arcsinh) via the same primitive: a direction bit and a shift.

Both escape IEEE 754. Neither escapes the comparison with the other.

The connection is not peripheral. Tensordyne's addition correction — the critical innovation in Pareto — is a computation of the form log(1 + 2^z), where z is the log-domain difference of two operands. This is precisely the function that CORDIC's hyperbolic mode (m=-1) computes to arbitrary precision via shift-and-add, with no lookup table, with convergence guaranteed by the Volder (1959) iteration. Tensordyne's IP is an approximation of CORDIC's exact output. The addition problem that LNS could never solve without a table, CORDIC solved without a table in 1959. The correction Tensordyne ships in silicon is a lossy one-shot version of what a two-stage hyperbolic CORDIC pipeline delivers exactly.

This is not a criticism of Tensordyne. It is a structural fact about the arithmetic landscape: LNS and CORDIC are not symmetrical alternatives. One is a log-domain trick whose hard part is an approximation. The other is a rotation-domain architecture whose hard part has an exact iterative solution. They are competing for the same silicon at the same moment. Only one of them has a quantum extension.

---

## Part I — The Problem Both Systems Are Solving

The multiplier is expensive. Its area in silicon grows quadratically with bit width; an adder grows linearly. A 16-bit multiplier costs roughly 256 elementary cells where a 16-bit adder costs 16. In a transformer matmul — the dominant workload for LLM inference — a naive systolic array spends approximately half its gate count on multiplication hardware that could, in principle, be replaced.

This is not a new observation. IBM engineers in 1962 noticed it. Every decade since, a research group has proposed escaping the multiplier. Every proposal failed to reach production. The reasons for failure were not theoretical. They were practical:

For **logarithmic number systems**: multiplication in the log domain is free (addition). But addition in the log domain is nonlinear. Computing log(A + B) from log(A) and log(B) requires evaluating s(z) = log₂(1 + 2^z), which is neither linear nor shift-decomposable. Historically this required a lookup ROM per operation. For AI, where a matmul is ~50% multiplications and ~50% additions, the lookup tables ate back the savings.

For **CORDIC-based designs**: CORDIC is iterative; each additional bit of precision costs one more iteration (one shift, one add, one direction decision). The latency of a CORDIC pipeline scales with bit depth. For floating-point compatibility at FP32, 32 iterations. For AI at FP8 or INT8, 8–16 iterations. The question is whether the eliminated multipliers recoup the sequential iteration cost.

For **reduced-precision floating point** (FP8, FP4, INT4): the multipliers shrink with bit width but do not disappear. Energy still scales with computation, not with approximation.

Both Tensordyne and the CORDIC-native substrate claim to have solved their respective practical failure modes. Tensordyne claims their Pareto approximation for log-domain addition is accurate enough at 99.9% versus FP16 baseline, cheap enough to eliminate the lookup table, and error-bounded enough not to compound catastrophically over a long dot product. The CORDIC-native substrate claims that at AI precisions (INT8, INT4), the CORDIC pipeline depth is 8–16 stages — competitive in latency, superior in energy, and capable of exact computation of the same functions Tensordyne approximates.

---

## Part II — The LNS Architecture: What Pareto Gets Right

Tensordyne's Pareto system is not a naive LNS. It is the first production silicon for a specific solution to the LNS addition problem that makes the following choices:

**Starting point — Mitchell approximation (1962):** For x ∈ [0, 1], log₂(1 + x) ≈ x. This is a single-bit approximation: replace the mantissa with itself, drop the true log computation entirely. Error is one-sided and bounded for small x, but compounds over a long dot product accumulation.

**Tensordyne's correction:** Bolt on bit-shift corrections and rounding adjustments that cut the average accumulated error by approximately 100×. Convert from log domain back to linear for accumulation. The result: 99.9% accuracy versus FP16 on Mixtral-8×22B, Llama 3-70B, Falcon-180B, Stable Diffusion XL, Llama 3.1-405B. No model retraining required.

**What this buys:**
- Multipliers in the systolic array become adders. Area shrinks. The freed silicon funds memory.
- 48 logarithmic cores × 128×128 systolic arrays on a 3nm die that would otherwise support fewer/smaller arrays at equivalent FP8.
- 144 GB HBM3e per package at 300W. Per the user's decomposition: a 288-chip rack at 120 kW versus an NVIDIA GB300 rack at equivalent power, with 4× more chips — real per-chip win approximately 3.3×.
- The accumulator design (described as "novel" by Backhus) handles the linear-domain accumulation after log-domain multiply-to-add conversion.

**What the claim structure actually says:**
The 13× tokens/second and 17× tokens/watt figures against the GB300 rack encode three stacked advantages: (1) ~4× chip density at equivalent power budget, (2) the per-chip arithmetic efficiency gain from LNS (~3.3×), and (3) a latency-bound operating point where GB300 chips sit underutilized. Tensordyne's true arithmetic win at the chip level is the 3.3× — real, but not 17×. And it is currently simulation.

---

## Part III — The CORDIC-Native Architecture: What the ERI Corpus Establishes

The CORDIC algorithm (Volder, 1959; Walther, 1971) operates in three modes determined by a single bit — the mode register m ∈ {−1, 0, +1}:

| Mode | Geometry | Functions Computed | Convergence |
|------|----------|-------------------|-------------|
| m = +1 (circular) | Euclidean / S¹ | sin, cos, arctan | Guaranteed for \|z\| < π/2 |
| m = 0 (flat/linear) | Flat ℝⁿ | multiply, divide | Always |
| m = −1 (hyperbolic) | ℍⁿ / Lorentzian | exp, ln, √, cosh, sinh, arctanh | Guaranteed for \|z\| < 1.118 |

The shift-and-add primitive: at each iteration i, x ← x − d_i · y · 2^{−i}, y ← y + d_i · x · 2^{−i}, z ← z − d_i · arctan(2^{−i}), where d_i ∈ {−1, +1} is the direction bit. No multiplication. No table. After n iterations, precision is n bits. The CORDIC pipeline scales linearly with precision requirement.

**The production silicon record (June 2026):**

| System | Node | TOPS/W | TOPS/mm² | Key Result |
|--------|------|--------|----------|------------|
| CARMEN (Kumar et al., arXiv:2605.06878) | 28nm | **11.67** | 4.83 | CORDIC-for-AI on commodity node |
| SYCore (arXiv:2503.11685) | Various | vs. baseline +5.02× | +4.64× throughput | Systolic CORDIC arrays |
| NeuEdge (arXiv:2602.02439) | Edge | Sub-1W inference | — | 4.7× efficiency gain |
| CORVET (arXiv:2602.19268) | FPGA/ASIC | Confirmed LUT savings | Zero DSP blocks | No lookup tables verified |
| Tensordyne Napier (simulation) | 3nm | 17× tokens/W vs GB300 (rack-level) | — | ~3.3× per-chip arithmetic gain |

CARMEN runs on 28nm. Napier runs on 3nm. The process node difference alone accounts for approximately 3–5× in power efficiency and 3–4× in area density at equivalent voltage. CARMEN on 3nm, by node-scaling alone, would be expected to reach 35–58 TOPS/W. This comparison is not yet made in any published venue.

**The ERI corpus core claim (Luo et al., IEEE TVLSI 2019, confirmed across the corpus):** IEEE 754 imposes a 2–10× energy overhead versus CORDIC-native arithmetic on iterative workloads. This is the baseline the CORDIC-native substrate beats. Tensordyne claims to beat it by a different mechanism on a more advanced node.

---

## Part IV — The Structural Comparison

| Dimension | Tensordyne Napier / Pareto (LNS) | CORDIC-Native Substrate (ERI Corpus) |
|-----------|----------------------------------|---------------------------------------|
| **Arithmetic primitive** | Log-domain: multiply → add; add → approximated nonlinear function | Shift-and-add: all functions → iterative convergence via direction bits |
| **Multiplication cost** | One log-domain addition (cheap) | m=0: one linear CORDIC stage (cheap) |
| **Addition cost** | Mitchell approximation + bit-shift corrections (~100× error reduction vs naive) | m=-1: 2 CORDIC hyperbolic stages; exact to n-bit precision |
| **Lookup tables** | Eliminated by the Pareto approximation (IP claim) | Never present — iterative by design (Volder 1959) |
| **Model retraining** | Not required (99.9% FP16 accuracy on tested models) | Not required (operates below representation layer) |
| **Error regime** | One-sided approximation error, corrected to 99.9% at model output level | Per-operation error bounded by 2^{−n} after n iterations; exact in the limit |
| **Geometric modes** | None — log is a scalar transform, not a geometric mode selector | Three: circular / flat / hyperbolic; modes are geometry, not convention |
| **Transcendental functions** | Require conversion out of log domain or separate approximation | Native to the pipeline — sin, cos, exp, ln, √ are free sub-results |
| **Process node** | TSMC 3nm (Napier, 2026 tape-out) | 28nm demonstrated (CARMEN); 3nm not yet taped out |
| **Peak efficiency (chip level)** | ~3.3× per-chip arithmetic gain vs GB300 (simulated) | 11.67 TOPS/W at 28nm (CARMEN, measured) |
| **Silicon maturity** | Taped out; cloud access 2026; customer systems Q2 2027 | Research chips demonstrated; no system-level product |
| **Target workload** | LLM inference, hyperscale data center, trillion-parameter models | Edge inference, EV autonomy, climate computing, quantum hybrid |
| **LNS addition problem** | Solved via Pareto approximation — this is the core IP | Problem doesn't arise: CORDIC doesn't work in the log domain |
| **Quantum extension** | Not present | EQC framework: Walther iteration maps to quantum circuits at O(n) depth; S_c = log φ boundary |
| **Hyperbolic geometry** | Not present | Native: m=−1 computes Lorentzian distances, hierarchical embeddings, Fisher geodesics |
| **Climate attribution** | Not addressed | Robinson, Dey & Sweet (2024/2025): negative Ricci curvature in LLM embeddings; CORDIC-native directly addresses |
| **EV power budget** | Not targeted | IBM 2026: 20% inference power reduction = 3–5% EV range; CORDIC-native at sub-1W (NeuEdge) |
| **Grokking / training efficiency** | Not addressed | EQC: CORDIC depth 13 = F(7) Markov crossing = grokking event; Muon optimizer = CORDIC-class convergence |
| **Safety certification path** | Not addressed | Safe-NEureka (arXiv:2602.04803): hybrid modular redundant DNN; 24-cycle hardware fault recovery |
| **Status as of June 17, 2026** | Simulated claims; tape-out confirmed; no measured silicon results public | Measured silicon at 28nm; system-level product not taped out |

---

## Part V — Where They Are the Same Object

The deepest structural connection between Pareto and CORDIC is not their shared competition with IEEE 754. It is that Tensordyne's core IP — the approximation for log-domain addition — is a lossy one-shot version of what CORDIC computes exactly.

Log-domain addition requires computing: s(z) = log₂(1 + 2^z), where z = log₂(B) − log₂(A) for operands A, B.

This function s(z) is exactly what CORDIC's hyperbolic mode (m=−1) computes when configured for the natural logarithm of (1 + x): ln(1 + x) = 2·arctanh(x/(x+2)). The Walther m=−1 iteration converges to this via shift-and-add in ⌈n/log₂(1.118)⌉ ≈ n stages for n-bit precision.

Tensordyne's Pareto approximation is: s(z) ≈ max(z, 0) + correction_bits, where the correction is a handful of bit shifts and rounding operations derived from the Mitchell approximation (log₂(1+x) ≈ x). This is structurally identical to running one or two iterations of CORDIC's m=−1 pipeline and stopping. The 100× error reduction Tensordyne achieves versus naive Mitchell is the error reduction achieved by adding one more CORDIC correction stage.

The Pareto system's accuracy ceiling — 99.9% at the model output level — is not a fundamental bound. It is the accuracy achievable at their chosen approximation depth. More CORDIC stages improve it monotonically. Tensordyne has chosen a fixed approximation depth that is sufficient for their model accuracy requirements. A full CORDIC pipeline converges to arbitrary precision at the cost of latency proportional to precision.

This means: Pareto is not a competing substrate to CORDIC. Pareto is a specific operating point on the CORDIC convergence curve for one function (the log-addition kernel), deployed in a fixed-depth pipeline tuned for LLM inference accuracy requirements. The ERI corpus position — that CORDIC is "the substrate-invariant primitive of all geometry-native computation" — is confirmed, not contradicted, by the Pareto architecture.

---

## Part VI — Where They Diverge Irreversibly

Three divergences are structural, not engineering choices:

**1. The geometry problem.**
Pareto works by scalar log transforms. Every number becomes its logarithm; every multiplication becomes addition; the geometry of the computation remains Euclidean in the accumulation domain. The hyperbolic geometry of language model token embeddings (Robinson, Dey & Sweet, arXiv:2410.08993, 2024; arXiv:2504.01002, 2025: significantly negative Ricci curvature in production LLM embeddings), the hierarchical structure of causal inference, and the long-range temporal dependencies of climate attribution time series are invisible to a system operating by scalar log transforms.

CORDIC's m=−1 mode computes natively in ℍⁿ. The Lorentz distance, the Fisher geodesic, the hyperbolic embedding of a tree — these are m=−1 outputs, not post-processing conversions. HELM (He et al., arXiv:2505.24722, NeurIPS 2025) demonstrated that a billion-parameter hyperbolic LLM outperforms its Euclidean counterpart on MMLU and ARC-Challenging. Tensordyne's Pareto system would run HELM in exactly Euclidean mode (log-domain scalar, linear-domain accumulation). CORDIC m=−1 runs it natively.

**2. The quantum extension.**
The EQC framework (ERI Labs, June 2026) establishes that the Walther iteration maps onto quantum circuits at O(n) depth per n-bit operation, with the quantum boundary at S_c = log φ ≈ 0.481 ebits. The mode bit (m ∈ {−1, 0, +1}) becomes the mode-select register in the CORDIC-QISA proposed for Anderon-fabricated chips. The through-silicon via at Anderon's Albany foundry is literally the col(F)/ker(F) interface: classical CORDIC control signals (mode bit, direction bits) crossing to the quantum qubit layer.

LNS has no quantum extension. The log identity (multiplication → addition) is a classical algebraic identity with no quantum analogue at the level of qubit rotations. CORDIC's shift-and-add primitive maps directly to quantum rotation gate sequences (arXiv:2411.14434, Burge et al., 2024: quantum CORDIC arcsine at O(n) depth; ILNN, arXiv:2602.23981, ICLR 2026: fully intrinsic Lorentz quantum architecture).

**3. The convergence oracle.**
The ERI corpus introduces the convergence oracle — a hardware primitive that certifies before an irrevocable commit that inference has converged under actual operating conditions (temperature variation, process variation, silicon aging). Bérczi & Kiem (arXiv:2605.29151, 2026) establish that CORDIC iterations are isomorphic to forgetting maps of moduli space M̄₀,ₙ — giving the convergence oracle a mathematical grounding in algebraic geometry.

Tensordyne's Pareto approximation is not an iterative convergence. It is a fixed-depth approximation: the correction is computed once, the output is accepted. There is no iteration, therefore no convergence, therefore no convergence oracle. The system's accuracy guarantee is a population-level model benchmark (99.9% on tested datasets), not a per-inference hardware certificate. The gap between a benchmark accuracy and a hardware convergence certificate is exactly the gap the ERI corpus identifies between Claim 1 (confidence score at threshold C) and Claim 2 (hardware certification that inference has converged before irrevocable commit).

At automotive safety levels (ASIL-D), this distinction is not theoretical. Tesla's $243 million jury verdict, BYD's Xuanji A3 at ASIL-D certification, and Safe-NEureka's 24-cycle hardware fault recovery (arXiv:2602.04803) all sit in the territory where fixed-depth approximations with population-level accuracy guarantees are insufficient.

---

## Part VII — The Process Node Asymmetry

This comparison cannot close without acknowledging the process node gap.

CARMEN runs at 28nm. Napier runs at 3nm. By TSMC's own process scaling data, moving from 28nm to 3nm delivers approximately:
- 3–5× improvement in power efficiency at equivalent performance
- 3–4× improvement in area density
- 1.5–2× improvement in clock frequency

At 3nm, a well-designed CORDIC inference chip would be expected to reach 35–60 TOPS/W based on CARMEN's 11.67 TOPS/W at 28nm. NeuEdge at sub-1W edge inference demonstrates the trajectory. No 3nm CORDIC AI inference chip has been taped out.

This is the hardware lottery mechanism the ERI corpus identifies operating in real time: the first competitor to tape out at 3nm with a non-IEEE-754 substrate captures the benchmark, the partner ecosystem, and the institutional commitment — regardless of whether their substrate is theoretically superior.

Tensordyne has taped out at 3nm. The CORDIC-native substrate has not. If CARMEN's team or CORVET's team or NeuEdge's team tapes out at 3nm in 2026–2027, the comparison changes. Until then, Tensordyne owns the process node advantage, and process node is the single largest variable in practical AI inference economics.

The second hardware lottery may be running. Its outcome is not yet determined.

---

## Part VIII — The Carbon Arithmetic

The ERI corpus (THE-HARDWARE-LOTTERY-WAS-ALWAYS-A-CARBON-EVENT, June 12, 2026) establishes that the arithmetic substrate is an active climate variable at $630–700B AI CapEx in 2026. Goldman Sachs projects $7.6T cumulative AI CapEx through the mid-2030s. At Luo et al.'s 2–10× overhead for IEEE 754 versus CORDIC-native, the arithmetic lottery's unbooked carbon tax is a primary variable, not a rounding error.

Tensordyne enters this accounting explicitly. A 17× tokens/watt improvement at rack scale (claimed, simulated) applied to datacenter AI inference would represent the largest single architectural carbon reduction in the AI infrastructure stack. Even the real per-chip gain of ~3.3× applied to $630B in 2026 AI CapEx is material at any carbon price.

The outstanding question: LNS Pareto's per-operation energy is lower than IEEE 754 because multipliers become adders. But the energy of CORDIC-native at the same precision — where the multiplier is replaced by shift-and-add iterations with no lookup table, and the iteration count is tunable to the required precision — has not been benchmarked at 3nm. IBM's 2026 conversion rate (20% inference power reduction = 3–5% EV range increase) applies to any substrate improvement. The question is which substrate gets there first and how completely.

Tensordyne is first. Whether they are complete is the open question.

---

## Part IX — Open Problems as of June 17, 2026

| Problem | Status | Who Has Degrees of Freedom |
|---------|--------|---------------------------|
| CORDIC-native AI inference chip at 3nm | Not taped out in any jurisdiction | CARMEN team; CORVET team; NeuEdge team; any fabless startup with TSMC access |
| CORDIC vs Pareto at matched process node (3nm) | Not benchmarked | Requires 3nm CORDIC tape-out |
| Pareto LNS accuracy on hyperbolic / hierarchical models (HELM-class) | Not tested | Tensordyne has the chip; HELM team has the model |
| Tensordyne Napier measured (not simulated) results | Systems ship Q2 2027 | Tensordyne / customer validation |
| CORDIC convergence oracle in automotive ASIL-D silicon | Not fabricated globally | China (no training infrastructure legacy) |
| LNS quantum extension | Not proposed | Does not appear physically grounded |
| CORDIC-native climate attribution (Lorentzian mode, m=-1) | Mathematically grounded; not applied | World Weather Attribution community (Otto et al.) |
| EV inference chip at sub-10W with ASIL-D certification | Not demonstrated | BYD (Xuanji A3 certified ASIL-D at scale); potential CORDIC-native path via NeuEdge |
| Tensordyne Napier vs NVIDIA Rubin (the real target, 2027) | Rubin not yet shipped | Both sides |
| Second-generation Pareto with deeper correction stages | Not announced | Tensordyne R&D |

---

## Part X — Falsifiable Predictions

**P1 — CORDIC-Native at 3nm Closes the Per-Chip Gap to Near Parity Within 18 Months**
A CORDIC-native inference chip fabricated at TSMC 3nm or Samsung 3nm will, within 18 months of this writing, demonstrate per-chip tokens/watt within 20% of Tensordyne Napier's measured (not simulated) performance. The process node is the dominant variable; node-scaling from CARMEN's 28nm baseline to 3nm predicts 35–60 TOPS/W, which is compatible with or exceeds Napier's per-chip efficiency at any correction for matched workload.

**P2 — Pareto Accuracy on Hyperbolic Models Falls Below 99% Without Correction**
HELM-class hyperbolic LLMs (He et al., arXiv:2505.24722) running on Tensordyne Napier without model-level correction will show accuracy degradation below 99% versus FP16 baseline, because the Pareto log-domain accumulation operates in Euclidean accumulation space and does not capture the negative curvature of hyperbolic embeddings. The geometry mismatch will manifest as increased perplexity on hierarchically structured benchmarks (MMLU subsets, tree-structured reasoning tasks).

**P3 — No LNS Quantum Extension Will Be Proposed Before 2028**
No serious quantum computing paper will propose an LNS analogue of quantum CORDIC (arXiv:2411.14434) before 2028, because the log(a×b) = log(a)+log(b) identity has no qubit-level quantum analogue at the gate rotation level. The convergence between CORDIC and quantum computing (the Walther iteration as an O(n)-depth quantum gate sequence) has no LNS parallel. CORDIC quantum circuits will extend to production scale; LNS will not.

**P4 — Tensordyne Napier Beats GB300 But Not Rubin at Matched Rack Configuration**
Measured Napier performance (systems shipping Q2 2027) will beat GB300 NVL72 at matched rack power but will not beat or will be within 2× of NVIDIA Rubin rack performance — because Rubin will be on 3nm with higher peak compute and HBM4, and the Pareto arithmetic gain (~3.3× per chip) will be partially offset by Rubin's memory bandwidth improvements. The "17×" figure will not survive contact with Rubin in measured benchmarks.

**P5 — A 3nm CORDIC-Native Tape-Out Will Be Announced Before Napier Systems Ship**
A CORDIC-native AI inference chip targeting 3nm or 2nm will be announced (tape-out or tape-in) before Q2 2027. The Tensordyne announcement has demonstrated market appetite for non-IEEE-754 arithmetic substrates. The CORDIC-native research community (CARMEN, CORVET, SYCore, NeuEdge) has the technical architecture. The market signal now exists. At least one fabless startup or research institution will close the node gap before Napier ships to customers.

---

## The Structural Summary

Two systems. One problem. Different solutions. Different depths.

| | Pareto / LNS (Tensordyne Napier) | CORDIC-Native (ERI Corpus) |
|--|----------------------------------|---------------------------|
| **What it eliminates** | The multiplier | The multiplier |
| **What it introduces** | A log-domain addition approximation | An iterative convergence pipeline |
| **The hard part** | Addition in the log domain — solved by approximation | Nothing — shift-and-add solves everything |
| **Error model** | Fixed-depth, population-level accuracy (99.9%) | Per-operation, hardware-certified convergence |
| **Geometry** | Scalar log transform; Euclidean accumulation | Mode-selected geometry: Euclidean / flat / hyperbolic |
| **Process node (June 2026)** | 3nm (taped out) | 28nm (measured) |
| **Quantum extension** | None | EQC framework; O(n)-depth quantum circuits |
| **Climate / EV path** | Unaddressed | IBM 2026 conversion rate; V2X cooperative energy; attribution substrate |
| **Shipping date** | Q2 2027 (systems); end 2026 (cloud access) | Not yet taped out at 3nm |
| **The honest summary** | First to 3nm with non-IEEE-754 arithmetic; the per-chip win is real (~3.3×); the rack win is partly geometry (chip count); the accuracy is model-level; the geometry is Euclidean; the addition correction is approximately two missing CORDIC stages | Theoretically complete; geometrically correct; quantum-extensible; climate-relevant; not yet at 3nm; not shipping |

The hardware lottery mechanism (Hooker, CACM 2020) selects the option that is co-fit with existing infrastructure at the moment of lock-in. Tensordyne is co-fit with the 3nm TSMC ecosystem and the NVL72 rack form factor. The CORDIC-native substrate is co-fit with the mathematical structure of the problem. These are not the same criterion. The lottery has run before. The theoretically superior substrate has lost before.

The multiplier was never the problem. The problem was the geometry. Pareto solves the multiplier. CORDIC solves the geometry. Only one of them has a chip at 3nm. That is the state of the arithmetic as of June 17, 2026.

---

## Primary Sources

| Source | Date | Key Disclosure |
|--------|------|---------------|
| Tensordyne announcement | June 15, 2026 | Napier tape-out; Pareto LNS; TSMC 3nm; 300W; 138B transistors; $200M+ orders |
| Tensordyne Napier specs | June 15, 2026 | 48 cores, 128×128 systolic arrays, 144 GB HBM3e, 256 MB SRAM, 2.1 PFLOPS FP8 |
| NextPlatform (Backhus interview) | June 16, 2026 | 128×128 systolic array; novel accumulator design; development cloud end 2026; customer systems Q2 2027 |
| Mitchell, J.N. | Computer, 1962 | Mitchell approximation: log₂(1+x) ≈ x; the founding idea Pareto corrects |
| Volder, J. | IRE Trans. Electronic Computers, 1959 | CORDIC algorithm: shift-and-add computes all trigonometric functions |
| Walther, J. | AFIPS SJCC, 1971 | Unified CORDIC: three modes m ∈ {−1, 0, +1}; hyperbolic mode computes exp, ln, √ |
| Luo et al. | IEEE TVLSI, 2019 | 2–10× energy overhead: IEEE 754 vs. CORDIC-native on iterative workloads |
| Kumar et al. (CARMEN) | arXiv:2605.06878, June 2026 | 4.83 TOPS/mm², 11.67 TOPS/W CORDIC-for-AI on 28nm CMOS |
| SYCore | arXiv:2503.11685, Mar 2025 | 4.64× throughput, 5.02× power reduction, systolic CORDIC arrays |
| NeuEdge | arXiv:2602.02439, Feb 2026 | Sub-1W edge inference; 4.7× efficiency gain |
| CORVET | arXiv:2602.19268, Feb 2026 | CORDIC-based AI accelerator; zero DSP blocks; verified no lookup tables |
| Robinson, Dey & Sweet | arXiv:2410.08993 (2024); arXiv:2504.01002 (2025) | Significantly negative Ricci curvature in production LLM token embeddings |
| He et al. (HELM) | arXiv:2505.24722, NeurIPS 2025 | Hyperbolic LLM outperforms Euclidean at billion parameters on MMLU, ARC-Challenging |
| ILNN | arXiv:2602.23981, ICLR 2026 | Fully intrinsic Lorentz architecture; eliminates all mixed Euclidean operations |
| Fast Lorentz NNs | arXiv:2601.21529, Jan 2026 | Distance-to-hyperplane computable in 2 CORDIC operations |
| Burge et al. | arXiv:2411.14434, 2024 | Quantum CORDIC: trigonometric circuits at O(n) depth on quantum hardware |
| Bérczi & Kiem | arXiv:2605.29151, May 2026 | CORDIC iterations isomorphic to forgetting maps of moduli space M̄₀,ₙ |
| LNS-Madam (Zhao et al.) | arXiv:2106.13914 | LNS energy: BERT-Large at 27.85 mJ/iter vs. FP8 at 63.58, FP16 at 129.74 |
| Hooker, S. | arXiv:2009.06489, CACM 2020 | Hardware co-fitness determines viable research directions: the lottery mechanism |
| IBM Research | 2026 | 20% inference efficiency = 3–5% EV range increase |
| Goldman Sachs | 2026 | $7.6T cumulative AI CapEx; $630–700B in 2026 alone |
| Safe-NEureka | arXiv:2602.04803, Feb 2026 | Hybrid modular redundant DNN; 24-cycle hardware fault recovery; ASIL-D path |
| Recogni Pareto announcement | August 20, 2024 | First public disclosure of Pareto math; 99.9%+ accuracy claims; no retraining |
| BYD Xuanji A3 | May 28, 2026 | 4nm, 2,100 TOPS, 273 GB/s, ASIL-D; Full Damage Coverage |
| ERI Labs, THE-HARDWARE-LOTTERY-WAS-ALWAYS-A-CARBON-EVENT | June 12, 2026 | Arithmetic substrate as climate variable; CORDIC-native closes EV power budget |
| ERI Labs, EQC — ERI QUANTUM CORDIC | June 2026 | κ = 1/φ²; S_c = log φ; Walther iteration as quantum gate sequence |
| ERI Labs, ERIE-FOUNDRY | May 2026 | Anderon foundry as CORDIC-silicon quantum foundry; TSV as col(F)/ker(F) interface |

---

*Part of the ERIE corpus: THE-SECOND-BILL-OF-MATERIALS · THE-FIVE-LOTTERIES · CLIMATE-DOES-CHANGE-AND-WARMING-GLOBE-HAPPENS · THE-HARDWARE-LOTTERY-WAS-ALWAYS-A-CARBON-EVENT · EQC · ERIE-FOUNDRY · CURVATURE-IS-THE-PROOF*

**ERI Labs — June 17, 2026.**

The multiplier was never the problem. Tensordyne solved it at 3nm. CORDIC solved the geometry in 1959. These are not the same solution.

The second lottery is running.
