# Last-Mile Delivery Exception — Multi-Agent System

**A multi-agent AI system that handles last-mile delivery exceptions end-to-end — detecting actionable exceptions, deciding a resolution from policy, escalating to a human only when rules require, and drafting a personalized customer message — with deterministic guardrails and auditable reasoning.**

### 📊 [**View the Project Report →**](https://fomenky.github.io/delivery-exception-multi-agent-system/)

Published via GitHub Pages — no download or setup required. The landing page is a clean **executive summary**; it links through to the **full detailed report**. Direct links:

- **Executive summary (Pages landing page):** https://fomenky.github.io/delivery-exception-multi-agent-system/
- **Full detailed report:** https://fomenky.github.io/delivery-exception-multi-agent-system/reports/delivery_exception_mas_full_report.html
- Source copies: [`reports/executive_summary.html`](reports/executive_summary.html) · [`reports/delivery_exception_mas_full_report.html`](reports/delivery_exception_mas_full_report.html)

---

## Problem

Roughly 10% of last-mile shipments hit exceptions — failed attempts, address mismatches, damaged or refused packages, weather delays. Most providers still triage these manually: staff read messy driver notes, cross-reference customer profiles and playbooks, decide an action, and draft a customer notification, all under time pressure. Rigid rule engines can't make the nuanced, multi-factor calls real exceptions demand — a VIP with a perishable package and a broken gate code on a second attempt needs a very different response than a standard customer's first failed attempt.

## Objective

Build a **proof-of-concept multi-agent system** that automates the full pipeline — ingest raw (noisy, duplicated) delivery logs, identify actionable exceptions, decide the resolution (reschedule, reroute to locker, replace, return to sender) by reasoning over playbook rules, customer context, package constraints, and locker eligibility, **escalate to a supervisor only when policy requires**, and generate a personalized notification calibrated to customer tier and severity — producing **auditable decisions** with step-by-step rationale and system metrics.

## Architecture

A single orchestrator routes each shipment through specialist agents over a shared, typed state. **Deterministic guardrails run before any LLM call, and a hard-coded rule engine — not an LLM — owns automatic escalation.**

```
Preprocessor → Orchestrator → Resolution Agent → Critic → Communication Agent → Critic → Finalize
(dedup,         (deterministic   (classify +        (validate   (customer         (validate   (package +
 guardrails)     router)          resolve)           resolution) message)          message)    metrics)
```

**Tools:** `read_delivery_logs`, `lookup_customer_profile` (PII-flagged), `check_locker_availability`, `search_playbook` (RAG over an operations playbook via Chroma, page-cited), `check_escalation_rules` (fully deterministic rule engine).
**Models:** `gpt-4o-mini` (agents), `gpt-4o` (critics/judge), `text-embedding-3-small`.

## Data

Course-provided, synthetic:

| File | Contents |
|------|----------|
| `data/customers.db` (SQLite) | 12 customers (tier, channel, credit — **synthetic PII: names**) + 6 lockers |
| `data/delivery_logs.csv` | 13 rows across 10 shipments (includes noise/duplicates) |
| `data/exception_resolution_playbook.pdf` | Operations playbook (RAG source, 36 chunks) |
| `data/ground_truth.csv` | Expected outcomes for the 10-scenario benchmark |

> **Privacy by design:** customer names are flagged as PII at the tool layer and withheld unless the Communication Agent explicitly needs them. The dataset is small and synthetic (not real customers).

## Evaluation & key results

On the 10-scenario curated benchmark:

| Dimension | Result |
|-----------|-------:|
| Task completion rate | **100%** |
| Escalation accuracy | **100%** |
| Tool-call accuracy | **100%** |
| Reasoning trajectory coherence | **4.4 / 5** |

Every decision is grounded in cited playbook pages; escalation is deterministic; noise and prompt-injection are filtered before any model call; a critic/revision loop (max 2) validates outputs.

## Interpretation

Combining **deterministic guardrails for the high-stakes decisions** (escalation, PII handling) with **LLM reasoning for the nuanced judgment** (resolution choice, message tone) yields consistent, auditable behavior — the properties an operations team needs before trusting automation on live feeds.

## Limitations & potential improvements

- Small curated benchmark (10 scenarios); real deployment needs a far larger, noisier exception set and live delivery feeds.
- Single-region, limited exception taxonomy and locker model.
- LLM-as-judge for reasoning coherence complements but does not replace human audit.
- No real-time integration, monitoring, or human-in-the-loop UI (out of PoC scope).

## Technologies

Python · **LangGraph** · LangChain · Chroma (RAG) · OpenAI (`gpt-4o-mini`, `gpt-4o`, `text-embedding-3-small`) · Pydantic · SQLite · DeepEval · Jupyter

## Reproducing the analysis

> **Requires an OpenAI API key.** The system calls hosted LLM/embedding APIs. Copy `config.example.json` to `config.json` (git-ignored) and fill in your key, or set environment variables. The Chroma store is rebuilt from the playbook on first run (not committed).

```bash
git clone https://github.com/fomenky/delivery-exception-multi-agent-system.git
cd delivery-exception-multi-agent-system
python3 -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt

cp config.example.json config.json   # then add your OPENAI_API_KEY (never commit it)
# LangSmith tracing is optional and off by default.

jupyter notebook notebooks/delivery_exception_mas.ipynb
```

## Repository layout

```text
delivery-exception-multi-agent-system/
├── README.md
├── index.html                 # GitHub Pages landing page (executive summary → full report)
├── notebooks/
│   └── delivery_exception_mas.ipynb
├── reports/
│   ├── delivery_exception_mas_full_report.html
│   └── executive_summary.html
├── data/                      # SQLite + logs + playbook + ground truth (synthetic)
├── config.example.json        # template — copy to config.json and add your key
├── requirements.txt
├── .gitattributes
└── .gitignore
```

---

### About this project

Completed as a capstone project in the **Post Graduate Program in Artificial Intelligence & Machine Learning (UT Austin / Great Learning)**. Shared to demonstrate applied multi-agent / LangGraph engineering with deterministic guardrails and evaluation. The business scenario and datasets are course-provided, synthetic academic material; results are a proof-of-concept on the curated benchmark and are **not** claims of production deployment or commercial outcomes. No API keys or credentials are included in this repository.
