# Federal Research Funding to Higher Education (FY2021–2026)

An interactive dashboard mapping U.S. federal research obligations to higher-education
institutions, built on the live [USAspending.gov](https://www.usaspending.gov) API.
Funding is shown by **recipient location** (the institution's home state), filtered to
higher-education recipients, across six agencies: DoD, DOE, HHS, DHS, NSF, and NASA.

## Features

- **Per-capita choropleth map** of obligations by state, by agency and fiscal year.
- **Top university recipients** (national or drilled down to a single state).
- **Funding trend** (FY2021–2026) for the selected agency / state.
- **Mississippi vs national, per agency** — a six-panel per-capita comparison of
  Mississippi against the population-weighted national benchmark
  (total obligations to the 50 states + DC ÷ their combined population).

All figures come from the same USAspending obligations data, re-aggregated; there is no
offline dataset and no API key is required (USAspending is a public, open API).

## Run locally

```bash
python -m venv .venv
# Windows:  .venv\Scripts\activate
# macOS/Linux:  source .venv/bin/activate

pip install -r requirements.txt
streamlit run research_funding_dashboard.py
```

The app opens at http://localhost:8501. The first load of the
Mississippi-vs-national panels fetches 6 agencies × 6 years and is slow on a cold
cache (~30–60s), then is cached for an hour.

## Deploy on Streamlit Community Cloud

1. Push this repo to GitHub.
2. Go to https://share.streamlit.io → **New app**.
3. Select the repo and branch.
4. Set **Main file path** to `research_funding_dashboard.py`.
5. (Optional) Under **Advanced settings**, set Python version to **3.12**.
6. Click **Deploy**.

No secrets are needed. `requirements.txt` and `.streamlit/config.toml` are picked up
automatically.

## Deploy on Render (alternative)

- **Build command:** `pip install -r requirements.txt`
- **Start command:** `streamlit run research_funding_dashboard.py --server.port $PORT --server.address 0.0.0.0`
- `runtime.txt` pins Python 3.12.

## Data & methodology notes

- **Geography = recipient location**, not place of performance. This ties dollars to
  the institution's home state rather than where work is physically performed.
- **National benchmark is population-weighted:** total obligations ÷ total population
  across the 50 states + DC, so large states are not down-weighted.
- Population figures are 2023 Census estimates.
- Data source: USAspending.gov API, higher-education recipient types only.

## Files

| File | Purpose |
|------|---------|
| `research_funding_dashboard.py` | Streamlit app |
| `requirements.txt` | Pinned dependencies |
| `.streamlit/config.toml` | Theme + server config |
| `runtime.txt` | Python version (Render) |
| `.gitignore` | Standard Python ignores |
