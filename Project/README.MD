# Burying the Lines

**Do underground power lines reduce outages at U.S. investor-owned utilities?**

MSBA 502: Analytics Programming, University of San Diego
Andrew Massa · Jeremiah Cedano

---

## Problem

Investor-owned utilities (IOUs) serve about three-quarters of U.S. electricity customers, yet reliability varies widely:

- **108 min:** median outage time per customer per year (normal days)
- **929 min:** worst utility-year in our data, almost 9× the median

**Question:** which investments actually reduce outage minutes?

**Stakeholders**
- **Regulators:** approve utility spending and set reliability targets
- **Executives:** decide where capital and maintenance dollars go
- **Investors:** affected by spending decisions and regulatory treatment
- **Customers:** live with outages and pay for upgrades through their bills

## Data

| Source | What we used | Level |
|---|---|---|
| EIA Form 861 | Outage minutes (SAIDI), customers, smart meters, circuits | Utility, state, year |
| FERC Form 1 | Overhead and underground line dollars, line maintenance spending | Company, year |
| PUDL crosswalk (Catalyst Cooperative) | Matches each company's FERC ID to its EIA ID | Company |

Years: **2021–2025**

**Row funnel**

| Step | Rows |
|---|---|
| IOU rows with outage data | 845 |
| IEEE-standard outage measure only | 705 |
| Utilities that also file with FERC | 582 |
| Final, after removing tiny service areas (<1,000 customers) | **577** (89 utilities) |

Only IEEE-standard outage figures were kept so every utility measures downtime the same way. Tiny service areas were dropped because their SAIDI swings wildly year to year.

## Variables

| Variable | Definition | Source |
|---|---|---|
| **SAIDI** (outcome) | Outage minutes per customer per year, major event days excluded | EIA |
| Underground share | Underground line $ ÷ all line $ | FERC |
| Maintenance per customer | Overhead + underground line maintenance $ ÷ customers | FERC |
| Smart meter share | Smart (AMI) meters ÷ all meters | EIA |
| Customers per circuit | Customers ÷ distribution circuits | EIA |

## Method

1. **Build:** pandas to filter, group, pivot and merge 6 files into one table of ratios (`clean_utility_data.csv`).
2. **Model:** statsmodels OLS
   - Regression 1: SAIDI on underground share alone
   - Regression 2: SAIDI on all four variables
3. **Stress-test:** rerun Regression 2 with SAIDI *including* major event days, then predict gains for the 10 worst 2025 utilities if raised to average underground share.

Standard errors are clustered by utility, since the same company appears across multiple states and years.

## Results

| Model | Underground share effect | Fit |
|---|---|---|
| Regression 1 (underground only) | −29 min per +10 pts (p < 0.001) | R² = 0.17 |
| Regression 2 (all four) | −22 min per +10 pts (p = 0.001) | Adj. R² = 0.27 |
| Regression 2, major events included | −88 min per +10 pts (p ≤ 0.001) | R² = 0.06 |

- **Underground share** is the only variable linked to fewer outages in every model.
- **Maintenance per customer:** +6.5 min per $10 (p = 0.03), likely reverse causality: utilities with more outages spend more on repairs.
- **Smart meters** (p = 0.82) and **customers per circuit** (p = 0.59): no significant effect.
- **Storm insurance:** underground lines matter ~4× more once disasters are included, though storms make outages much harder to predict overall.
- **Worst 10 utilities (2025):** raising underground share to the 37.9% average saves **37 min (~9.4%)** on average, up to 75 min for Kentucky Power. Alaska Electric Light & Power is already at average and saves nothing, so other factors drive its outages.

## Limitations

- **Correlation, not causation:** utilities with more buried lines may also be newer or more urban.
- **Dollars, not miles:** underground line costs more per mile, so the dollar share overstates how much is actually buried.
- **Weather:** ordinary storms are not controlled for.

## Next Steps

- Cost per outage minute saved: undergrounding cost per mile vs. minutes gained
- Add state-level weather data to separate storms from utility performance

## How to Run

```bash
pip install pandas numpy matplotlib seaborn statsmodels
```

1. Run the cleaning and merging notebook to build `clean_utility_data.csv`.
2. Run the analysis notebook for summary stats, plots, regressions and predictions.

## Contributions

| Member | Code | Slides |
|---|---|---|
| Andrew Massa | 50%: data collection, cleaning and merging | 50% |
| Jeremiah Cedano | 50%: analysis, regressions and plots | 50% |

Repos: [Andrew](https://github.com/amassa96/MSBA_502/tree/main/Project) · [Jeremiah](https://github.com/jrcedano/MSBA-502-Utility-Reliability-Analysis/tree/main)
