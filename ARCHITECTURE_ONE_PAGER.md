# AO1 — One-Page System Diagram (Public)

This one-page summary is tuned for recruiters and engineering managers: a compact system sketch, runtime boundaries, and the deterministic evaluation path.

```mermaid
flowchart LR
  A["Auction page (DOM) & route state"]
  B["Platform Adapters (isolate host-specific DOM parsing)"]
  C["Normalized Vehicle Model"]
  D["Deterministic Evaluation Engine"]
  E["Decision Policy Layer (labels: TARGET / WATCH / PASS)"]
  F["Advisory Sidecar (market signals, optional) — advisory only"]
  G["UI & Operator Surfaces (in-page HUD, popup)"]
  H["Local Persistence (settings, cases, loss history)"]

  A --> B --> C --> D --> E --> G
  F --> G
  D --> H
  G --> H
```

At a glance
- Input: live auction page content and navigation state.
- Ingestion: adapters extract and normalize platform-specific data into a stable vehicle model.
- Deterministic core: an economic+risk evaluation path computes non-proprietary outputs used by a policy layer to surface operator-facing labels.
- Advisory sidecar: optional market or model-based signals provide operator-facing context but do not replace deterministic outputs in this public description.
- Interface: results surfaced in an in-page HUD and a small operator popup for case review.

This page is illustrative — full runtime behavior and implementation details are private.
