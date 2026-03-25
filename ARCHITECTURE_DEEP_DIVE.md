# AO1 — Architecture Deep Dive (Public, Sanitized)

Executive summary

This deep-dive explains the architecture and engineering trade-offs behind AO1 at a high level. It is intentionally sanitized: implementation code, formulas, and operational heuristics are private and not included here.

Scope
- High-level runtime boundaries and responsibilities
- Data flow and persistence model
- Decisioning layers (conceptual) and why they are separated
- Key engineering trade-offs and risks

Runtime boundaries
- Content runtime: responsible for page detection, data ingestion, and UI state updates.
- Background/privileged runtime: optional small surface for privileged operations (e.g., network or local-model proxying).
- Popup/operator runtime: control surface for strategy selection and case review.

Primary subsystems (conceptual)
- Ingestion/Adapters: isolate DOM parsing and platform variability.
- Normalization: map platform outputs into a single Vehicle contract for downstream logic.
- Deterministic evaluation core: computes economic and risk-related outputs from the normalized vehicle (public description only).
- Policy layer: interprets deterministic outputs into operator-facing labels and actionable guidance.
- Advisory sidecar: optional market/context signals and human-readable narrative; advisory only in this public artifact.
- Persistence: local storage of settings, saved cases, and historical loss entries used for operator memory.

Data flow (high level)
1. Page DOM and navigation state are observed.
2. Platform-specific adapter extracts listing information.
3. Extracted data is normalized to the shared Vehicle model.
4. Deterministic evaluation computes non-proprietary outputs used by the policy layer.
5. The UI reads policy results and renders an in-page HUD; operator actions may be persisted to local storage.

Engineering rationale
- Separating ingestion from normalization reduces platform-specific coupling and makes the evaluation core portable across hosts.
- Keeping the evaluation core deterministic and auditable improves repeatability and recruiter-friendly interpretability.
- Advisory signals are kept separate to avoid conflating deterministic outputs with heuristic or model-based commentary.

Sanitization notes
- No implementation files, formulas, thresholds, or production configuration are included.
- Component/module names referenced in private code are described conceptually here only.

Appendix
- This document is intentionally non-operational. For code-derived behavior and implementation details, the private repository contains the authoritative source.
