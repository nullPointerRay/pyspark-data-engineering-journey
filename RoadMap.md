# FDE Training Plan — 12-Week Roadmap

**Target:** Forward Deployed Engineer roles at AI platform companies (Cohere, Scale AI, Palantir, and stretch targets at frontier labs) — $300K+ total comp within 6 months, built on a foundation meant to stay relevant 5 years out, not just clear one interview loop.

**Approach:** Databricks-native, SparkSQL-first, project-based — no syntax drills, no hello-world exercises. Every phase produces a real, portfolio-grade artifact on GitHub (`nullPointerRay`).

**Progress:** 2 of ~60 working sessions complete · Phase 1, Project 1 — 40% through

---

## Phase 1 — PySpark & Databricks Data Engineering (Weeks 1–4)

### Project 1: Medallion Stock Pipeline (Live NASDAQ-100 Data)
Repo: [`pyspark-data-engineering-journey`](https://github.com/nullPointerRay/pyspark-data-engineering-journey)

- [x] **Portfolio infrastructure** — Databricks Repos ↔ GitHub sync configured, repo structure established before any code written, repo descriptions and READMEs written properly
- [x] **Bronze layer — Day 1** — yfinance API ingestion, config-driven ticker list (`config/tickers.json`), column normalization, idempotent `MERGE INTO` keyed on `(ticker, date)`, idempotency proven via double-run + `DESCRIBE HISTORY` transaction log audit
- [x] **Bronze data quality finding** — identified and documented a real vendor gap: COST missing one trading day (2026-08-28), confirmed as a yfinance-side issue via set-difference analysis, deliberately left unfixed to motivate Week 3
- [x] **Silver layer — Day 2** — type casting (`TIMESTAMP`→`DATE`), NULL-safe `daily_return` via `LAG()` (NULL means "no prior data," not a fabricated zero), `rolling_avg_7d` with explicit 7-row completeness gating (no fake partial averages), verified via row-count parity and predicted-vs-actual NULL distribution checks
- [x] **Notebook hygiene** — separated production cells from exploratory/debug cells into a dedicated `00_dev_playground` scratch notebook, so scheduled Job runs don't re-execute diagnostic queries
- [ ] **Gold layer** — business-level aggregations and features, first real "what does this need to answer" design decisions
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

## Wins Log (real debugging stories — pull these for interviews)
- Caught a genuine vendor data gap (COST, missing trading day) through independent set-difference verification, not by assumption
- Proved MERGE idempotency with two independent methods (merge stats + row count) before trusting the pipeline
- Caught and fixed a silent semantic bug: a `0` default that visually looked correct but conflated "no change" with "no data" — same for a rolling average that was silently averaging incomplete windows
- Diagnosed a `NO_SUCH_CATALOG_EXCEPTION` from first principles (`SHOW CATALOGS`) instead of guessing

---

*Last updated: Day 2 (Bronze + Silver complete, Gold layer next session)*

