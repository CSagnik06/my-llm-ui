# Architecture

```
┌──────────────────────────────┐        HTTP /api        ┌────────────────────────────────┐
│  Frontend (Vite :5173)       │  ───────────────────▶   │  Backend — FastAPI (:8000)     │
│                              │                          │                                │
│  index.html                  │                          │  main.py                       │
│   • React (ESM) + htm        │                          │   └─ api/ routers              │
│   • BIProvider (global state)│   ◀───────────────────   │        upload · analytics ·    │
│   • Chart.js visuals         │        JSON              │        chat · forecast ·       │
│   • Web Speech (voice)       │                          │        recommendations ·       │
│                              │                          │        reports · history · db  │
│  src/ (modular reference)    │                          │                                │
│   services/api.js (axios)    │                          │  agents/                       │
│   contexts/DataContext.jsx   │                          │   ingestion → analytics →      │
│   hooks/useVoice.js          │                          │   chat / forecast / recommend  │
└──────────────────────────────┘                          │   / report                     │
                                                           │                                │
                                                           │  core/  config · store ·       │
                                                           │         serialize              │
                                                           │  storage/ (pickle cache)       │
                                                           └────────────────────────────────┘
```

## Request lifecycle (upload → insight)
1. **Upload** — file streamed to `POST /api/upload`.
2. **Ingestion agent** — parse (pandas) → normalize columns → coerce types
   (currency/date) → drop duplicates → impute missing → **detect each column's
   semantic role**. Returns cleaned frame + metadata; frame cached to `storage/`.
3. **Analytics agent** — from roles: KPIs, chart specs (Chart.js-ready), findings
   (top products/customers, regional split, IQR outliers, correlation).
4. **Forecast agent** — revenue resampled monthly → Prophet (if installed) or a
   trend + seasonal model → point forecast + 95% CI.
5. **Recommendation agent** — heuristics over computed stats → actions with
   confidence / impact / reasoning.
6. **Chat agent** — builds a compact factual brief; an LLM (OpenAI/Gemini) reasons
   over it, or a deterministic rule-based analyst routes keywords to real pandas
   computations when no key is configured.
7. **Report agent** — PDF / Excel / CSV from the same computed results.

## Design notes
- **No hard-coded business values.** Every KPI, chart, forecast and recommendation
  is derived from the uploaded frame. UI defaults remain only as pre-upload
  placeholders so the prototype looks identical before data is loaded.
- **Graceful degradation.** Missing LLM key → rule-based analyst. Missing Prophet →
  statistical forecaster. Missing SQL drivers → connectors report cleanly.
- **`core/serialize.to_native`** converts numpy/pandas scalars so every payload is
  valid JSON.
- **State** is in-memory with a pickle mirror in `storage/` for restart recovery.
