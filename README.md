# Compass: a Social Impact Planner for Turin

Where should a foundation, a third-sector organisation or a public body invest its welfare budget in Turin? Compass crosses 11 open datasets from the City of Turin and the Piedmont Region over the 92 statistical zones of the city, builds three need indices (demographic isolation, lack of services, socio-economic fragility) and matches them with the profile of the organisation. The result is a map of need, a plan that says where to invest, with which intervention, for how many people and why, and a chatbot that answers questions about the plan citing only the data. In the demo, a foundation with 150,000 € gets five zones, 360 direct beneficiaries and an average robustness of 93% over 200 simulations.

Built for the Campionato Universitario AI 2026, Turin stage, 4 November 2026, track 3 Welfare.

**Authors:** Francesco Screti, Francesco Rolando & Francesco Lazzarotto (team Attention Is All We Need)

---

## Project Structure

```
AI2B-Turin/
├── README.md
├── requirements.txt
├── docs/
│   └── METODO.md            # method, data limits and API contract (Italian)
├── data/
│   ├── raw/                 # downloaded open data (not versioned)
│   └── processed/           # zones.csv, zones.geojson, drivers.json (versioned)
├── src/
│   ├── build.py             # territorial indicators, PCA weights, k-means profiles
│   ├── api.py               # organisation profile, matching, planner, chat, FastAPI
│   └── catalog.json         # interventions, unit costs, indicator labels
├── serve/
│   ├── start_llm.sh         # LLM and embedding models on vLLM
│   ├── start_api.sh         # API and front end on port 8700
│   └── qwen38_nothink.jinja
├── web/                     # front end (HTML, CSS, JavaScript, MapLibre, D3)
└── presentation/            # pitch deck with the recorded demo
```

---

## How it works

The territorial indicators are computed without knowing the organisation, which enters only at the matching step.

1. **Territorial intelligence** (`src/build.py`). For each zone the pipeline computes indicators of share of people over 65 and over 80, ageing index and its change from 2013 to 2020, widows and people living alone (estimated from district data), economic assistance per 1,000 residents, share of foreign residents, and the average distance of the zone's house numbers from the nearest health facility, residential facility and pharmacy. A principal component analysis sets the weight of each factor inside its index, and k-means groups the zones into five profiles.
2. **Organisation profile.** The organisation fills in a form or describes itself in a few lines. The language model turns the text into a structured profile, constrained to the categories and interventions of `src/catalog.json`.
3. **Matching** (`POST /api/match`). Every zone and intervention pair gets a score from four parts: need fit, service gap, mission fit (semantic similarity between the stated mission and the intervention, with multilingual-e5 embeddings) and feasibility (how well the scale of the zone fits the organisation's budget and staff).
4. **Planner** (`POST /api/plan`). The top zones get the best intervention, the expected beneficiaries, the budget share and a robustness index: the share of 200 simulations with perturbed weights in which the zone stays in the plan.

The ranking is decided by a transparent formula, not by the language model. The model writes the plan text and the chat answers, and every figure it writes is checked against the data: if a number is not there, the text is discarded and replaced by a template.

| Component | Method | Role |
|---|---|---|
| Language model | Qwen3.8-27B INT4 served with vLLM | reads the organisation's text, writes the plan, answers questions |
| Embeddings | multilingual-e5-base | mission fit between the organisation and each intervention |
| Unsupervised learning | PCA and k-means (scikit-learn) | factor weights and zone profiles |

---

## Data

| Source | Datasets |
|---|---|
| City of Turin, AperTO | population and foreign residents by zone; age by zone, 2013 and 2020; marital status and households by district, 2025; economic assistance; house numbers with coordinates |
| City of Turin, geoportal | boundaries of the statistical zones |
| Piedmont Region | ASL health facilities; pharmacies |

Raw files are not versioned, while the processed outputs in `data/processed/` are, so the API runs out of the box. Known limits (estimated values for people living alone and widows, distortion of assistance data in central zones, 219 of 251 health facilities located) are described in `docs/METODO.md`.

---

## Setup

### Dependencies

```bash
python3 -m venv .venv && .venv/bin/pip install -r requirements.txt
```

### Data (optional)

Rebuild the indicators after updating `data/raw/`:

```bash
.venv/bin/python src/build.py
```

### Models and API

```bash
bash serve/start_llm.sh    # Qwen3.8-27B on :8001 and multilingual-e5 on :8002 (GPU)
bash serve/start_api.sh    # API and front end on http://127.0.0.1:8700
```

The language model takes about twenty minutes to load. Without it the system still works: map, ranking, plan and robustness do not depend on the model, the plan text becomes a template and the chat falls back to fixed rules.

### Front end

Open http://127.0.0.1:8700. The interface has four screens: set up the organisation (or "Compila con l'AI" from free text), map of need, plan, and "Chiedi al piano". A self-running walkthrough is available at http://127.0.0.1:8700/?demo=1.

### Presentation

Open `presentation/index.html` in any browser, even offline: fonts, images and the recorded demo are included. Arrow keys move between slides, F enters full-screen presentation mode.

---

## AI Act

Compass works on aggregate data by statistical zone, does not assess individuals and does not decide access to public assistance, so it does not fall among the high-risk systems of Annex III, point 5 of Regulation (EU) 2024/1689. The chat states that answers are generated by an AI system and may contain errors, in line with the transparency obligations of Article 50.
