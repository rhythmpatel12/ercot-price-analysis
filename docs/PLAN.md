# Project Plan: ERCOT Price Analysis

> **The question:** What drives Texas electricity prices, and can we predict them?
>
> **The product:** An interactive dashboard that explains how ERCOT prices behave and forecasts next-day prices.

| | |
|---|---|
| **Purpose** | Portfolio project covering data analysis and data science |
| **Deliverable** | Interactive Streamlit dashboard, hosted online |
| **Timeline** | 3–4 weeks |
| **How we work** | Step by step. I write the code; Claude explains, hints, and reviews |

---

## 1. Why this project

ERCOT runs the Texas power grid. It is an *energy-only* market: there are no separate payments for keeping capacity available, so prices alone have to signal shortages. As a result prices swing from below $0 to $5,000/MWh, and the data has real stories in it:

- **Winter Storm Uri (Feb 2021):** prices held at the $9,000 cap for days during blackouts
- **Summer 2023:** record demand and evening price spikes
- **Solar and battery growth:** cheap midday power and expensive evenings
- **RTC+B (Dec 2025):** a major redesign of how the real-time market works

---

## 2. The story (dashboard chapters)

Each chapter is one dashboard page. Each needs one clear headline finding backed by a number.

| # | Chapter | Question it answers | Key visuals |
|---|---------|---------------------|-------------|
| 1 | **Overview** | What does a year of prices look like? | KPI tiles: average price, max, negative hours, spike hours |
| 2 | **Daily patterns** | When is power cheap or expensive, and how has that changed? | Hour × month heatmap; daily price shape by year (duck curve) |
| 3 | **Spikes & scarcity** | How much of the cost comes from a few extreme hours? | Price duration curve; Uri and Aug 2023 timelines |
| 4 | **Geography** | Why do prices differ across Texas? | Hub-to-hub spreads; negative prices at HB_WEST; map |
| 5 | **Day-ahead vs real-time** | Does the day-ahead price predict the real one? | DA–RT spread over time; before/after RTC+B |
| 6 | **Forecasting** | Can we predict tomorrow's prices? | Predicted vs actual; error by season; feature importance (SHAP) |
| 7 | *Stretch:* **Battery value** | What would a battery earn buying low and selling high? | Yearly revenue by hub |

---

## 3. The data

| Dataset | What it is | Granularity | Source |
|---------|-----------|-------------|--------|
| **Day-Ahead prices** | Prices set the day before delivery | Hourly | ERCOT report NP4-180-ER |
| **Real-Time prices** | Actual delivered prices | 15-minute | ERCOT report NP6-785-ER |
| **Load** | Electricity demand, actual and forecast | Hourly | ERCOT via `gridstatus` |
| **Wind & solar** | Generation, actual and forecast | Hourly | ERCOT via `gridstatus` |
| **Weather** | Temperature in Houston, Dallas, Austin, Midland | Hourly | Open-Meteo |

**Scope:** 2019 to present, trading hubs and load zones only:
- Hubs: `HB_NORTH`, `HB_SOUTH`, `HB_WEST`, `HB_HOUSTON`, `HB_PAN`, `HB_BUSAVG`
- Load zones: `LZ_*`

**The key idea:** `net load = load − wind − solar`. Net load is the demand that gas and coal plants must cover, and it is the strongest single driver of price.

---

## 4. Roadmap

Do one step at a time. After each step, add 1–3 sentences of findings to `notes/story.md`.

### Week 1: Learn the data
- [x] **Step 1. Set up the project.** `uv`, `pyproject.toml`, folder layout, `.gitignore`.
- [ ] **Step 2. Explore by hand.** Download the 2024 Day-Ahead hub prices, open them in a notebook, and find the average, max, negative hours, and cheapest hub.
- [ ] **Step 3. Real-time vs day-ahead.** Pull 15-minute real-time prices, convert to hourly (time zones, DST), and compare with day-ahead.
- [ ] **Step 4. Drivers.** Pull load, wind, and solar data, compute net load, and plot it against price.

### Week 2: Build the analysis
- [ ] **Step 5. Scale up.** Move notebook code into `src/ercot/ingest.py`, cover 2019 to present, store as Parquet and query with DuckDB, and add data tests.
- [ ] **Step 6. Analysis notebooks.** Patterns, spikes, geography, and DA vs RT, with one headline finding each.

### Week 3: Model
- [ ] **Step 7. Forecasting.** Start with a seasonal-naive baseline, then linear regression and LightGBM. Use a walk-forward backtest and explain predictions with SHAP.

### Week 4: Ship
- [ ] **Step 8. Dashboard.** One Streamlit page per chapter, deployed to Streamlit Community Cloud.
- [ ] **Step 9. Polish.** README with the pitch, screenshots, findings, and how to run it.

---

## 5. Tech stack

| Purpose | Tool |
|---------|------|
| Language & environment | Python 3.12, `uv` |
| Data download | `gridstatus`, ERCOT website, Open-Meteo |
| Storage & queries | Parquet + DuckDB |
| Analysis | pandas, Jupyter |
| Modeling | scikit-learn, LightGBM, SHAP |
| Dashboard | Streamlit + Plotly |
| Quality | pytest, ruff |

---

## 6. Repo layout (target)

```
ercot-price-analysis/
├── README.md            # pitch, screenshots, findings, how to run
├── pyproject.toml
├── docs/PLAN.md         # this file
├── notes/story.md       # findings collected step by step
├── notebooks/           # exploration: 01_eda, 02_spikes, 03_geography, 04_forecasting
├── src/ercot/
│   ├── ingest.py        # download raw data
│   ├── transform.py     # clean, align time, build features
│   ├── db.py            # DuckDB helpers
│   └── model/           # features, training, backtest
├── app/                 # Streamlit: Home.py + pages/
├── data/                # raw/ and processed/ (gitignored)
└── tests/
```

---

## 7. How we'll know it works

- **Data is correct:** no duplicate timestamps; daylight-saving days have 23 or 25 hours; row counts per hub per year are as expected.
- **Matches known history:** HB_HOUSTON real-time prices sit near $9,000 on Feb 15–18, 2021, and the cap is $5,000 from 2022 on.
- **The model earns its place:** it beats the seasonal-naive baseline on a held-out year, and results are reported honestly, including where it fails (spikes).
- **The dashboard runs:** `streamlit run app/Home.py` works locally, and the hosted link loads quickly.

---

## 8. Glossary

| Term | Meaning |
|------|---------|
| **ERCOT** | Electric Reliability Council of Texas, the grid and market operator |
| **$/MWh** | Price per megawatt-hour; one MWh powers roughly 300–500 Texas homes for an hour |
| **Settlement point** | A location where prices are set (node, hub, or load zone) |
| **Hub** | An average price across a region; the prices traders quote |
| **Load zone** | The price that utilities and retailers pay in a region |
| **DAM** | Day-Ahead Market: hourly prices for tomorrow |
| **RTM** | Real-Time Market: 15-minute prices at delivery |
| **Hour Ending (HE)** | ERCOT labels hours by their end: HE 01:00 means 00:00–01:00 |
| **Net load** | Load minus wind and solar; the demand other plants must cover |
| **Duck curve** | Cheap midday prices from solar, then a steep evening ramp |
| **Price cap** | The maximum allowed price: $9,000/MWh before 2022, $5,000 since |
| **RTC+B** | Real-Time Co-optimization plus Batteries, a market redesign launched Dec 2025 |
