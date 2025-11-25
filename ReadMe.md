# AI-Assisted SDLC for Research — Overview & Repo Guide

## What we’re doing

We’re building AI-assisted tooling that helps researchers turn early ideas into working products with minimal friction. The workflow is simple and repeatable: start from approved requirements, generate clear architecture and design, then produce a small, runnable implementation slice—measurable, traceable, and easy to extend.

## Why this matters

* Faster path from concept → blueprint → running code
* Decisions, boundaries, and security are explicit and testable
* Same process runs across our three research products for easy comparison

---

## Phases at a glance

**Requirements (inputs)**
AI-generated, IEEE-aligned SRS and INVEST-checked user stories provide the authoritative “what” and “why”.

**Architecture & Design (blueprint)**
C4 Context/Container diagrams, ADRs, Use Case diagram, STRIDE summary, and sequence diagrams translate requirements into a clear, testable plan.

**Implementation (starter code)**
A small, end-to-end slice (request → validation → processing → persistence → response) with contracts, tests, Docker, and a Quickstart. Focused, not overbuilt.

---

## Repository structure

```
/requirements/               # SRS and User Stories (inputs)
/ArchitectureAndDesign/      # C4, ADRs, Use Cases, STRIDE, Sequences (by project and AI tool)
/Implementation/             # Runnable slices (service, tests, API spec, ops, README)
/traceability/               # RTM linking requirements ⇄ decisions ⇄ diagrams ⇄ endpoints ⇄ tests
/prompts/                    # Exact prompts used (reproducibility)
```

Each **project** (e.g., `AnimalEcology`, `AOOCT`) has its own subfolder under `ArchitectureAndDesign/` and `Implementation/`.
Inside each project, artifacts are grouped by **AI tool** (ChatGPT, Claude, Gemini, etc.) with files numbered in reading order:
`01-context`, `02-container`, `03-adrs`, `04-usecases`, `05-stride`, `06-sequences`, `07-rtm.csv`.

---

## How to read this repo (quick path)

1. **Start with inputs** → open `/requirements/` to see what the system must do.
2. **Understand the blueprint** → in `/ArchitectureAndDesign/<Project>/<Tool>/`, read in order:
   `01 Context` → `02 Container` → `03 ADRs` → `04 Use Cases` → `05 STRIDE` → `06 Sequences`.
3. **Run the slice** → open `/Implementation/<Project>/<Tool>/README.md` and follow **Quickstart**
   (one command to run, sample `curl` calls, one command to test).
4. **Verify links** → use `/traceability/rtm.csv` to jump from a requirement ID to the related ADRs, diagrams, endpoints, and tests.
5. **Compare AI tools** → open the same numbered file (e.g., `03-adrs.md`) across tool folders.

---

## Artifact guide (what and why)

* **C4 — System Context**: shows the system in its environment (actors, externals, relationships).
* **C4 — Container**: shows deployable parts and data stores with responsibilities and key interfaces.
* **ADRs**: record important choices, alternatives, trade-offs, and revisit criteria.
* **Use Case diagram**: keeps functional intent visible and tied to stories.
* **STRIDE summary**: lists threats at each boundary with planned mitigations.
* **Sequence diagrams**: make key flows concrete (success, retries, errors).
* **Implementation slice**: minimal service + OpenAPI slice + tests + Dockerfile + Quickstart.

---

## How we build each implementation slice (high level)

* **Inputs**: Project Context, SRS, User Stories, and the Architecture & Design workproducts
* **Select a few stories**: highest value or risk
* **Keep names consistent**: exactly match C4/ADRs across code and tests
* **Ship a complete starter**: contracts, code, tests, Docker, README—runnable as-is

---

## How we measured quality

[We shd refer to IBM report and write a comprehensive content here]

---

## References (slide PDFs)

* **Inception Phase PDF:** [Need to add link](sandbox:/mnt/data/Architecture%20Phase.pdf)
* **Requirement analysis Phase PDF:** [Need to add link](sandbox:/mnt/data/Architecture%20Phase.pdf)
* **Architecture Phase PDF:** [Need to add link](sandbox:/mnt/data/Architecture%20Phase.pdf)
* **Implementation Phase PDF:** [Need to add link](sandbox:/mnt/data/Implementation_phase%20%281%29.pdf)

---

## Where to start

* Repo home: [https://github.com/GautamMg/AI4SE](https://github.com/GautamMg/AI4SE)
* Quick tour: read `/requirements/`, then one project under `/ArchitectureAndDesign/`, then run its slice in `/Implementation/`

That’s the whole story: requirements in, architecture & design to clarify, a small runnable slice to prove the path, and simple metrics so we know it’s working.
