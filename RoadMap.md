# FDE Training Plan — 12-Week Roadmap

**Target:** Forward Deployed Engineer roles at AI platform companies (Cohere, Scale AI, Palantir, and stretch targets at frontier labs) — $300K+ total comp within 6 months, built on a foundation meant to stay relevant 5 years out, not just clear one interview loop.

**Approach:** Databricks-native, SparkSQL-first, project-based — no syntax drills, no hello-world exercises. Every phase produces a real, portfolio-grade artifact on GitHub (`nullPointerRay`).

**Progress:** 4 of ~60 working sessions complete · Phase 1, Project 1 — 40% through

---

## Phase 1 — PySpark & Databricks Data Engineering (Weeks 1–4)

### Project 1: Medallion Stock Pipeline (Live NASDAQ-100 Data)
Repo: [`pyspark-data-engineering-journey`](https://github.com/nullPointerRay/pyspark-data-engineering-journey)

- [x] **Portfolio infrastructure** — Databricks Repos ↔ GitHub sync configured, repo structure established before any code written, repo descriptions and READMEs written properly
- [x] **Bronze layer — Day 1** — yfinance API ingestion, config-driven ticker list (`config/tickers.json`), column normalization, idempotent `MERGE INTO` keyed on `(ticker, date)`, idempotency proven via double-run + `DESCRIBE HISTORY` transaction log audit
- [x] **Bronze data quality finding** — identified and documented a real vendor gap: COST missing one trading day (2026-08-28), confirmed as a yfinance-side issue via set-difference analysis, deliberately left unfixed to motivate Week 3
- [x] **Silver layer — Day 2** — type casting (`TIMESTAMP`→`DATE`), NULL-safe `daily_return` via `LAG()` (NULL means "no prior data," not a fabricated zero), `rolling_avg_7d` with explicit 7-row completeness gating (no fake partial averages), verified via row-count parity and predicted-vs-actual NULL distribution checks
- [x] **Notebook hygiene** — separated production cells from exploratory/debug cells into a dedicated `00_dev_playground` scratch notebook, so scheduled Job runs don't re-execute diagnostic queries
- [x] **Gold layer** — business-level aggregations and features, first real "what does this need to answer" design decisions
- [ ] **Week 3 — Data Quality Engine** — reusable validation framework (Pandera / Great Expectations + DLT expectations), the COST gap becomes the motivating real-world example
- [ ] **Week 4 — Performance & Governance** — partitioning, Z-ordering, broadcast joins, query plan reading, Unity Catalog access control at scale (likely using the larger Kaggle historical dataset to force real optimization decisions)

---

## Phase 2 — ML & AI Libraries, Hands-On (Weeks 5–7)

- [ ] **Week 5 — EDA & Feature Pipeline** — statistical profiling, feature engineering logged to MLflow
- [ ] **Week 6 — Distributed Model Build** — PySpark MLlib at scale + scikit-learn baseline comparison, full train/eval/track loop
- [ ] **Week 7 — Serving & Monitoring** — Databricks Model Serving endpoint, batch inference job, drift monitoring basics

---

## Phase 3 — GenAI, Agents, FDE Sprint (Weeks 8–12)

- [ ] **Week 8 — RAG over Gold Tables** — vector search + retrieval grounded in real pipeline data (Databricks Vector Search, not a toy corpus)
- [ ] **Week 9 — Text-to-SQL Agent + Service Layer** — LLM + tool calling against a live Databricks SQL warehouse, exposed via **FastAPI** (added after gap analysis) as the real service layer beneath the agent
- [ ] **Week 10 — Flagship: NL2SQL Agent + Streamlit UI** — portfolio centerpiece; Gold Delta table as the live queried data source
- [ ] **Week 11 — Productionize** — Docker, GitHub Codespaces, logging, an eval harness for agent output quality
- [ ] **Week 12 — Portfolio Polish + Interview Reps** — case-study README, live demo rehearsals (Venkat, Nakul, Ashish), FDE interview prep

---

## Parallel Track — Job Search Execution
*Started alongside skill-building, not after it — interview loops take 4–8 weeks on their own.*

- [ ] Flagship demo functional enough to show (~Week 8–9 checkpoint) → begin warm-network outreach
- [ ] Target applied-AI startup tier first (Scale AI, Cohere — mid-level band already clears $300K)
- [ ] Referral conversations with Venkat, Nakul, Ashish
- [ ] Frontier lab stretch applications (Anthropic, OpenAI) once flagship project is fully live-demoable
- [ ] Comp research refreshed close to active interview stage (bands move)

---

## Design Decisions & Technical Judgment

[#design-decisions--technical-judgment](#design-decisions--technical-judgment)

**Bronze — Data Sourcing & Quality**
- Identified a genuine upstream vendor data gap (COST missing one trading day) through independent set-difference verification rather than trusting row counts at face value — treated as a documented, deliberate finding rather than silently patched, since fabricating a missing row would have been a worse outcome than an honest gap.
- Designed the ingestion layer around idempotent `MERGE` semantics from day one, anticipating incremental ticker additions, and proved idempotency empirically — two independent methods (merge operation statistics and a full row-count rerun) — before trusting the pipeline as rerun-safe.

**Silver — Data Semantics**
- Established a firm distinction between "no data exists" and "no change occurred" in derived metrics — a `NULL` default was chosen over a fabricated `0` for `daily_return`'s boundary case, since collapsing those two meanings into one value would silently mislead any downstream consumer, human or model.
- Applied the same completeness-first principle to rolling averages: a window with fewer observations than its stated size is explicitly nulled rather than allowed to silently report a partial average under a label implying completeness.

**Gold Daily — Metric Design**
- Designed four independent business metrics (rolling volume, relative volume, return volatility, swing volatility), each with its own correctness-appropriate completeness gate rather than one generic rule copy-pasted across all four.
- Identified and corrected a dependency design flaw prior to materialization: a price-derived metric (`swing_volatility_20d`) was initially coupled to an unrelated column's completeness state (`volume`). The coupling was invisible against current data — a defect class that only surfaces under future data conditions — which is exactly the kind of hidden dependency a design review needs to catch, not a runtime failure.
- Made a deliberate distinction between two related but non-equivalent volatility concepts: close-to-close return volatility versus intraday high/low range, recognizing that a stock can appear "calm" on daily returns while still exhibiting significant intraday swings — the same reasoning underlying real quantitative volatility estimators (Parkinson's, Garman-Klass).
- Selected sample-based standard deviation over population-based, correctly reasoning that a rolling window is an estimate drawn from an ongoing, incomplete process rather than a closed, fully-observed population.

**Gold Weekly — Grain & Aggregation Design**

- Recognized independently that weekly aggregation required a fundamentally different query pattern (`GROUP BY`) than the row-preserving window functions used throughout Bronze/Silver/Gold Daily — a genuine grain change, not an incremental extension of prior work.
- Corrected an initial design flaw before implementation: naively deriving weekly open/close via `MIN`/`MAX` would have fabricated candles from mismatched trading days rather than reflecting the actual first and last session of the week — resolved using ordered first/last-value logic instead.
- Independently verified a flagged discrepancy against raw source numbers rather than accepting a correction at face value, and was proven right — the correction itself had miscounted a column position.
- Identified and corrected a non-obvious SQL semantics trap in default window framing that would have silently and identically mis-priced every weekly close — a class of bug invisible without deliberate frame-boundary testing.
- Clarified an ambiguous metric definition ("trending") into two precise, independently meaningful measures — return-based ranking and volume-based ranking — rather than allowing one overloaded label to obscure which concept was actually being surfaced.
- Caught a self-introduced labeling error before shipping: an initially proposed "52-week trailing volatility" was actually a full annual measure, not a monthly-comparable one — corrected both the window size and the column name to honestly reflect what was being measured.
- Introduced a conservation check (sum of trading days across all weekly periods equals the total daily row count) as a stronger correctness proof than row count alone — verifying no daily record was lost or double-counted across grain boundaries.
---

*Last updated: Day 4 (Gold layer Weekly metrics; Gold monthly next session)*

