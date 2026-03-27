# AO1 — Public Architecture Portfolio

This repository is a documentation-only, public-facing portfolio artifact that explains the architecture and system-design thinking behind AO1. It intentionally does not include any source code, build artifacts, or implementation files. The full implementation and operational logic remain private.

Purpose
- Present AO1 as a deterministic, architecture-first decision-support system for real-time acquisition workflows.
- Demonstrate system boundaries, data flows, and engineering trade-offs for recruiters and engineering managers.
- Protect proprietary implementation and operational details — this repo is for inspection only.

Quick links
- [One-Page System Diagram](./ARCHITECTURE_ONE_PAGER.md)
- [Architecture Deep Dive](./ARCHITECTURE_DEEP_DIVE.md)
- [Engineer Handoff (sanitized)](./docs/ENGINEER_HANDOFF.md)

Screenshot

<p align="center">
  <!-- Place `ao1-wxt-hud-screen.png` in `assets/` before publishing -->
  <img src="./assets/ao1-wxt-hud-screen.png" alt="AO1 HUD screenshot" width="600"/>
</p>

What is intentionally omitted
- No source code, no runtime artifacts, no extension code, and no production configuration are included.
- No proprietary decision formulas, thresholds, or operational heuristics are published.
- This is not an installable or runnable repository — it is a design and architecture portfolio.

Audience guidance
- Recruiters and engineering managers: read the one-pager for a 2-minute overview, then open the deep-dive for a structured, code-derived architectural report.

Notes for maintainers
- Keep this repository docs-only. When updating, do not paste implementation snippets or build files.
- Only sanitized screenshots and high-level pseudocode are permitted.
 
## Recruiter-Facing Architecture Summary

AO1 is a deterministic decision-support system for vehicle acquisition workflows. It is designed to reduce ambiguity and enable fast, auditable decisions at the point of inspection on auction platforms. This repository documents the system design, not the implementation.

TL;DR — what to look for (2-minute pass)
- One-page diagram: runtime boundaries, primary data flow, and interfaces.
- Deep-dive: stage-by-stage responsibilities (ingestion, normalization, cost aggregation, deterministic evaluation, risk augmentation, policy).
- Deterministic guarantees: outputs are reproducible and auditable; no hidden stochastic decisions.

How to review (recommended order)
1. Open the [One-Page System Diagram](./ARCHITECTURE_ONE_PAGER.md) for a quick runtime sketch.
2. Read the sanitized [Architecture Deep Dive](./ARCHITECTURE_DEEP_DIVE.md) for stage-level responsibilities and trade-offs.
3. Inspect the `docs/ENGINEER_HANDOFF.md` for maintenance notes (sanitized).

What this repo demonstrates
- Clear separation of concerns between ingestion, normalization, deterministic evaluation, and policy layers.
- Auditability: evaluation artifacts and provenance are persisted to enable reproducible reviews.
- Operational thinking: how to keep advisory/model signals separate from authoritative decision outputs.

IP and safety notes
- This repo intentionally omits any code, numeric formulas, thresholds, or private heuristics. The private repository contains the full implementation and is not public.

## Demonstration of System Behavior

The following examples illustrate how AO1 processes real-world inputs and produces structured, deterministic outputs. All values are representative and slightly rounded for clarity. Proprietary formulas and thresholds are intentionally omitted.

All examples are derived from real system outputs, with values slightly normalized for clarity and to protect proprietary logic.

### Example 1 — Moderate Margin, Controlled Risk

**INPUT**

- Vehicle: 2017 Fiat 124 Spider Classica  
- Mileage: ~102,000 miles  
- Platform: IAA  
- Condition: Moderate wear, typical auction uncertainty  

**OUTPUT**

- **Decision:** WATCH  
- **Bid Cap:** ~$1,900  
- **Estimated Exit Value:** ~$5,800  
- **Headroom:** ~$3,400  
- **Risk Tier:** LOW  

**Reasoning**

- Projected resale value provides sufficient margin, but not enough to justify immediate action  
- Cost stack remains within acceptable bounds, though limited buffer reduces aggressiveness  
- No major risk indicators present, but margin compression suggests monitoring rather than targeting  

### Example 2 — High Headroom, Opportunistic Candidate

**INPUT**

- Vehicle: 2016 BMW X5 xDrive40e  
- Mileage: ~148,000 miles  
- Platform: IAA  
- Condition: Higher mileage, premium segment  

**OUTPUT**

- **Decision:** WATCH  
- **Bid Cap:** ~$4,600  
- **Estimated Exit Value:** ~$8,800  
- **Headroom:** ~$6,100  
- **Risk Tier:** LOW  

**Reasoning**

- Strong headroom relative to acquisition cost indicates potential opportunity  
- Elevated mileage introduces uncertainty, limiting immediate classification as a target  
- Maintained as a monitored candidate pending price movement or additional signals  

### Example 3 — Low Margin, Rejected Opportunity

**INPUT**

- Vehicle: 2010 Honda Accord LX  
- Mileage: ~120,000 miles  
- Platform: IAA  
- Condition: Standard wear  

**OUTPUT**

- **Decision:** PASS  
- **Bid Cap:** $0  
- **Estimated Exit Value:** ~$3,300  
- **Headroom:** ~$1,400  
- **Risk Tier:** LOW  

**Reasoning**

- Margin does not meet minimum acceptable threshold  
- Limited headroom reduces ability to absorb unexpected costs  
- System rejects opportunity to enforce disciplined acquisition criteria

Contact / next steps
- If you want a short walkthrough or an interview-oriented narrative, I can prepare a 2-slide summary and a short speaking script tailored for hiring managers.

 
## What you will find here

- A one-page system diagram for a quick 2-minute overview.
- A sanitized deep-dive that explains runtime boundaries, data flow, and deterministic guarantees.
- A short set of behavior demonstrations (three realistic examples) showing INPUT → OUTPUT and human-readable rationale.

This repository is a documentation artifact only — it is not runnable, and it deliberately omits implementation details and numeric logic.
