# CircuitMind

AI-powered electronics assistant for **Digital Logic Studio**. It turns a short English prompt into structured circuit JSON, then explains, diagnoses, and exports that circuit.

This is **not** a custom-trained GNN. Generation uses **Groq (Llama)** first, with a **rule-based fallback** if the API key is missing or the LLM fails.

---

## What it does

| Module | What you get |
|--------|----------------|
| **Generate** | Prompt → circuit JSON (`components`, `connections`) |
| **Explain** | Plain-English how-it-works + component roles + warnings |
| **Diagnose** | Electrical checks (no power, LED without resistor, short, floating parts) |
| **Export** | `spice`, `svg`, or `gate_json` |

Optional extras:

- **Streamlit UI** (`app_streamlit.py`) — local playground
- **CV module** (`cv_module/`) — image → circuit JSON (YOLO; extra deps, not required for the API)

---

## How it fits Digital Logic Studio

Boolforge / Circuit Forge talks to this service via `REACT_APP_CIRCUITMIND_API_URL`:

| Environment | Default URL |
|-------------|-------------|
| Local | `http://localhost:8000` |
| Production | `https://circuit-mind-two.vercel.app` |

Hints from Circuit Forge currently POST to CircuitMind `/hint`. The FastAPI app in this repo exposes **generate / explain / diagnose / export** — not `/hint`. Circuit-generation and grading hints for DLD problems also exist on the **main backend** at `/api/ai/hint` and `/api/ai/generate-circuit`.

To use CircuitMind with the frontend:

1. Run this API on port **8000**
2. Keep `REACT_APP_CIRCUITMIND_API_URL=http://localhost:8000` in `frontend/.env` (own line, no inline `#` comments)

---

## Requirements

- Python **3.10+** (Docker image uses 3.11)
- A Groq key for smart generation ([console.groq.com](https://console.groq.com)) — optional; fallback rules still work

---

## Project structure

```text
CircuitMind/
├── api/app.py                 # FastAPI server
├── generate/generate.py       # Prompt → circuit JSON (Groq + rules)
├── explain/explain_module.py  # Circuit JSON → explanation
├── diagnose/diagnose_module.py
├── export/export_module.py    # spice | svg | gate_json
├── utils/component_resolver.py
├── app_streamlit.py           # Optional local UI
├── tests/test_all_modules.py
├── cv_module/                 # Optional image pipeline
├── docker-compose.yml
├── Dockerfile                 # API listens on 7860 inside the container
├── requirements.txt
└── .env.example
```

---

## Local setup

```powershell
cd D:\QuantumLogics\DigitalLogicStudio\CircuitMind

python -m venv .venv
.\.venv\Scripts\Activate.ps1

pip install -r requirements.txt
```

Copy `.env.example` to `.env` and set:

```env
GROQ_API_KEY=your_groq_api_key_here

# Optional — defaults in code are Streamlit origins only
ALLOWED_ORIGINS=http://localhost:3000,http://localhost:8501

# Optional — if set, clients must send header X-API-Key
# CIRCUITMIND_API_KEY=your_shared_secret
```

Start the API:

```powershell
uvicorn api.app:app --reload --host 0.0.0.0 --port 8000
```

- Docs: [http://localhost:8000/docs](http://localhost:8000/docs)
- Health: [http://localhost:8000/health](http://localhost:8000/health)

Optional Streamlit UI (second terminal, venv on):

```powershell
streamlit run app_streamlit.py
```

UI: [http://localhost:8501](http://localhost:8501)

---

## API

Rate limits apply per IP (e.g. generate `5/minute`). If `CIRCUITMIND_API_KEY` is set, send `X-API-Key`.

| Method | Path | Body |
|--------|------|------|
| GET | `/` | — |
| GET | `/health` | — |
| POST | `/generate` | `{ "prompt": "make me a LED circuit" }` |
| POST | `/explain` | `{ "circuit_json": { ... } }` |
| POST | `/diagnose` | `{ "circuit_json": { ... } }` |
| POST | `/export` | `{ "circuit_json": { ... }, "export_format": "spice" }` |
| POST | `/generate-and-explain` | `{ "prompt": "555 timer circuit" }` |

`export_format` must be one of: `spice`, `svg`, `gate_json`.

Example generate:

```json
POST /generate
{ "prompt": "make me a LED circuit" }
```

```json
{
  "circuit_name": "LED Circuit",
  "components": ["battery", "resistor", "led"],
  "connections": ["battery -> resistor -> led"],
  "confidence": "high",
  "source": "llm"
}
```

Rule-based fallback keywords include: LED, motor, buzzer, fan, temperature sensor, solar, 555 timer, RC filter.

---

## Docker

```powershell
cd D:\QuantumLogics\DigitalLogicStudio\CircuitMind
docker compose up --build
```

| Service | Host URL |
|---------|----------|
| FastAPI | http://localhost:8000 (container port 7860) |
| Streamlit | http://localhost:8501 |

---

## Tests

From the CircuitMind root (venv on):

```powershell
pip install pytest
pytest tests/
```

Module-level scripts still exist:

```powershell
python generate/test_manual.py
python explain/test_cases.py
python diagnose/test_cases.py
python export/testcases.py
```

---

## Circuit JSON shape

Shared by generate, explain, diagnose, and export:

```json
{
  "circuit_name": "LED Circuit",
  "components": ["battery", "resistor", "led"],
  "connections": ["battery -> resistor -> led"]
}
```

Component names use underscores (`op_amp`, `power_supply`, `555_timer`). Connections accept `->` or `--`.

---

## Optional CV module

`cv_module/` is a separate image pipeline (dataset prep, YOLO, topology extraction). It is **not** mounted on the FastAPI app. Extra packages are listed in `cv_module/requirements_cv.txt` and commented out in the root `requirements.txt`. See `cv_module/README.SHAYAN.md`.
