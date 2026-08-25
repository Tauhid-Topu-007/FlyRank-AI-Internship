# Refresh Scoring Model — FlyRank ML Capstone

**Predicting content page decline with Random Forest**

A machine learning capstone project that ranks content pages by their likelihood of decline, helping content teams prioritize which pages to review and refresh first.

---

## What It Does and For Whom

### What it does

The system uses 90-day search and engagement signals—such as impressions, clicks, average position, CTR, content age, and engagement—to score and rank pages by decline risk. It produces an actionable review queue with reason codes explaining why a page was flagged.

### For whom

Content reviewers and SEO/content teams at SaaS companies that manage large numbers of pages but have limited capacity for manual review.

### The one action

A reviewer opens the ranked queue, starts from the highest-priority page, and reviews or refreshes pages in priority order.

---

## Key Results

| Method | Precision@50 | Correct out of 50 | Improvement |
|---|---:|---:|---:|
| Baseline (Hand-coded Rule) | 0.240 | 12 | — |
| Logistic Regression | 0.400 | 20 | 1.67× |
| Decision Tree | 0.540 | 27 | 2.25× |
| **Random Forest** | **0.740** | **37** | **3.08×** |

> **Result:** The Random Forest identifies approximately 3× more true problems in the top 50 pages than the hand-coded baseline.

---

## Architecture & Data Flow

```text
┌─────────────────────────────────────────────────────────────────┐
│                         DATA FLOW                               │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌──────────────────────┐                                       │
│  │ Starter Dataset      │                                       │
│  │ 30,000 pages         │                                       │
│  │ 44 features          │                                       │
│  └──────────┬───────────┘                                       │
│             │                                                   │
│             ▼                                                   │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │ Feature Engineering                                      │   │
│  │ • impressions_90d, clicks_90d, ctr, avg_position        │   │
│  │ • content_age_days, engagement_rate, scroll_rate        │   │
│  │ • position_tier, age_tier, impression_tier              │   │
│  └─────────────────────────┬────────────────────────────────┘   │
│                            │                                    │
│                            ▼                                    │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │ Client-Holdout Split                                    │   │
│  │ • Train: 70% clients                                     │   │
│  │ • Test: 30% clients                                      │   │
│  └─────────────────────────┬────────────────────────────────┘   │
│                            │                                    │
│                            ▼                                    │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │ Random Forest Classifier                                 │   │
│  │ • n_estimators: 100                                      │   │
│  │ • max_depth: 15                                          │   │
│  │ • min_samples_split: 20                                  │   │
│  └─────────────────────────┬────────────────────────────────┘   │
│                            │                                    │
│                            ▼                                    │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │ Ranked Queue + Reason Codes                              │   │
│  │ • REVIEW_NOW: stale + high volume                       │   │
│  │ • REVIEW_SOON: high volume or position ≤ 5              │   │
│  │ • MONITOR: no specific risk identified                   │   │
│  └──────────────────────────────────────────────────────────┘   │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## Setup

### Prerequisites

- Python 3.8+
- Git
- Jupyter Notebook or Google Colab (optional, for notebooks)

### 1. Clone the Repository

```bash
git clone https://github.com/Tauhid-Topu-007/FlyRank-AI-Internship.git
cd FlyRank-AI-Internship
```

### 2. Install Dependencies

```bash
pip install -r requirements.txt
```

### 3. Run the Pipeline

```bash
python scripts/run_all.py
```

The pipeline prepares the features, generates the baseline score, trains the models, evaluates the results, exports the ranked queue, and generates the report artifacts supported by the repository.

### 4. Explore the Notebooks

Open the capstone and weekly notebooks under `work/notebooks/` in Jupyter or Google Colab to inspect the complete analysis, validation, and decision-making workflow.

---

## Usage Examples

### Run the Full Pipeline

```bash
python scripts/run_all.py
```

The full workflow covers:

1. Feature preparation
2. Baseline scoring
3. Model training
4. Evaluation
5. Ranked-queue generation
6. Report/export generation

### Run a Notebook Individually

```text
1. Open the required notebook under work/notebooks/.
2. Run the cells from top to bottom.
3. Inspect the generated tables, metrics, charts, and outputs.
```

### Load the Ranked Queue

```python
import pandas as pd

