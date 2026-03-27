Engineer Handoff — AO1 (Internal, IP-Safe)

This handoff is written for engineers who will work on AO1. It explains system behavior, operational constraints, data flows, invariants, extension points, and failure modes without revealing implementation code, numeric formulas, or private heuristics.

The intent is practical: enable an incoming engineer to reason about where to make safe changes, how to validate behavior, and how to extend the system while preserving determinism and auditability.

## 1. System Overview (Engineer Perspective)

AO1 is a deterministic decision-support system for vehicle-acquisition workflows. It operates on live auction pages and produces reproducible, auditable acquisition recommendations (labels: `TARGET`, `WATCH`, `PASS`). The system is built as a pipeline with explicit boundaries: Input ingestion → Normalization → Cost Stack aggregation → Deterministic Evaluation → Risk Augmentation → Decision Policy.

Operationally, AO1 runs at the point of inspection. A content-side runtime extracts listing data, the pipeline normalizes and evaluates the listing, and an in-page HUD surfaces the decision and readable rationale. All transient and persisted evaluation artifacts carry provenance metadata so every output can be replayed and audited.

Key operational expectations:
- Low-latency per-listing evaluation suitable for human-in-the-loop decisioning.
- Persisted evaluation artifacts for audit and post-mortem analysis.
- Clear separation between deterministic outputs (authoritative) and advisory signals (informational).

## 2. Core Invariants

- Deterministic outputs: identical, normalized inputs plus strategy/profile and persisted memory always produce the same evaluation and label.
- No stochastic behavior: the system does not use randomness in producing the final decision label.
- No hidden state: all inputs that materially affect decisions are either provided explicitly, derivable from persisted artifacts, or recorded in the provenance metadata.
- Reproducibility: any evaluation can be re-run from persisted inputs to verify and audit the output.

These invariants are essential to operator trust and post-hoc incident analysis. Any change that risks these invariants must be accompanied by an explicit migration/review plan.

## 3. Data Flow (Stage-by-Stage)

The pipeline stages below describe what enters, what leaves, and what each stage enforces. Keep implementation details private; the descriptions below explain observable behavior and boundaries.

### 3.1 Input Ingestion
- What enters: raw, host-specific listing signals scraped from page DOM and route state (example: title text, visible damage cues, seller tags, VIN hints, mileage hints, images, and timestamped provenance).
- What leaves: an adapter payload (a conservative, typed carrier of raw fields and extraction provenance).
- What is enforced: extraction must record provenance (host, selector, timestamp) and flag unverifiable fields. Adapters must tolerate DOM drift and fail gracefully (emit an extraction error artifact, not silent failure).

### 3.2 Normalization
- What enters: adapter payload (heterogeneous, host-specific fields).
- What leaves: a normalized `Vehicle` contract — identity, year/make/model, mileage estimate, condition cues, disclosures, location band, and image references — plus provenance.
- What is enforced: normalization preserves provenance and flags unverifiable claims. Normalization must not invent values; unverifiable or ambiguous fields are represented as explicit flags.

### 3.3 Cost Stack (Aggregation)
- What enters: normalized Vehicle + session context (strategy/profile, operator location band).
- What leaves: cost summary object describing aggregated cost categories (transport band, recon class, fee category, hard-cost summary) — note: public docs refer to categorical summaries only, not numeric formulas.
- What is enforced: cost categories are conservative and auditable; each aggregated category records its contributing inputs.

### 3.4 Evaluation (Deterministic Core)
- What enters: normalized Vehicle, cost summary, persisted memory (case/loss snapshots), strategy/profile.
- What leaves: evaluation artifact (exit-class, margin-class, headroom-class, reasons[]), each with provenance.
- What is enforced: deterministic mapping from inputs to evaluation artifact; no stochastic or hidden inputs influence the artifact.

### 3.5 Risk Classification
- What enters: evaluation artifact, Vehicle, loss-memory snapshots.
- What leaves: riskAssessment (riskLevelClass, explicit riskFlags[]). Risk outputs are categorical flags for operator interpretation.
- What is enforced: risk flags are descriptive and traceable to observed inputs or persisted historical entries.

### 3.6 Decision Policy
- What enters: evaluation artifact + riskAssessment + strategy/profile.
- What leaves: final decision {label: TARGET|WATCH|PASS, rationale[], recommended next action}.
- What is enforced: policy deterministically maps inputs to labels. Advisory data (e.g., AI commentary) is not used to change the label.

