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
