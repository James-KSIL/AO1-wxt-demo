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

## 1. System Overview

AO1 is a deterministic decision-support system designed to help an operator evaluate vehicle acquisition opportunities directly on auction pages. It ingests platform-specific listing data, normalizes it into a stable vehicle contract, computes a deterministic economic and risk assessment, and surfaces an operator-facing label (TARGET / WATCH / PASS) alongside readable rationale and persisted case metadata.

Key properties:
- Deterministic: the same normalized input produces the same evaluation and label.
- Auditable: evaluation outputs and intermediate artifacts are persisted for later review.
- Architecture-first: ingestion, normalization, evaluation, policy, and advisory layers are separated to reduce coupling and protect repeatability.

## 2. Problem Definition

Real auction decision-making is constrained by:
- Unstructured and platform-specific vehicle data embedded in page DOMs.
- Inconsistent human decisions under time pressure.
- High risk from mis-evaluated purchases (repair, transport, reconditioning cost overruns).
- Need for fast, repeatable decisions at scale.

AO1 addresses these by producing a compact, deterministic evaluation that reduces ambiguity and supports consistent operator actions.

## 3. System Architecture (High-Level)

Pipeline: Input → Normalization → Cost Stack → Evaluation → Risk Layer → Decision Output

Stage: Input (Ingestion)
- Responsibility: detect page context, select platform adapter, and extract raw listing fields.
- Input shape: raw DOM-derived fields (title text, VIN, mileage hint, visible damage cues, location, seller metadata).
- Output shape: adapter payload containing typed fields and provenance metadata (timestamp, host, selector used).
- Constraints: must be robust to DOM drift, operate with low latency, and avoid privileged network access in-page.

Stage: Normalization
- Responsibility: map adapter payload into a single Vehicle contract used by downstream stages.
- Input shape: adapter payload with heterogeneous fields.
- Output shape: `Vehicle` (identity, year/make/model, mileage estimate, condition cues, disclosures, location, images, provenance).
- Constraints: normalization must preserve provenance and flag unverifiable claims.

Stage: Cost Stack (Aggregation)
- Responsibility: assemble non-proprietary cost buckets (transport placeholder, estimated reconditioning class, platform fees placeholder) into an aggregated cost summary object.
- Input shape: normalized Vehicle + session context (strategy profile, location band).
- Output shape: costStack {transportEstimate, feeEstimate, reconClass, hardCostsSummary} (non-numeric placeholders public-facing).
- Constraints: publicly described as aggregated categories, not numeric formulas.

Stage: Evaluation (Deterministic Core)
- Responsibility: compute an auditable evaluation artifact (exit signal, margin signal, headroom class) from the normalized vehicle and costStack.
- Input shape: Vehicle, costStack, optional historical flags from local persistence.
- Output shape: evaluation {exitSignalClass, marginSignalClass, headroomClass, reasons[] }.
- Constraints: no stochastic processes; purely deterministic mapping from inputs to evaluation artifact.

Stage: Risk Layer
- Responsibility: augment evaluation with observable risk flags (e.g., probable hidden damage indicators, disclosure mismatches, platform-specific risk markers) using historical loss-memory lookups.
- Input shape: evaluation, Vehicle, lossMemory snapshots.
- Output shape: riskAssessment {riskLevelClass, riskFlags[], confidenceBand} (classes/flags only; no probabilities published here).

Stage: Decision Policy
- Responsibility: map evaluation + riskAssessment + strategyProfile into an operator-facing label: `TARGET`, `WATCH`, or `PASS`.
- Input shape: evaluation, riskAssessment, strategyProfile.
- Output shape: decision {label, rationale[], recommendedNextAction}.
- Constraints: policy is deterministic and auditable; advisory signals remain separate.

## 4. Deterministic Guarantees

- The system enforces determinism: identical normalized Vehicle + costStack + strategyProfile + persisted operator memory produce identical evaluation and decision outputs.
- No hidden state: all inputs that influence output are stored or derivable from persisted artifacts and explicit session state.
- No stochastic decision-making: randomness or probabilistic selection is not used in final decision outputs.
- Reproducibility: evaluation artifacts are persisted with provenance metadata so any evaluation can be replayed and audited.

## 5. Role of AI (Strictly Bounded)