df = pd.read_csv("work/outputs/action_playbook_queue.csv")
print(df.head(10))
```

---

## Evaluation Results

### Precision@50 Comparison

| Method | Precision@50 | Correct out of 50 |
|---|---:|---:|
| Baseline (Hand-coded Rule) | 0.240 | 12 |
| Logistic Regression | 0.400 | 20 |
| Decision Tree | 0.540 | 27 |
| **Random Forest** | **0.740** | **37** |

### Why Precision@50?

Precision@50 measures how many of the first 50 pages in the ranked queue are genuinely relevant problem pages. This metric matches the operational goal of the project: reviewers have limited time, so the most important question is whether the top of the queue contains useful pages to review.

### Top Features

1. **`impressions_90d`** — search volume is the strongest signal
2. **`avg_position`** — ranking position is a leading performance indicator
3. **`content_age_days`** — older pages can carry greater refresh risk
4. **`ctr`** — click-through rate provides an engagement/search-result signal
5. **`engagement_rate`** — user interaction provides an additional quality signal

### Confusion Matrix

The Random Forest evaluation used the following confusion-matrix summary:

| | Predicted No | Predicted Yes |
|---|---:|---:|
| **Actual No** | 4,200 | 1,800 |
| **Actual Yes** | 1,500 | 4,500 |

---

## Action Playbook

The model is designed to support a simple operational workflow rather than only produce a classification score.

| Priority | Meaning | Example trigger |
|---|---|---|
| **REVIEW_NOW** | Highest priority pages requiring immediate review | Stale content + high search volume |
| **REVIEW_SOON** | Pages worth reviewing in the near term | High volume or strong ranking-risk signal |
| **MONITOR** | No specific high-priority risk identified | Continue monitoring performance |

This turns model output into a reviewer-friendly queue that can support content refresh decisions at scale.

---

## Limitations

| Limitation | Explanation |
|---|---|
| **Proxy label** | `trend_direction` is a current-window proxy, not a direct future-decline label. |
| **Starter dataset only** | Results are based on the available 30,000-row starter dataset; performance may differ on a larger production warehouse. |
| **No seasonality** | The model cannot reliably distinguish seasonal dips from genuine long-term decline. |
| **Client variation** | Performance can vary across different clients and content portfolios. |
| **No content understanding** | The model uses structured search/engagement signals and does not directly read or semantically understand page content. |

These limitations mean the model should be treated as **decision support**, not as a guarantee of future Google ranking behavior or page performance.

---

## Where AI Helped

This project was built with **Claude** as an AI development assistant.

AI assistance was used for:

- **Code generation:** initial scaffolding for feature engineering, model training, and evaluation.
- **Debugging:** identifying and fixing implementation issues such as categorical-variable handling and missing-value processing.
- **Documentation:** structuring the README and research documentation.
- **Concept tutoring:** explaining concepts such as client-holdout validation, Precision@50, and feature importance.

### What I Checked Myself

- All code was reviewed and tested by me.
- The data contract and feature selection were my decisions.
- The validation design was my choice.
- The results were interpreted and verified by me.
- Claims are based on the measured project results and documented limitations.

> **Principle:** I don't ship code I don't understand. Every important part of this repository is something I can explain, evaluate, and defend.

---

## Repository Structure

```text
FlyRank-AI-Internship/
├── README.md
├── data/
│   └── raw/
│       └── content_refresh_anonymized.csv
├── work/
│   ├── notebooks/
│   │   ├── capstone.ipynb
│   │   ├── w02_ml_task_framing.ipynb
│   │   ├── w03_data_contract.ipynb
│   │   ├── w04_baseline_score.ipynb
│   │   ├── w05_model.ipynb
│   │   ├── w06_validation_audit.ipynb
│   │   ├── w07_action_playbook.ipynb
│   │   └── w09_hardening.ipynb
│   └── outputs/
│       ├── action_playbook_queue.csv
│       └── model_results.json
├── scripts/
│   ├── 01_prepare_features.py
│   ├── 02_baseline_score.py
│   ├── 03_train_model.py
│   └── run_all.py
├── requirements.txt
└── LICENSE
```

> File names can vary slightly depending on the current repository version. Use the actual files present in `work/`, `scripts/`, and `outputs/` as the source of truth.

---

## Internship Workflow

The project follows a progression from problem framing to an actionable ML system:

```text
Problem Framing
      ↓
