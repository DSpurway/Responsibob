---
name: responsibob
description: >
  Use when the user asks about energy efficiency, responsible computing, sustainable code,
  or carbon impact of software. Also use when reviewing language or platform choices where
  energy matters, or when generating a Code Energy Performance Certificate (EPC) for any
  codebase. Activates automatically when terms like "energy", "efficiency", "sustainability",
  "carbon", "responsible computing", "green code", or "EPC" appear in context.
---

# Responsibob — Code Energy Performance Skill

## Purpose

Generate rigorous, evidence-grounded energy efficiency assessments for code and systems.
Produce A–G rated Code Energy Performance Certificates (EPC-style) and advise on
language, platform, and algorithmic choices from an energy perspective.

This skill covers:
- EPC certificate generation for any codebase or stack
- Language/platform selection guidance with energy multipliers
- Algorithm efficiency assessment (ops per transaction, streaming, batching)
- Infrastructure layer context (hardware efficiency, utilisation)
- The four-layer Responsibob framework (Policy / Infrastructure / Algorithm / AI model selection)

---

## Reference Data

Before generating any certificate or making energy recommendations, read the reference data file:

```
.bob/skills/responsibob/reference-data.md
```

This file contains:
- Language energy multipliers table (arXiv 2410.05460)
- Defra carbon reduction curve with real IBM Power measurements
- Two-axis rating model (language × algorithm)
- A–G scale descriptor
- IBM Power hardware context (E950→E1150 data)
- Jevons' Paradox risk table
- Platform energy positions

**Do not re-reason these numbers from scratch.** Load them from the reference file.

---

## Step 1 — Understand What Is Being Assessed

Identify which of these assessment types applies:

**A. Certificate generation** — user wants an EPC for a specific codebase or stack.
Gather: language(s) used, key operations performed (DB queries, inference calls, API calls),
platform/hardware, any known performance figures (ops/transaction, response times).

**B. Language/platform choice** — user is choosing between languages or platforms.
Gather: the task's compute profile (is it CPU-intensive, I/O-intensive, small/large volume?),
whether the language choice is fixed or open, and the deployment target.

**C. Retrospective review** — user wants to understand the energy impact of work already done.
Gather: what was built, what language/framework choices were made, what the workload profile is.

**D. Ongoing task advisory** — user is in the middle of building something and wants to
keep energy efficiency front of mind.
This is the "what would have changed the GenAI demo" scenario. Provide advisory notes
at decision points: language choice, model selection, build pipeline, streaming vs buffering.

If unclear, ask one focused question to determine the type before proceeding.

---

## Step 2 — Assess the Two Axes

For every component in scope, assess independently:

### Axis 1 — Language Multiplier (load reference-data.md for the table)

Identify the language(s) used. Look up the energy multiplier. Note:
- This is structural and fixed — it cannot be improved without changing language
- It is the floor, not the ceiling — algorithmic waste sits on top
- For mixed stacks (e.g. C++ inference + TypeScript UI), assess each layer separately
- State the tier: Tier 1 (×1–1.4), Tier 2 (×1.4–6), Tier 3 (×4–8), Tier 4 (×8–30), Tier 5 (×30+)

### Axis 2 — Algorithmic Efficiency

Look for these specific patterns, ordered by energy impact.

These patterns align with CAST AIP business criterion 20140522 (Efficiency) — the same rules
that automated static analysis tools flag as runtime performance anti-patterns. Where a CAST
rule ID is given, it confirms industry-wide recognition of the pattern.

**High impact (rate F–G if present):**
- N+1 database query patterns (individual CHAIN/READ per row instead of SQL JOIN/cursor)
  → CAST 7424 Critical: "Avoid using SQL queries inside a loop"
  → CAST 7790 Critical: "Avoid Cursors inside a loop"
  → CAST 7868 Critical: "Avoid Open SQL queries in loops" (SAP/ABAP)
- Remote/API calls inside a loop (HTTP, RPC, external service per iteration)
  → CAST 7962 Critical: "Avoid direct or indirect remote calls inside a loop"
