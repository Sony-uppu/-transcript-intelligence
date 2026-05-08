# Transcript Intelligence — AegisCloud

**Sony Uppu · AegisCloud Take-Home Assessment · May 2026**

An AI-powered pipeline that analyses 105 customer call transcripts to surface churn risk, feature gaps, and revenue intelligence — automatically, in under a minute.

---

## Live Dashboard

**[sony-uppu.github.io/-transcript-intelligence](https://sony-uppu.github.io/-transcript-intelligence/)**

Live data · Auto-refreshes from GitHub · Six pages · ARIA AI analyst built in

---

## What This Does

AegisCloud had 105 recorded customer calls with no systematic analysis. This pipeline:

- Classifies every call by **type** (external / internal / support) and **theme** (Compliance, Incident, Renewal, etc.)
- Scores **sentiment** per call and flags accounts below the pre-churn threshold
- Extracts **churn signals** and **feature gaps** from AI-tagged key moments
- Exports everything to a **live dashboard** any stakeholder can open in their browser
- Powers **ARIA** — an AI analyst that answers natural language questions grounded in the live data

---

## Key Findings

| Metric | Value |
|--------|-------|
| Transcripts analyzed | 105 (100 original + 5 synthetic) |
| Call type split | 43 external · 30 internal · 32 support |
| High-risk calls (score ≤ 2.5) | 8 |
| Churn signals detected | 63 |
| Feature gaps surfaced | 52 |
| Action items (untracked) | 407 |

**Core finding:** Incidents are not a support problem — they are a churn problem. Every at-risk account traces back to an unresolved incident. Incident & Outage calls average 2.53/5. Compliance & Audit calls average 3.62/5.

---

## Repository Structure

```
-transcript-intelligence/
│
├── dataset/                         ← Original 100 transcript folders (untouched)
│   └── <meeting-id>/
│       ├── meeting-info.json        ← Title, emails, duration
│       ├── summary.json             ← Sentiment, topics, action items, key moments
│       ├── transcript.json          ← Raw sentence-level transcript
│       ├── speakers.json
│       ├── events.json
│       └── speaker-meta.json
│
├── analysis.ipynb                   ← Full 23-cell pipeline notebook
├── dashboard_data.json              ← Processed data fetched by the live dashboard
├── index.html                       ← Live dashboard (GitHub Pages)
├── results.json                     ← Base 100-record output
├── results_augmented.json           ← 105-record output (inc. synthetic)
├── sentiment_by_theme.png           ← Base sentiment chart
├── sentiment_by_theme_augmented.png ← Augmented sentiment chart
└── README.md
```

---

## Pipeline — How It Works

The notebook (`transcript_pipeline.ipynb`) runs in 23 cells across four stages:

### Stage 1 — Load Dataset
Reads `meeting-info.json` and `summary.json` from all 100 folders. The summaries already contain AI-extracted sentiment scores, topics, action items, and key moments — AegisCloud's platform had already done an LLM pass over the raw transcripts.

### Stage 2 — Classify Call Type
```python
# Title check runs BEFORE domain check
if re.search(r"support case", title, re.IGNORECASE):
    call_type = "support"
elif domains == {"aegiscloud.com"}:
    call_type = "internal"
else:
    call_type = "external"
```
**Why this matters:** Domain-only classification misclassified 27 real support calls as external. The `Support Case #` title pattern is the reliable discriminator.

### Stage 3 — Classify Theme
Keyword matching on the AI-tagged topics field. Eight categories in priority order so specific themes win over broad ones.

| Theme | Calls | Avg Sentiment |
|-------|-------|---------------|
| Compliance & Audit | 67 (64%) | 3.62 |
| Incident & Outage | 20 (19%) | 2.53 |
| Renewal & Churn Risk | 5 (5%) | 3.16 |
| Security & Identity | 5 (5%) | 3.16 |
| Product & Roadmap | 3 (3%) | 3.77 |
| Onboarding & Deployment | 2 (2%) | 4.60 |
| Technical Support | 2 (2%) | 2.60 |
| Other | 1 (1%) | 3.90 |

### Stage 4 — Extract Signals
Loops through the `keyMoments` array in every summary and counts by type:
- `churn_signal` → 63 total
- `feature_gap` → 52 total
- `action_items` → 407 total (none currently tracked)

---

## Sentiment Threshold

**2.5 out of 5** is the pre-churn threshold.

On a 1–5 scale, 2.5 is the exact midpoint — below it means the call was net negative. Validated by spot-checking: every call below 2.5 at the individual account level shows explicit customer distress in the summary content.

In production, this threshold would be backtested against historical churn data to find the score below which churn rate actually spikes.

---

## Synthetic Data

5 support call records were generated to fill the missing call type. The assessment FAQ explicitly permits this.

Every synthetic record:
- Follows the exact dataset schema
- Uses the `Support Case #` title pattern matching real support calls
- Is labeled `"synthetic": true` in all output files
- Can be filtered out: `[r for r in records if not r.get("synthetic")]`

---

## ARIA — AI Analyst

ARIA (AegisCloud Real-time Intelligence Analyst) is built into the dashboard and powered by the Claude API.

**How it works:**
1. On page load, the dashboard fetches live `dashboard_data.json`
2. Builds a context summary: sentiment averages, at-risk customers, feature gaps, competitors
3. Injects that as the system prompt to Claude Sonnet
4. User asks questions — ARIA answers from the actual dataset, not general knowledge

**Example questions:**
- *"Which customers are closest to churning?"*
- *"What should the PM prioritise in Q3?"*
- *"Why do incident calls score so low?"*
- *"Who is SentinelShield and why do they matter?"*

> A paid Anthropic API key is required. Open `index.html` and replace `PASTE_YOUR_KEY_HERE` with your key from [console.anthropic.com](https://console.anthropic.com).

---

## Feature Gap Breakdown

| Category | Count | Key Detail |
|----------|-------|------------|
| Identity & Access | 20 | Nested roles, JIT provisioning, conditional MFA |
| SIEM Connector UX | 13 | 6-page config vs competitor 3 clicks — in deal-loss calls |
| Compliance Reporting | 10 | On-demand reports unavailable — 3 customers blocked on audits |
| Detect / Alerting | 7 | Alert fatigue after patch — team ignoring all alerts |
| Other | 2 | Miscellaneous |

---

## Top At-Risk Accounts

| Customer | Score | Churn Signals | Theme | Status |
|----------|-------|---------------|-------|--------|
| Blackridge Investments | 1.6 | 1 | Incident & Outage | ⚠ Critical |
| Cobalt Software | 1.8 | 1 | Incident & Outage | ⚠ Critical |
| Northstar Pharma | 2.1 | 2 | Compliance & Audit | ⚠ Critical |
| Helix Data | 2.3 | 1 | Incident & Outage | High |
| Meridian Capital | 2.4 | 1 | Compliance & Audit | High |
| Quantum Edge | 2.4 | 1 | Compliance & Audit | High |
| Summit Trust | 2.4 | 2 | Incident & Outage | High |
| Brightpath Commerce | 2.6 | 1 | Compliance & Audit | Watch |

---

## How to Run

```bash
# Clone the repo
git clone https://github.com/Sony-uppu/-transcript-intelligence.git
cd -transcript-intelligence

# Install dependencies
pip install matplotlib

# Open the notebook
jupyter notebook analysis.ipynb
```

Running the notebook end-to-end exports `dashboard_data.json`. Push to GitHub → dashboard updates automatically.

---

## How I Used Claude AI

Three ways:

1. **Thinking through the problem** — Worked through edge cases in classification logic, validated threshold rationale, structured the analysis approach.

2. **Building ARIA** — ARIA runs on Claude Sonnet API with a dynamically built system prompt grounded in the live dataset. Not a generic chatbot — a grounded analyst.

3. **Building faster** — Used Claude to write cleaner code, structure notebook markdown, and build the HTML dashboard. All AI assistance is transparently noted.

---

## Verification

- All numbers come directly from looping through actual JSON files — nothing estimated
- Spot checked 5 lowest and 5 highest scoring calls — classifications match content
- Caught and fixed a real bug: domain-only classification misclassified 27 support calls as external — corrected with title-first logic
- Pipeline is deterministic — identical output on every run

---

## Recommendations

| Priority | Action |
|----------|--------|
| 🔴 Critical | Escalate 8 at-risk accounts — CSM + executive outreach within 5 days |
| 🔴 Critical | Fix LogVault SIEM connector — competitor wins in 3 clicks, we need 6 pages |
| 🟡 High | Ship on-demand compliance reporting — 3 customers blocked on active audits |
| 🟡 High | Sequence Identity gaps into Q3 — customers evaluating alternatives |
| 🟢 Medium | Wire pipeline to live call ingestion — dashboard updates after every call |

---

## Tech Stack

Python · Jupyter Notebook · Matplotlib · Claude API · GitHub Pages · Chart.js

---

*Sony Uppu · AegisCloud Assessment · May 2026*