- AI components (if present) provide advisory commentary, classification assistance, and human-language explanations only.
- AI outputs are never used as primary inputs to the deterministic core; they are surfaced in the HUD as contextual narrative or flagged suggestions.
- The deterministic evaluation and policy layers always own the final label.

## 6. Demonstration of System Behavior (CRITICAL)

Below are three realistic, sanitized examples that illustrate how AO1 transforms inputs into a decision and human-readable rationale. Numbers are high-level categories or placeholders; proprietary numeric calculations are intentionally omitted.

Example 1 — TARGET
INPUT:
- Year: 2018
- Make/Model: Honda Civic
- Mileage: 42,000
- Condition cues: clean title indicator, minor cosmetic damage, drivable
- Location: regional auction A

OUTPUT:
- Decision: TARGET
- Cost stack summary: transport: low-band; fees: nominal; recon class: light-repair
- Margin signal: healthy-margin-class
- Risk flags: none (no disclosure mismatch; low historical loss signal)

Reasoning (human-readable):
- Vehicle normalizes to a low-mileage compact sedan with minimal visible damage. Aggregated cost categories indicate modest outlay for transport and light reconditioning. Historical loss-memory contains low incident counts for similar make/model/transmission in this region. The deterministic evaluation maps these inputs to a healthy margin class; the policy layer issues `TARGET` for operators wanting actionable acquisitions.

Example 2 — WATCH
INPUT:
- Year: 2014
- Make/Model: Ford F-150
- Mileage: 130,000
- Condition cues: title-brand hint present, inconsistent seller disclosures, visible rust in photos
- Location: distant auction hub

OUTPUT:
- Decision: WATCH
- Cost stack summary: transport: high-band; fees: nominal; recon class: medium-repair
- Margin signal: marginal-margin-class
- Risk flags: disclosure-mismatch, elevated-historical-loss-class

Reasoning (human-readable):
- Normalized vehicle shows higher mileage and several condition concerns. Cost aggregation yields a larger transport and recon impact. Risk layer surfaces disclosure mismatch and elevated historical loss entries for similar vehicle configurations. The deterministic evaluation produces a marginal margin class and flags risk; the policy maps this to `WATCH` suggesting operator monitoring or manual review rather than immediate bidding.

Example 3 — PASS
INPUT:
- Year: 2008
- Make/Model: Luxury Sedan (ambiguous model)
- Mileage: 220,000
- Condition cues: heavy visible damage, title-brand confirmed, non-drivable note
- Location: local auction

OUTPUT:
- Decision: PASS
- Cost stack summary: transport: medium; fees: nominal; recon class: heavy-repair
- Margin signal: negative-margin-class
- Risk flags: heavy-damage, title-brand, non-drivable

Reasoning (human-readable):
- Normalized record exposes multiple high-severity condition cues (heavy damage, title-brand, non-drivable). Aggregated cost categories and the evaluation artifact indicate a negative margin class. Risk layer confirms multiple high-severity flags. Policy deterministically maps these inputs to `PASS` to avoid operator exposure to high loss risk.

## 7. Constraints & Tradeoffs

- Determinism vs probabilistic modeling: determinism favors reproducibility, auditability, and operator trust at the expense of probabilistic flexibility. For a decision-support tool operating under time pressure, consistent repeatable outputs reduce cognitive load and enable easier post-mortem analysis.
- Flexibility vs safety: restricting AI and heuristic influence reduces the risk of opaque behavior; it requires more explicit domain encoding in normalization and policy layers.

## 8. Real-World Impact

- Faster decisions: operators can process more listings per session with consistent labels.
- Reduced risk: consistent policy and explicit risk flags lower incidence of unexpected losses.
- Improved consistency: reproducible outputs support objective comparisons across operators and time.
- Scalability: clear boundaries allow batch or list-mode evaluation at scale without modifying the deterministic core.

## 9. What Is Intentionally Omitted

- No source code, no numeric formulas, no parameter thresholds, and no internal heuristics are published here. That is intentional: the private repository contains implementation details and IP-sensitive logic.

## 10. Audience Guidance

- Recruiters / engineering managers: start with the one-page diagram for a 2-minute overview, then read the deep-dive to understand system boundaries, determinism guarantees, and trade-offs.
- Look for: clear data flows, separation of concerns, auditability, and reproducible decision outputs.

---

This public document is intentionally concise and focused on architecture and system behavior while preserving all implementation IP.