- Cartesian product queries (unjoined tables producing row explosion)
  → CAST 7820: "Never use SQL queries with a cartesian product"
  → CAST 7482 High: "Do not use Cartesian products in Universe properties"
- Full-precision AI models where quantised alternatives exist (e.g. fp16 when Q4_K_M is available)
- Compile-from-source on every deployment when pre-built binaries exist (e.g. llama.cpp vs Ollama)
- Synchronous blocking where streaming/async would release resources earlier
- SELECT * queries — fetching all columns when only a subset is needed
  → CAST 7344: "Avoid 'SELECT *' queries" / CAST 7530 (SAP variant)

**Medium impact (rate D–E if present):**
- Debug/verbose logging always on in production (logs every request, full headers)
  → CAST 7734: "Avoid using debug() method without calling isDebugEnabled()"
- String concatenation inside loops (creates object churn, GC pressure)
  → CAST 7200: "Avoid String concatenation in loops"
  → CAST 7198 (.NET), CAST 7954 High (indirect concatenation)
- Object/class instantiations inside loops (avoidable allocation per iteration)
  → CAST 7210 High, CAST 7212 High: "Avoid instantiations inside loops"
- Untuned inference parameters (no thread count, no context window, no batch size)
- Unconditional package installs on re-deployment (e.g. `dnf update` every time)
- Large framework/dependency installs where only a small subset is used
  (e.g. full PyTorch installed solely for a shared library path)
- Missing resource release — open connections, file handles, streams not closed
  → CAST 3612 Critical, CAST 8104 Critical: "Avoid missing release of SQL connection"
  → CAST 8108 Critical: "Avoid missing release of stream connection"
  → CAST 7692: "Each opened file must be closed"
- Forcing UNION (with implicit sort/dedup) where UNION ALL is sufficient
  → CAST 7436: "Prefer UNION ALL to UNION"
- Forcing GC explicitly (GC.Collect / System.gc calls)
  → CAST 8154 High: "Avoid using GC.Collect()"
  → CAST 4694 High: "Avoid using 'System.gc' and 'Runtime.gc'"
- Functions on indexed columns in WHERE clause (defeats index, forces full scan)
  → CAST 7418: "Avoid SQL queries using functions on indexed columns in the WHERE clause"
- Implicit type conversions in WHERE clause (defeats index, forces full scan)
  → CAST 7420: "Avoid SQL queries with implicit conversions in the WHERE clause"
- Method invocation in loop termination expression (re-evaluated every iteration)
  → CAST 7204: "Avoid method invocation in a loop termination expression"
- Calling properties that clone values inside loops
  → CAST 7352: "Avoid calling properties that clone values in loops"
- Nested loops without justification (O(n²) or worse compute complexity)
  → CAST 7880: "Avoid nested loops"

**Low impact (rate C or note as minor):**
- Missing memoisation for expensive recomputations (e.g. prompt rebuilding on every render)
- Bundle size not tree-shaken (e.g. Carbon React without transpilePackages)
- Client instantiated multiple times per session instead of once as singleton
- Redundant or too many database indexes (write amplification)
  → CAST 7346, CAST 7348: "Avoid redundant indexes" / "Avoid too many Indexes on one Table"

**Good patterns (note positively, raise the rating):**
- SQL JOINs instead of multiple sequential queries
- Streaming inference (reduces peak RAM, improves time-to-first-token)
- Quantised models (Q4_K_M, Q8_K_M) over full precision
- PassportEye principle: traditional deterministic algorithm chosen over LLM for bounded structured task
- useMemo / memoisation for prompt/message construction
- Pre-built binaries over compile-from-source
- UNION ALL instead of UNION where duplicates are acceptable
- Explicit resource release (connection.close(), stream.close() in finally blocks)

---

## Step 3 — Determine the Combined Rating

Use the two-axis model from reference-data.md.

Combined rating guidelines:
- Both axes A → A overall
- One axis A, other F → combined rating roughly D–E (the F drags the combined down)
- The PassportEye exception: G language + A algorithm can still beat a better language
  with an F algorithm *if* compute volume differs by orders of magnitude
- Always state both axes separately — the combined rating alone loses important information

---

## Step 4 — Apply the Defra Curve Context