Data Contract
      ↓
Feature Preparation
      ↓
Baseline Scoring
      ↓
Model Training
      ↓
Client-Holdout Validation
      ↓
Action Playbook
      ↓
Hardening / Audit
      ↓
Capstone
      ↓
Research Documentation
```

This workflow emphasizes that the capstone is not only about selecting a model. It is about connecting the business problem, data, validation strategy, evaluation metric, and final reviewer action into one reproducible ML workflow.

---

## Demo Video Script

A 3–5 minute demonstration can follow this structure:

| Time | Section | What to Show |
|---|---|---|
| 0:00–0:30 | Introduction | Show the research paper and briefly explain the project goal. |
| 0:30–1:00 | Problem | Explain why content teams need a prioritized review queue. |
| 1:00–2:00 | Method | Walk through data → features → model → client-holdout validation. |
| 2:00–3:00 | Results | Show the Precision@50 comparison and explain the Random Forest result. |
| 3:00–3:30 | Recommendations | Show the ranked queue and `REVIEW_NOW`, `REVIEW_SOON`, and `MONITOR` categories. |
| 3:30–4:00 | Limitation | Explain that the target is a proxy and does not directly predict future decline. |
| 4:00–4:30 | AI Transparency | Explain how Claude assisted with coding, debugging, documentation, and learning. |
| 4:30–5:00 | Closing | Show the repository and explain how the workflow can be reproduced. |

---

## Final Package

The final submission can include:

| Component | Location / Link |
|---|---|
| Week 2 — ML Task Framing | `work/notebooks/w02_ml_task_framing.ipynb` |
| Week 3 — Data Contract | `work/notebooks/w03_data_contract.ipynb` |
| Week 4 — Baseline Score | `work/notebooks/w04_baseline_score.ipynb` |
| Week 5 — Model Training | `work/notebooks/w05_model.ipynb` |
| Week 6 — Validation Audit | `work/notebooks/w06_validation_audit.ipynb` |
| Week 7 — Action Playbook | `work/notebooks/w07_action_playbook.ipynb` |
| Week 9 — Hardening | `work/notebooks/w09_hardening.ipynb` |
| Week 10 — Capstone | `work/notebooks/capstone.ipynb` |
| Final — README | `README.md` |
| Final — Research Paper | Add deployed paper URL when available |
| Final — Demo Video | Add video URL when available |

---

## Links

- **GitHub Repository:** [FlyRank-AI-Internship](https://github.com/Tauhid-Topu-007/FlyRank-AI-Internship)
- **Portfolio:** [Tauhid Topu Portfolio](https://portfolio-frontend-rust-six.vercel.app/)
- **Research Paper:** Add the deployed URL when available
- **Demo Video:** Add the published video URL when available

---

## License

This project follows the repository's included `LICENSE` file. The dataset and its usage restrictions should be treated according to the project's data-use documentation and internship requirements.

---

## Author

**Tauhid Topu**  
CSE Student · Machine Learning & Data Science Enthusiast

---

© 2026 Tauhid Topu · Built as part of the FlyRank ML Internship capstone.
