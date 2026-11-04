# Compass: a Social Impact Planner for Turin

Allocating welfare funding across urban areas requires fine-grained evidence on need, yet in Turin the relevant open data are fragmented across municipal and regional sources and rarely combined below the district level. We present Compass, a decision-support system that recommends where a funding organisation should invest, with which intervention and for how many beneficiaries. Compass integrates 11 open datasets over the 92 statistical zones of the city into three composite indices of demographic isolation, service shortage and socio-economic fragility, whose weights are derived by principal component analysis, and matches them with the organisation's profile through a transparent scoring function. A language model parses the organisation's self-description and explains the resulting plan, and every figure it generates is verified against the underlying data. In a demonstration scenario, a foundation with a budget of 150,000 € is assigned five zones and 360 direct beneficiaries, with an average robustness of 93% across 200 perturbations of the weights. Whether data-driven weights would agree with those elicited from domain experts remains an open question for future work.

Built for the Campionato Universitario AI 2026, Turin stage, 4 November 2026, track 3 Welfare.

**Authors:** Francesco Screti, Francesco Rolando & Francesco Lazzarotto

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

| Step | What happens |
|---|---|
| 1. Zones | `src/build.py` computes need indicators for the 92 zones: ageing, widows, economic assistance, foreign residents, distance from health services. PCA weights them into three indices, k-means groups the zones into five profiles. |
| 2. Organisation | The organisation fills in a form or describes itself in a few lines; the language model turns the text into a profile. |
| 3. Matching | Each zone gets a score from need, service gap, mission fit and feasibility. |
| 4. Plan | The top zones get an intervention, beneficiaries, budget and a robustness index over 200 simulations. |

A transparent formula decides the ranking; the language model only writes the text, and every figure it writes is checked against the data.

| Component | Method |
|---|---|
| Language model | Qwen3.8-27B INT4 on vLLM |
| Embeddings | multilingual-e5-base |
| Unsupervised learning | PCA and k-means (scikit-learn) |

Details of the method and of the API are in `docs/METODO.md`.

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

## Future work: experts or data?

The weights of the three need indices come from the data: a principal component analysis decides how much each factor counts. The open research question is how much these data-driven indices would disagree with indices built by people who work in the field, such as social workers, health district staff and third-sector operators.

A study to answer it would:

1. ask a panel of experts to weight the same factors, for example by pairwise comparison or by splitting 100 points among them;
2. build the expert indices on the same 92 zones;
3. compare the two rankings with a rank correlation and with the overlap of the zones that enter the plan;
4. look at the zones where data and experts disagree most, which are the most useful cases to discuss with the people who know the territory.

Two further steps would complete it: an evaluation of the interface with foundations and organisations (understanding of the plan, trust in the choices, usability), and data that are missing today, such as older people living alone by zone and the map of social care services.

---

## AI Act

Compass works on aggregate data by statistical zone, does not assess individuals and does not decide access to public assistance, so it does not fall among the high-risk systems of Annex III, point 5 of Regulation (EU) 2024/1689. The chat states that answers are generated by an AI system and may contain errors, in line with the transparency obligations of Article 50.
