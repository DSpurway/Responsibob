# Responsibob

**An energy-aware development framework for IBM Bob — Code Energy Performance Certificates, A–G ratings, and responsible computing built into your AI-assisted development workflow.**

---

## What Is Responsibob?

Responsibob is a **Bob skill + mode** that makes code energy efficiency measurable, certifiable, and improvable. It applies an EPC-style A–G rating to codebases based on two independently assessable axes:

- **Language energy multiplier** — the structural energy floor set by your language choice (from [arXiv 2410.05460](https://arxiv.org/html/2410.05460v1))
- **Algorithmic efficiency** — the variable layer: N+1 queries, streaming vs buffering, build pipeline waste, model quantisation choices

The framework is grounded in the **Defra four-layer carbon reduction curve** (validated by real IBM Power hardware measurements) and aligned with the **CAST AIP Efficiency criterion (20140522)** — confirming that the patterns Responsibob detects are the same ones that enterprise-grade static analysis tools treat as Critical/High severity runtime anti-patterns.

> **Core insight:** Good engineering decisions and responsible computing decisions are usually the same decision. Responsibob makes that visible.

---

## Why This Matters

| Layer | Potential saving | Who controls it |
|---|---|---|
| Policy / Design (don't build it) | Up to 100% | Product / architecture |
| Architecture (consolidate, minimise) | Up to 60% | Architect |
| Infrastructure (hardware efficiency) | ~40% | Platform team |
| **Code / Algorithm — Responsibob** | **~20%** | **Developers** |

The 20% band is the one developers directly own. Responsibob makes it measurable.

**Real examples from engagements this framework has been applied to:**
- SAMCO RPG modernisation: 300 DB I/O ops → 1 op per business transaction = **F → A** (N+1 CHAIN pattern eliminated)
- IBM Power hardware refresh (E950 → E1150): 39% energy saving, 56t CO₂/year (matches Defra "design clever" band exactly)
- PassportEye demo: traditional computer vision chosen over Granite Vision LLM = **100–300× less energy**, regardless of Python's language penalty

---

## Contents of This Repo

```
.bob/
  skills/
    responsibob/
      SKILL.md            ← The Bob skill (7-step EPC generation procedure)
      reference-data.md   ← Language multipliers, Defra curve, rating model, Jevons' table
custom_modes.yaml         ← The Responsibob Bob mode (add to your .bob/settings/ folder)
responsibob-skill-and-mode-guide.html  ← Visual installation and usage guide
certificates/             ← Example EPC certificates (see note below)
```

### A note on the certificates

The `certificates/` folder contains HTML EPC certificates generated during the development of this framework — v1 through v6. They are **evolutionary artefacts**, not a polished final suite. Each version reflects where the framework was at the time:

| File | What it shows |
|---|---|
| `certificate-v1-samco-rpg.html` | First certificate — SAMCO RPG only, basic A–G scale |
| `certificate-v2-samco-and-genai.html` | Added GenAI demo stack assessment |
| `certificate-v3-two-axis-model.html` | Introduced the language × algorithm two-axis model |
| `certificate-v4-unified-definitive.html` | Full four-layer framework, Defra curve, Jevons' |
| `certificate-v5-with-infrastructure.html` | Added real IBM Power hardware data (E950→E1150) |
| `certificate-v6-power-learning-and-vault.html` | Portfolio view across 6 projects |

They are included because they show what the output looks like and how the thinking developed. The v6 certificate represents the current framework best.

---

## Installation

### Prerequisites
- [IBM Bob](https://www.ibm.com/products/bob) installed and running

### Step 1 — Install the skill

Copy the skill folder into your Bob user skills directory:

```
%USERPROFILE%\.bob\skills\responsibob\
```

So you end up with:
```
C:\Users\<you>\.bob\skills\responsibob\SKILL.md
C:\Users\<you>\.bob\skills\responsibob\reference-data.md
```

### Step 2 — Install the mode

Open (or create) your Bob custom modes file:
```
C:\Users\<you>\.bob\settings\custom_modes.yaml
```

Copy the contents of `custom_modes.yaml` from this repo and append it to that file.

### Step 3 — Reload Bob

Reload the Bob window. The **Responsibob** mode will appear in the mode picker.

---

## How to Use

### As a mode (energy-aware throughout a task)

Select **Responsibob** from the mode picker at the start of a task involving:
- IBM i / RPG modernisation
- GenAI / LLM application development
- Any codebase review where energy efficiency should be first-class
- Language or platform selection decisions

Bob will flag energy implications at every decision point and generate EPC certificates as you go.

### As a skill (on-demand in any mode)

In any mode, the skill auto-activates on terms like `energy`, `efficiency`, `sustainability`, `carbon`, `green code`, `EPC`, `responsible computing`.

**Example prompts:**
```
"Review this RPG program for energy efficiency"
"Generate a Code Energy Performance Certificate for this Node.js service"
"Which is more energy-efficient — Python or Java for this data pipeline?"
"What's the Responsibob rating for our GenAI demo stack?"
"Is there a Responsibob angle on this architectural decision?"
```

---

## The Two-Axis Model

Every assessment rates two axes independently, then combines them.

### Axis 1 — Language Multiplier (structural, fixed per language choice)

| Language | Energy (×C) | Tier |
|---|---|---|
| C, Rust, RPG | ~1.0–1.05 | Tier 1 |
| C++ | 1.34 | Tier 1 |
| Java | 1.98 | Tier 2 |
| C# | 3.14 | Tier 2 |
| JavaScript | 4.45 | Tier 3 |
| TypeScript | 21.50 | Tier 4 |
| Python | 75.88 | Tier 5 |

Source: [arXiv 2410.05460](https://arxiv.org/html/2410.05460v1). RPG estimated based on compiled Power ISA output — not in the study but consistent with Tier 1.

### Axis 2 — Algorithmic Efficiency (variable, improvable)

Key patterns, cross-referenced with CAST AIP Efficiency criterion 20140522:

| Pattern | Impact | CAST Rule |
|---|---|---|
| SQL / queries inside a loop (N+1) | Very high — F/G | 7424, 7790, 7868 (all Critical) |
| Remote / API calls inside a loop | Very high — F/G | 7962 Critical |
| Cartesian product queries | High — F | 7820, 7482 |
| SELECT * (over-fetching all columns) | High | 7344, 7530 |
| String concatenation inside loops | Medium | 7200, 7198, 7954 |
| Object instantiations inside loops | Medium | 7210, 7212 High |
| Debug logging unconditionally on | Medium | 7734 |
| Missing resource / connection release | Medium | 3612, 8104, 8108 (Critical) |
| Functions on indexed WHERE columns | Medium | 7418 |
| Implicit type conversions in WHERE | Medium | 7420 |
| UNION instead of UNION ALL | Low–medium | 7436 |
| Explicit GC calls (GC.Collect etc.) | Medium | 8154, 4694 High |
| Redundant / excessive indexes | Low | 7346, 7348 |

---

## The PassportEye Principle

The single biggest energy win available is usually **not** optimising code — it is **choosing the right tool for the job**. A traditional deterministic algorithm for a well-defined bounded task beats an LLM by 100–300×, regardless of language penalty.

> Named after an IBM demo that used OpenCV/PassportEye for passport OCR instead of Granite Vision — 100 MB vs 4–6 GB model, orders of magnitude faster, negligible compute vs the inference alternative.

This is the essence of the Responsibob message: responsible computing decisions and good engineering decisions are usually the same decision.

---

## Related Work & Credits

- **arXiv 2410.05460** — "Energy Efficiency of Programming Languages" (language multiplier source data)
- **Ned Gartside / Defra** — Carbon reduction curve (policy → design → clever → code)
- **Derek Newton / IBM** — Responsible computing framework, Application Cost Efficiency Rating Platform, DWP/Defra collaboration
- **Home Office Immigration Technology** — "Hosting Cost Efficiency Rating" A–G scale (live in production — independent parallel development of the same idea)
- **CAST AIP criterion 20140522** — Efficiency rules (independent industry validation of the algorithmic anti-patterns Responsibob detects)
- **IBM Power Energy Estimator** — E950→E1150 hardware data (39% energy saving / 56t CO₂/year)

---

*Made with IBM Bob*
