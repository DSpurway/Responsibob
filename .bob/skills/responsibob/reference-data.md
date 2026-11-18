# Responsibob Reference Data

Core reference tables for the Responsibob energy efficiency framework.
Load this file when generating any certificate or making language/platform choices.

---

## Language Energy Multipliers (arXiv 2410.05460)

Energy normalised to C = 1.00. Lower = more efficient.
RPG is estimated — not in the arXiv study but consistent with Tier 1 based on compiled Power ISA output.

| Language     | Energy (×C) | Tier   | Notes                                      |
|---|---|---|---|
| C            | 1.00        | Tier 1 | Baseline                                   |
| Rust         | 1.03        | Tier 1 |                                            |
| C++          | 1.34        | Tier 1 | llama.cpp inference engine                 |
| RPG          | ~1.05       | Tier 1 | Estimated. IBM i / Power ISA compiled code |
| Ada          | 1.70        | Tier 1 |                                            |
| Java         | 1.98        | Tier 2 |                                            |
| Scala        | 2.11        | Tier 2 |                                            |
| Go           | 3.23        | Tier 2 |                                            |
| C#           | 3.14        | Tier 2 |                                            |
| Haskell      | 4.20        | Tier 3 |                                            |
| JavaScript   | 4.45        | Tier 3 | Node.js proxy server                       |
| TypeScript   | 21.50       | Tier 4 | Next.js / React UI layer                   |
| PHP          | 29.30       | Tier 4 |                                            |
| Lua          | 45.98       | Tier 4 |                                            |
| Python       | 75.88       | Tier 5 | PassportEye OCR service                    |
| Ruby         | ~100+       | Tier 5 |                                            |

**Key insight:** Language multipliers represent the structural energy floor — unchangeable for a given language choice. Algorithmic efficiency is the variable layer on top. When compute volumes differ by orders of magnitude (e.g. LLM vs traditional CV), algorithmic choice dominates over language choice.

---

## Defra Carbon Reduction Curve

Credit: Ned Gartside (Defra), cited in Derek Newton / DWP (Oct 2024).
Validated by IBM Power Energy Estimator: E950→E1150 hardware refresh = 39% measured (Defra "design clever" = 40% theoretical).

| Band                    | Potential saving | Layer              | Example                                    |
|---|---|---|---|
| Design & develop nothing | Up to 100%      | Policy / Strategy  | Challenge the need; use existing systems   |
| Design to use less       | Up to 60%       | Architecture       | Consolidation; minimise scope              |
| Design clever            | ~40%            | Infrastructure     | Hardware refresh E950→E1150 = 39% measured |
| Develop efficiently      | ~20%            | Code (Responsibob) | ART200→ART400 = 300× fewer I/O ops        |

**Responsibob position:** "Develop efficiently" band (20%). This is the band developers directly control. Policy and design decisions outweigh it — Responsibob should acknowledge this and point upward to larger levers when relevant.

---

## Two-Axis Rating Model

Every assessment should rate on both axes independently, then combine.

