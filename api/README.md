# CircuitMind API

**Developer:** Alia  
**Module:** REST API Server — connects all 4 CircuitMind modules

---

## What It Does

This FastAPI server connects all 4 modules (Generate, Explain, Diagnose, Export) into a single running service. Any frontend, tool, or script can send requests to this API and get circuit results back.

---

## Endpoints

| Method | Endpoint | Description |
|---|---|---|
| GET | `/` | Health check — confirms API is running |
| POST | `/generate` | Convert natural language prompt to circuit JSON |
| POST | `/explain` | Explain a circuit in plain English |
| POST | `/diagnose` | Check circuit for electrical issues |
| POST | `/export` | Export circuit to SPICE netlist or SVG |
| POST | `/hint` | Digital-logic circuit-builder hint (truth table + student's current gate graph → one non-spoiler hint) |
| POST | `/generate-and-explain` | Generate + explain + diagnose in one request |

---

## Setup

### 1. Install dependencies
```bash
pip install -r requirements.txt
```

### 2. Add API key
```bash
cp .env.example .env
```
Open `.env` and add your Groq API key. Get a free key at [console.groq.com](https://console.groq.com).

```
GROQ_API_KEY=your_key_here
```

### 3. Run the server
```bash
uvicorn api.app:app --reload
```

### 4. Open interactive docs
```
http://localhost:8000/docs
```

---

## Example Requests

### Generate a circuit
```json
POST /generate
{
  "prompt": "make me a LED circuit"
}
```

Response:
```json
{
  "circuit_name": "LED Circuit",
  "components": ["battery", "resistor", "led"],
  "connections": ["battery -> resistor -> led"],
  "confidence": "high",
  "source": "llm"
}
```

### Explain a circuit
```json
POST /explain
{
  "circuit_json": {
    "circuit_name": "LED Circuit",
    "components": ["battery", "resistor", "led"],
    "connections": ["battery -> resistor -> led"]
  }
}
```

### Diagnose a circuit
```json
POST /diagnose
{
  "circuit_json": {
    "circuit_name": "Bad Circuit",
    "components": ["battery", "led"],
    "connections": ["battery -> led"]
  }
}
```

### Export to SPICE
```json
POST /export
{
  "circuit_json": {
    "circuit_name": "LED Circuit",
    "components": ["battery", "resistor", "led"],
    "connections": ["battery -> resistor -> led"]
  },
  "export_format": "spice"
}
```

### All in one
```json
POST /generate-and-explain
{
  "prompt": "555 timer circuit"
}
```

### Get a hint for a digital-logic problem
```json
POST /hint
{
  "problem_title": "Half Adder",
  "problem_description": "Two 1-bit inputs A and B, outputs Sum (S) and Carry (C).",
  "inputs": ["A", "B"],
  "outputs": ["S", "C"],
  "truth_table": [
    {"A": 0, "B": 0, "S": 0, "C": 0},
    {"A": 0, "B": 1, "S": 1, "C": 0},
    {"A": 1, "B": 0, "S": 1, "C": 0},
    {"A": 1, "B": 1, "S": 0, "C": 1}
  ],
  "gates": [{"id": 1, "type": "INPUT", "label": "A"}, {"id": 2, "type": "INPUT", "label": "B"}],
  "wires": [],
  "last_result": null
}
```

Response:
```json
{
  "hint": "It looks like you've just started ... consider using a gate that produces an output of 1 when the two inputs are different.",
  "source": "llm"
}
```

`gates`/`wires` use the same shape as the logic-gate simulator's export format (`export_format=gate_json`): `{id, type, label, ...}` gates and `{fromId, toId, toIndex}` wires. `last_result` is optional context from the caller's own validator, e.g. `{"passed": false, "failing_rows": [...]}`.

---

## Files

| File | Purpose |
|---|---|
| `app.py` | Main FastAPI server code |
| `README.md` | This file |
