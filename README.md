# A/B Testing & Experimentation Framework
## One-Click Trade Execution - Trading Platform Feature Experiment

> **Trading experimentation project**  
> Built on top of an existing `brokerage_db` PostgreSQL 17 data warehouse schema.

---

## Project Overview

This project implements a complete, **A/B Testing & Experimentation Framework** for a trading and brokerage analytics system. The experiment evaluates whether a new **One-Click Trade Execution** feature increases client trading activity and brokerage revenue while maintaining platform stability.

The framework covers the full experimentation lifecycle:
- Experiment design (hypothesis, MDE, power analysis)
- Schema extension and data generation
- Data preparation and cleaning
- Exploratory data analysis
- SQL-based metric computation
- Statistical hypothesis testing (chi-square, z-test, t-test, confidence intervals, SRM, power)
- Segment analysis
- Decision framework and executive reporting

---

## Trading Context

A retail and HNI (High Net-worth Individual) brokerage platform serves clients across Equity and F&O (Futures & Options) segments. The standard order flow required clients to complete 5 steps to place a trade. The new **One-Click Trade Execution** feature reduces this to a single interaction using pre-configured defaults.

**Business Question:** Does removing order-placement friction meaningfully increase trading participation and revenue?

---

## Dataset

### Existing Schema (brokerage_db)

| Table | Key Columns | Description |
|---|---|---|
| `clients` | client_id, client_type, risk_profile | 600 client profiles |
| `trading_accounts` | account_id, client_id, account_type, status | Client trading accounts |
| `trades` | trade_id, account_id, trade_value, segment, channel | Individual trade records |
| `brokerage_revenue` | trade_id, brokerage_fee, total_revenue | Revenue per trade |
| `operational_events` | event_id, account_id, event_type, resolution_time_hours | Platform events |

### Schema Extensions (added to this repo)

| Table | Key Columns | Description |
|---|---|---|
| `experiment_assignments` | client_id, experiment_id, variant, assigned_date | Control/treatment assignment |
| `feature_flags` | client_id, feature_name, enabled | Feature gate per client |

### Generated Analysis Dataset

`data/client_metrics.csv` - One row per client with all metrics joined:

| Column | Description |
|---|---|
| `variant` | `control` or `treatment` |
| `conversion_flag` | 1 if client placed ≥1 trade during experiment |
| `total_trades` | Number of trades placed |
| `total_revenue` | Total brokerage revenue generated |
| `avg_trade_value` | Average value per trade |
| `error_flag` | 1 if client encountered an operational error |
| `avg_resolution_hours` | Average time to resolve operational events |

---

## Experiment Design

| Parameter | Value | Rationale |
|---|---|---|
| Experiment ID | `EXP_2024_ONE_CLICK` | |
| Window | 2024-01-01 → 2024-02-29 | 60 days, 2+ trading cycles |
| Sample Size | 600 clients | 300 control / 300 treatment |
| Randomization Unit | Client ID | Prevents cross-contamination |
| Split | 50/50 | Maximizes statistical power |
| Alpha (α) | 0.05 | Industry standard |
| Power (1–β) | 0.80 | Standard product experiment threshold |
| Baseline Conversion | 35% | Historical platform average |
| MDE | +3pp absolute | ~8.6% relative - business meaningful |

**Hypotheses:**

- **H₀:** Conversion rate of treatment = Conversion rate of control
- **H₁:** Conversion rate of treatment > Conversion rate of control (one-sided)

**Guardrail Metrics (must not breach):**
- Operational error rate: ≤ +3pp increase
- Avg resolution time: ≤ +1 hour increase

---

## Statistical Methods

All methods are fully implemented in code with visible outputs and interpretations:

| Method | Library | Purpose |
|---|---|---|
| **Chi-Square Test** | `scipy.stats.chi2_contingency` | Test independence of variant and conversion |
| **Two-Proportion Z-Test** | `statsmodels.stats.proportion.proportions_ztest` | Directional test: treatment > control |
| **Welch's T-Test** | `scipy.stats.ttest_ind(equal_var=False)` | Revenue per client comparison |
| **Confidence Interval (Conversion)** | Manual (normal approximation) | 95% CI for conversion lift |
| **Confidence Interval (Revenue)** | Manual (Welch-Satterthwaite) | 95% CI for revenue lift |
| **Sample Ratio Mismatch** | Chi² goodness-of-fit | Validate 50/50 assignment |
| **Power Analysis** | `statsmodels.stats.power.NormalIndPower` | Verify sample size adequacy |

---

## Results Summary

| Metric | Control | Treatment | Lift | Significant? |
|---|---|---|---|---|
| Conversion Rate | ~35% | ~42% | +7pp |  Yes (p < 0.05) |
| Revenue per Client | ~₹45 | ~₹55 | +22% |  Yes (p < 0.05) |
| Avg Trades/Client | ~1.05 | ~1.68 | +60% |  Yes |
| Error Rate | ~15% | ~17% | +2pp | Within limit |
| Avg Resolution Time | ~1.5 hrs | ~1.6 hrs | +0.1 hr | Within limit |

---

## Tools Used

| Tool | Version | Purpose |
|---|---|---|
| Python | 3.10+ | Core analysis language |
| Pandas | 2.x | Data manipulation and SQL-equivalent queries |
| NumPy | 1.x | Numerical computation, random data generation |
| SciPy | 1.x | Chi-square, t-test, normal distribution |
| statsmodels | 0.14+ | Z-test, power analysis |
| Matplotlib | 3.x | Charts and visualizations |
| Seaborn | 0.13+ | Statistical visualizations |
| PostgreSQL | 17 | Source database (brokerage_db) |
| Jupyter Notebook | 7.x | Interactive analysis environment |

---

## Project Structure

```
a-b_testing/
├── notebooks/
│   └── ab_testing_trading.ipynb     # Main analysis notebook (10 sections)
├── sql/
│   ├── schema_extensions.sql        # DDL for experiment_assignments & feature_flags
│   └── analysis_queries.sql         # 13 analysis SQL queries + view definition
├── scripts/
│   └── data_preparation.py          # Synthetic data generation + PostgreSQL loader
├── reports/
│   └── executive_summary.md         # 1-page executive report
├── data/                            # Generated CSVs (after running data_preparation.py)
│   ├── client_metrics.csv
│   ├── trades.csv
│   ├── brokerage_revenue.csv
│   └── ...
└── README.md
```

---

## Quick Start

### Option A: Run Notebook Standalone (no database needed)

```bash
pip install pandas numpy scipy statsmodels matplotlib seaborn jupyter

cd notebooks
jupyter notebook ab_testing_trading.ipynb
```


### Option B: Full Pipeline with PostgreSQL

```bash
# 1. Update DB credentials in scripts/data_preparation.py
# 2. Run schema extensions
psql -U postgres -d brokerage_db -f sql/schema_extensions.sql

# 3. Generate data and load to DB
python scripts/data_preparation.py --load-db

# 4. Run analysis queries
psql -U postgres -d brokerage_db -f sql/analysis_queries.sql

# 5. Open notebook
jupyter notebook notebooks/ab_testing_trading.ipynb
```