### Axis 1 — Language Multiplier (structural, fixed per language)
Rate A–G based on the language multiplier table above:
- A: ×1.0–1.4 (C, Rust, RPG, C++)
- B: ×1.4–3.0 (Java, Scala, Go, C#)
- C: ×3.0–6.0 (Haskell, JavaScript)
- D: ×6.0–15× (mid-tier scripting)
- E: ×15–30× (TypeScript, PHP)
- F: ×30–60× (Lua range)
- G: ×60+ (Python, Ruby)

### Axis 2 — Algorithmic Efficiency (variable, improvable)
Rate A–G based on operations per business transaction or equivalent metric:
- A: Near-optimal — set-based, single query, no unnecessary work (e.g. ART400: 1 I/O op)
- B: Minor optimisation available — streaming used, ctx tuned, logging gated
- C: Some inefficiency — cursor-based SQL, untuned inference, minor redundancy
- D: Moderate inefficiency — debug logging always on, no streaming, untuned parameters
- E: Significant waste — unnecessary repeated operations, unconditional heavy installs
- F: Major inefficiency — N+1 patterns, full-precision model when quantised available (e.g. ART200: 300 ops)
- G: Extreme inefficiency — thousands of individual operations, no batching, bloated model choices

### Combined Rating
Combined = product of both axes (approximately). The axis with the worse rating dominates.
A G-language with an A-algorithm can still beat an E-language with an F-algorithm if the compute volumes differ by an order of magnitude (PassportEye principle).

---

## A–G Rating Scale (EPC-style)

| Band | Score  | Descriptor                                                      |
|---|---|---|
| A    | 92–100 | Excellent — near-optimal on both axes                          |
| B    | 81–91  | Good — minor improvements available                             |
| C    | 69–80  | Above average — some inefficiency present                       |
| D    | 55–68  | Average — meaningful improvements available                     |
| E    | 39–54  | Below average — significant inefficiency                        |
| F    | 21–38  | Poor — major inefficiency; priority for improvement             |
| G    | 1–20   | Very poor — severe inefficiency across one or both axes         |

---

## IBM Power Hardware Context

From IBM Power Energy Estimator (ibm.com/it-infrastructure/resources/energy-estimator):

| Comparison         | Source: 4× E950 | Target: 2× E1150 | Saving        |
|---|---|---|---|
| Active rPerf       | 3,482           | 3,482 (same)     | —             |
| Total energy (W)   | 23,632          | 14,560           | 39% / 9,072W  |
| CO₂ saved/year     | —               | —                | 56t           |
| Energy cost saved  | —               | —                | £15,894/year  |
| Rack footprint     | 16U             | 8U               | 50%           |
| Perf headroom      | —               | 76% free         | Consolidation |

IBM Power hardware is among the most energy-efficient server platforms for IBM i and AIX workloads.
Staying on Power (rather than migrating to x86 cloud) while refreshing hardware generations is the most energy-efficient path for this class of workload.

---

## Jevons' Paradox — Differentiated Risk

The risk that efficiency improvements lead to increased consumption by unlocking latent or new demand.

| Context                              | Risk level | Rationale                                                             |
|---|---|---|
| Operational system, not at capacity  | Low (ST)   | No latent demand waiting; efficiency gain is largely real             |
| Efficiency enables new features      | Medium (MT)| New demand over 1–2 years; not classical Jevons but worth monitoring  |
| GenAI inference — cheaper tokens     | Higher     | Cheaper AI tends to expand scope of use; new demand created by cost   |
| Bounded deterministic task (PassportEye) | Negligible | Volume determined by business process, not compute cost            |

ST = short term. MT = medium term.

---

## Platform Energy Positions

| Platform / Stack         | Energy profile                                             | Notes                                              |
|---|---|---|
| IBM i on Power           | Tier 1 language (RPG), efficient hardware, SLIC-optimised | Most efficient for IBM i/AIX workloads              |
| IBM Power Linux          | Efficient hardware (Power ISA), depends on language       | Better per-watt than x86 for same workload          |
| x86 cloud (generic)      | Varies; typically less efficient than Power for i/AIX     | Embodied carbon of migration often overlooked       |
| llama.cpp on Power       | C++ (Tier 1) + OpenBLAS PPC64LE = near-optimal inference  | Q4_K_M GGUF model is the right quantisation choice  |
| Ollama on Power          | Same inference efficiency; removes build-time energy      | Pre-built binary eliminates 10-15 min compile/deploy|
| Node.js proxy (thin)     | JS (Tier 3); acceptable if proxy is genuinely thin        | Verbose debug logging can add meaningful I/O burden  |
| Next.js / TypeScript UI  | TS (Tier 4: 21.5×); acceptable if UI does minimal compute | Worst case if heavy client-side logic is added       |
| Python service           | Python (Tier 5: 75.9×); acceptable for small bounded tasks| PassportEye principle: algorithm wins at tiny scale  |
