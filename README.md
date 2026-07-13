# Federal Research Funding to Higher Education (FY2021–2026)

Benchmarking federal research funding to Mississippi's universities against the nation — built on the live [USAspending.gov](https://api.usaspending.gov) API.

Federal research funding drives university research capacity and regional economic growth, but states differ sharply in what their institutions receive. This dashboard makes those differences visible and reproducible. It pulls federal obligations to higher-education recipients across six agencies (**DoD, DOE, HHS, DHS, NSF, NASA**), normalizes them per capita, and compares any state — Mississippi by default — against a population-weighted national benchmark.

There is no offline dataset, no API key, and no manual data entry. Every figure on screen is re-aggregated from the same live USAspending pull.

---

## Features

**Per-capita choropleth map** — obligations by state, for a selected agency and fiscal year. Per capita is the default fill, because raw totals mostly just draw a population map.

**Top university recipients** — nationally, or drilled down to a single state.

**Funding trend (FY2021–2026)** — for the selected agency and state.

**Mississippi vs national, per agency** — a six-panel per-capita comparison against the population-weighted national benchmark. Independent y-axes per panel, so small-budget agencies aren't flattened by HHS.

**Headline verification** — the per-agency table, the aggregate "cents on the dollar" figure, and a FY2021→FY2025 trend table flagging whether each agency's gap is narrowing, flat, or widening. Computed from the same cached data as the charts, so presentation numbers can't drift from the dashboard. Includes a robustness panel and CSV export.

---

## Run locally

```bash
python -m venv .venv
# Windows:      .venv\Scripts\activate
# macOS/Linux:  source .venv/bin/activate

pip install -r requirements.txt
streamlit run research_funding_dashboard.py
```

Opens at `http://localhost:8501`.

The Mississippi-vs-national panels fetch 6 agencies × 6 years on a cold cache — expect **30–60 seconds** on first load, then it's cached for an hour. Use the sidebar's **Refresh data** button to force a fresh pull.

---

## Deploy

### Streamlit Community Cloud

1. Push this repo to GitHub.
2. Go to [share.streamlit.io](https://share.streamlit.io) → **New app**.
3. Select the repo and branch.
4. Set **Main file path** to `research_funding_dashboard.py`.
5. *(Optional)* Under **Advanced settings**, set Python version to 3.12.
6. **Deploy.**

No secrets required. `requirements.txt` and `.streamlit/config.toml` are picked up automatically.

### Render

| Setting | Value |
|---|---|
| Build command | `pip install -r requirements.txt` |
| Start command | `streamlit run research_funding_dashboard.py --server.port $PORT --server.address 0.0.0.0` |

`runtime.txt` pins Python 3.12.

---

## Methodology

These four choices determine every number the dashboard reports. Read them before citing it.

### Geography is recipient location — never place of performance

Dollars are attributed to the **institution's home state**, not to wherever the work is physically carried out.

This distinction is not cosmetic. Under place of performance, a University of Puerto Rico award executed at Stennis Space Center counts as Mississippi funding — which answers *"where did the work happen?"* rather than *"which state's institutions received the money?"* Only the second question supports a state-level benchmark. All API calls use `scope: "recipient_location"` and `recipient_locations`; `place_of_performance` appears nowhere in the codebase, and should not be reintroduced.

### Award types are specified explicitly

Grants (`02`, `03`, `04`, `05`) and contracts (`A`, `B`, `C`, `D`).

The USAspending geography endpoint **under-counts when `award_type_codes` is omitted** — DOE FY2025 returned roughly \$3B instead of the full ~\$8B. The codes are listed in full to force a complete total. Do not "simplify" this by removing them.

### The national benchmark is population-weighted

```
national per-capita = total obligations across 50 states + DC
                      ÷ combined population of those same 50 states + DC
```

Numerator and denominator cover the same geography, so large states are not down-weighted the way an unweighted mean of state rates would do. Population figures are 2023 Census estimates.

### The aggregate headline is dollar-weighted

The "cents on the dollar" figure is total Mississippi per-capita across all six agencies divided by total national per-capita — equivalent to `(all MS obligations ÷ MS pop) ÷ (all national obligations ÷ national pop)`.

Because HHS is most of the money, HHS dominates this figure. That's intended: it answers *what does Mississippi actually receive per person?* The unweighted mean of the six agency ratios is reported alongside it and is **much higher**, since it lets a sub-dollar-per-capita agency count as much as a \$78-per-capita one. Don't quote the unweighted figure as the headline — but know it exists, because it's the obvious objection.

---

## Known caveats

**FY2026 is a partial fiscal year.** It ends September 30. Every series — Mississippi and national alike — drops sharply in FY2026, and that cliff is an artifact of incomplete data, not a funding collapse. All headline statistics are computed over **FY2021–2025 only**. The FY2026 points remain visible in the trend charts and are labeled as partial.

**DHS ratios are volatile.** Mississippi exceeds the national DHS rate, but the base is well under \$1 per person, so a single award can swing the percentage dramatically. Check the recipient table before leaning on it.

**Per capita is a framing choice.** It answers *what does the state's population get back?* It does **not** answer *are a given institution's proposals judged fairly?* A state with fewer research universities will mechanically show fewer dollars per resident. Both questions are legitimate; this dashboard answers the first.

**Obligations, not outlays.** These are amounts the government has committed, not cash disbursed.

---

## Validating against USAspending.gov

To reproduce any figure in the UI:

1. **Advanced Search** → set the fiscal year.
2. **Agency** → awarding agency (not funding agency).
3. **Recipient Type** → Higher Education.
4. **Location** → **Recipient Location**, not Place of Performance.
5. **Award Type** → contracts and grants.

Then compare against **Spending by Geography**. Expect sub-percent differences from display rounding.

> Validation performed before the geography fix should be re-run. Earlier spot-checks used place-of-performance filters and will not match current output.

---

## Files

| File | Purpose |
|---|---|
| `research_funding_dashboard.py` | Streamlit app |
| `requirements.txt` | Pinned dependencies |
| `.streamlit/config.toml` | Theme + server config |
| `runtime.txt` | Python version (Render) |
| `.gitignore` | Standard Python ignores |

---

## Data source

[USAspending.gov API](https://api.usaspending.gov) — public, open, no key required. Filtered to higher-education recipient types (public, private, and minority-serving institutions).

## Citation

> Mukherjee, E., Pricope, N., & Choi, S. (2026). *Benchmarking Federal Research Funding to Mississippi Higher Education: An Interactive, Validated Dashboard Across Six Agencies (FY2021–2026).* Mississippi State University.

The dashboard generalizes to any state — swap the target state code and the benchmark logic is unchanged.
