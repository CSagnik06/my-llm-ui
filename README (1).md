# 🐇 Talking Rabbitt — AI Business Intelligence Platform

Conversational AI + interactive analytics over **your own data**. Upload a CSV
or Excel file and the platform ingests it, infers the schema, computes real KPIs,
auto-selects charts, forecasts future performance, generates recommendations, and
answers natural-language questions — all from the uploaded dataset. **No hard-coded
metrics.**

> The UI is the finalized "Talking Rabbitt" prototype, wired end-to-end to a
> FastAPI backend that does genuine data science with pandas / NumPy / scikit-learn.

---

## ✨ What actually works

| Capability | How it's implemented |
|---|---|
| **File upload** | Real drag-and-drop + picker → `POST /api/upload`, streamed progress, pandas parse |
| **Ingestion agent** | Schema inference, type coercion (currency/date), dedupe, missing-value handling, **semantic role detection** (date/revenue/cost/qty/region/product/customer) |
| **Analytics agent** | Revenue, profit, ROI, margins, growth, top products/customers, regional split, outliers (IQR), correlation matrix |
| **Dynamic dashboard** | KPI cards + Chart.js charts generated from the data; chart type auto-selected by column roles |
| **Chat agent** | Dataset-aware Q&A. Uses OpenAI/Gemini if a key is set; otherwise a built-in rule-based analyst answers from real pandas computations |
| **Forecast agent** | Time-series forecast with 95% confidence interval. Prophet if installed, else a robust trend + seasonality model |
| **Recommendation agent** | Data-driven actions with confidence / impact / reasoning |
| **Reports** | Executive **PDF** (ReportLab), **Excel** (XlsxWriter), **CSV** exports |
| **Voice agent** | Web Speech API — ask by voice, hear the answer |
| **History** | Uploads, chats, forecasts, reports tracked |
| **Settings** | Persisted to `localStorage` |

The chat works with **zero API keys**. Add a key to upgrade to full LLM answers.

---

## 🚀 Quick start

### 1. Backend (FastAPI)
```bash
cd backend
python -m venv .venv && source .venv/bin/activate     # Windows: .venv\Scripts\activate
pip install -r requirements.txt
cp .env.example .env          # optional: add OPENAI_API_KEY or GEMINI_API_KEY
uvicorn main:app --reload     # → http://localhost:8000  (docs at /docs)
```

### 2. Frontend (Vite)
```bash
cd frontend
npm install
npm run dev                   # → http://localhost:5173
```
Vite proxies `/api` to the backend automatically. Open the app, go to
**Upload Dataset**, and drop `samples/sample_sales.csv`.

### 3. Or run everything with Docker
```bash
cp backend/.env.example backend/.env
docker compose up --build     # frontend :5173  ·  backend :8000
```

---

## 🔌 API

Interactive docs at `http://localhost:8000/docs`.

| Method | Endpoint | Purpose |
|---|---|---|
| `GET`  | `/api/health` | status + active LLM provider |
| `POST` | `/api/upload` | upload CSV/Excel → dataset + metadata |
| `GET`  | `/api/datasets` | list datasets |
| `GET`  | `/api/dataset/{id}` | paginated rows + schema |
| `GET`  | `/api/dashboard` | KPI cards + charts |
| `GET`  | `/api/analytics` | findings (top products, regions, outliers, correlation) |
| `POST` | `/api/chat` | dataset-aware NL answer (+ table/chart) |
| `GET`  | `/api/forecast?periods=6` | forecast + confidence interval |
| `GET`  | `/api/recommendations` | ranked recommendations |
| `GET`  | `/api/reports/{pdf\|excel\|csv}` | download a report |
| `GET`  | `/api/history` | activity log |
| `POST` | `/api/database/connect` | run a SQL query as a dataset (needs SQLAlchemy) |

Endpoints default to the most recently uploaded dataset; pass `?dataset_id=` to target a specific one.

---

## 🗂️ Structure

```
talking-rabbitt/
├── backend/
│   ├── main.py                 # FastAPI app + router registration
│   ├── api/                    # one router per resource (upload, chat, forecast …)
│   ├── agents/                 # ingestion · analytics · chat · forecast · recommendation · report
│   ├── core/                   # config · dataset store · numpy→json serializer
│   ├── storage/                # dataset cache (gitignored)
│   └── requirements.txt
├── frontend/
│   ├── index.html              # the runnable app (React via ESM + htm — UI unchanged)
│   ├── vite.config.js          # dev server + /api proxy
│   └── src/
│       ├── services/api.js     # axios client (modular reference)
│       ├── contexts/DataContext.jsx
│       ├── hooks/useVoice.js
│       └── utils/format.js
├── samples/sample_sales.csv    # 4,000-row demo dataset
├── docs/ARCHITECTURE.md
├── docker-compose.yml
└── README.md
```

### Why the frontend is a single `index.html`
The original prototype uses **React + htm** (JSX-free) loaded via ESM, so it needs
**no build step** and the UI renders exactly as designed. Vite serves it and proxies
the API. The `src/` folder mirrors the same logic as clean ES modules for teams that
prefer a JSX/TSX build — the inline `BIProvider` and the exported `DataContext` share
one contract.

---

## 🔐 Environment variables

| Var | Where | Default | Notes |
|---|---|---|---|
| `OPENAI_API_KEY` | backend | – | enables OpenAI chat |
| `GEMINI_API_KEY` | backend | – | enables Gemini chat |
| `MAX_UPLOAD_MB`  | backend | 100 | upload size cap |
| `CORS_ORIGINS`   | backend | localhost:5173,3000 | comma-separated |
| `VITE_API_TARGET`| frontend | http://localhost:8000 | proxy target |

## 🧪 Optional dependencies
Uncomment in `backend/requirements.txt` to enable:
`prophet` (ML forecasting), `sqlalchemy` + `psycopg2-binary` / `pymysql`
(Postgres/MySQL connectors). Everything degrades gracefully when they're absent.

## 📄 License
MIT — see [LICENSE](LICENSE).