Position the assessment within the four-layer framework (from reference-data.md):

1. **Policy / Design (up to 100%)** — was this work necessary? Could existing systems serve the need?
2. **Infrastructure (up to ~40%)** — is the hardware appropriate and efficiently utilised?
3. **Algorithm / Code — Responsibob (up to ~20%)** — this is what we are assessing
4. **AI model selection** — is AI the right tool, or is a deterministic algorithm better?

Always acknowledge that code efficiency (Layer 3) is the 20% band. Point to Layers 1 and 2
when a larger lever is available and relevant.

---

## Step 5 — Assess Jevons' Paradox Risk

Apply the differentiated risk table from reference-data.md. Do NOT apply a blanket warning.

- Operational system not at capacity ceiling → low short-term risk; note medium-term monitoring
- GenAI inference → higher risk (cheaper tokens tend to unlock more usage)
- Bounded deterministic task → negligible risk
- New features enabled by efficiency headroom → note as value creation, not Jevons' Paradox

---

## Step 6 — Generate the Output

### For a full EPC certificate (use create_html_artifact):

Use the `create_html_artifact` tool. Structure:

1. **Header** — project name, language, platform, date, Responsibob version
2. **Rating scale** — A–G bars with current and potential ratings marked
3. **Component breakdown table** — each component: language axis rating / algo axis rating / combined / key finding / evidence (file name and line if available)
4. **Recommendations** — ordered by energy impact (High / Medium / Low / Future), each grounded in specific code evidence
5. **Methodology note** — state what was measured, what was estimated, what the limits are
6. **Defra curve positioning** — show where this work sits in the four layers
7. **Jevons' caveat** — differentiated per context

Design: follow the Responsibob v5 certificate style (see `responsibob/certificate-v5-with-infrastructure.html` for reference). A–G colour scale: A=#008054, B=#19a643, C=#8dc21f, D=#ffd500, E=#f4aa00, F=#e8680a, G=#c8181f.

### For language/platform guidance (inline response):

Provide a concise table:

| Option | Language | Multiplier | Tier | Algo opportunity | Combined verdict |
|---|---|---|---|---|---|

Then explain the trade-offs. Always note:
- Language multiplier is the floor; algorithm is the variable
- Tier 4–5 languages are often fine if the compute volume per call is tiny
- The PassportEye principle: choosing a deterministic algorithm over an LLM is usually the biggest energy win available regardless of language

### For ongoing advisory (inline notes):

At each decision point flag:
- Language being chosen and its tier
- Whether streaming/async is available and being used
- Whether the model/library choice is the lightest appropriate option
- Whether the build/deploy pipeline has unnecessary energy cost

---

## Step 7 — Save Outputs Per the Checkpoint Rule

If this is part of a multi-phase task (per `.bob-rules.md`):
- Save certificates as HTML files in a project subdirectory (e.g. `responsibob/`)
- Update or create `_checkpoint.md` with the new artefact paths
- Note the combined rating and top 3 improvements in the checkpoint

---

## Quick Reference — Most Common Findings

| Pattern found | Impact | Rating hit | Fix |
|---|---|---|---|
| N+1 RLA/CHAIN queries | Very high | F on algo axis | Replace with SQL JOIN or cursor |
| LLM for structured task | Very high | F on algo axis | Evaluate deterministic algorithm (PassportEye principle) |
| Compile from source on each deploy | High | D on algo axis | Switch to pre-built binary (Ollama vs llama.cpp) |
| stream:false on LLM call | Medium | D on algo axis | Switch to stream:true; use onToken callback |
| Debug logging always on | Medium | D on algo axis | Gate behind env var; reduce logLevel |
| Full framework for small subset | Medium | D on algo axis | Target install (e.g. openblas-devel vs PyTorch) |
| TypeScript UI (thin layer) | Structural E | E on lang axis | Acceptable if UI does minimal compute |
| Python for small bounded task | Structural G | G on lang axis, but algo may compensate | Evaluate if deterministic task; Python+A algo can still beat TS+F algo |
| useMemo missing on expensive compute | Low | C on algo axis | Add memoisation |