## 4. Decision Ownership Model

- The deterministic evaluation + policy layers own the final decision label. This ownership is deliberate: it ensures repeatability, simplifies auditing, and constrains operational risk.
- AI or external advisory signals are strictly bound: they may annotate the HUD with explanations, classify non-critical fields, or suggest areas for operator attention, but they do not alter authoritative outputs.
- Why this constraint exists: separating advisory signals from authoritative outputs minimizes opaque system behavior, preserves operator trust, and simplifies incident review when decisions are questioned.

## 5. Extension Points (Where to Safely Change the System)

Engineers should use these explicit extension points to add capabilities while preserving invariants:

- Adapter layer (ingestion): add new host adapters to support additional auction platforms. Ensure adapters always record provenance and emit flagged fields for unverifiable data.
- Normalization rules: add mapping for new fields or improved heuristics to better extract canonical attributes. Changes must preserve provenance and add explicit flags for ambiguity.
- Cost stack categories: extend categories or add new aggregated buckets; ensure every category cites its contributing inputs and stays non-proprietary in public summaries.
- Evaluation artifacts: add new descriptive classes (e.g., additional headroom classes) but do not embed stochastic elements. Any change to mapping logic must be versioned and accompanied by replay tests.
- Risk classification: add new risk flags or sources; map them into the riskAssessment structure and document provenance and trigger conditions at a conceptual level.
- Persistence interfaces: add new persisted artifacts (e.g., new case metadata fields) while preserving replayability of past evaluations.

For any change that can affect outputs, include: (a) replayable test vectors, (b) migration plan for persisted artifacts, and (c) explicit audit documentation describing the rationale and the expected behavioral delta.

## 6. Failure Modes and Stability Measures

Common failure classes and how the system is designed to remain stable:

- Incomplete data (missing VIN, ambiguous mileage): ingestion emits a flagged artifact; normalization preserves uncertainty; evaluation proceeds with conservative categories and flags the output for manual review.
- Conflicting signals (e.g., seller claim contradicting observed photos): normalization records both values with provenance; risk layer emits a disclosure-mismatch flag; policy can surface `WATCH` or `PASS` depending on aggregated severity.
- Edge-case vehicles (rare makes, ambiguous model references): the system falls back to conservative normalization (explicitly flagged ambiguity); evaluation uses broader categories and typically reduces headroom classification to avoid false positives.
- Adapter failures / DOM drift: adapters must fail fast and produce extraction error artifacts; the UI should display a clear degraded-state indicator rather than silently showing stale evaluations.

Stability measures:

- Conservative defaults and explicit flags for unverifiable inputs.
- Persisted provenance for every artifact for post-mortem replay.
- Clear separation between advisory overlays and authoritative labels to avoid accidental overrides.

## 7. Operational Context

AO1 operates in high-latency-sensitive, high-noise auction environments. Key operational constraints that shape architecture:

- Time pressure: operators need decisions within seconds; evaluate design and algorithmic choices prioritize predictable, low-latency paths.
- Noisy input sources: auction pages are heterogeneous and frequently change; adapters and normalization must be resilient and observable.
- Human-in-the-loop decisions: system outputs are recommendations; operators retain final responsibility and must be shown readable rationale and provenance.

Operational guidance for engineers:

- Prefer conservative behavior on ambiguous inputs.
- Ensure all operator-facing messages include provenance and an explanation concise enough for a quick judgment.
- Instrument any change with telemetry focused on decision drift and human overrides.

## 8. What Is Intentionally Omitted

This handoff intentionally omits:
- All source code and code snippets.
- Numeric formulas, thresholds, and internal scoring mechanics.
- Implementation-specific heuristics and production configuration.

Those elements are proprietary and kept in the private implementation repository. This document is intended to be sufficient for an engineer to reason about behavior and safe extension surfaces, while protecting IP.

---

If you are taking ownership of a component, your immediate tasks should be:
1. Run replay tests for a representative set of persisted artifacts to confirm current evaluation artifacts match expected labels.
2. Add unit-level replay vectors for any changes that touch normalization, evaluation, or policy.
3. Document any schema changes to persisted artifacts and provide migration steps and backward-compatibility notes.

For questions about private implementation details, contact the repository owner and request access to the private codebase under appropriate NDAs and access controls.