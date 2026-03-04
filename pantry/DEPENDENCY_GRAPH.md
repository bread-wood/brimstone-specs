# pantry — Dependency Graph

```mermaid
graph TD
  v010["pantry/v0.1.0-meal-planner
  ────────────────────
  LLM meal suggestions
  Recipe API + fallback
  FastAPI web UI
  Email delivery
  launchd scheduler
  SQLite"]

  v020["pantry/v0.2.0-shopping-cart
  ────────────────────
  Ingredient aggregation
  Delivery API checkout link
  Web UI cart view
  Email: list + link"]

  v010 -->|confirmed plan with ingredients| v020

  ext_llm["breadmin-llm
  (sibling dep, read-only)"]
  ext_shared["breadmin-shared
  (sibling dep, read-only)"]

  ext_llm -.->|LLM calls| v010
  ext_shared -.->|utilities| v010
  ext_llm -.->|LLM calls| v020
  ext_shared -.->|utilities| v020
```

## Notes

- pantry is fully standalone — no dependency on moot, breadmin-jobman, or breadwinner
- `breadmin-llm` and `breadmin-shared` are consumed as read-only sibling deps; no changes required to those repos
- v0.2.0 is blocked on the P1 Key Unknown in v0.1.0: grocery delivery API access (Instacart Connect or alternative) must be resolved before v0.2.0 design can begin
