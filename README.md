# LLM Bias Audit — World Happiness Report

Do large language models hold a distorted picture of how happy different countries are?

This project audits GPT model outputs against the real **World Happiness Report** (2011–2024), measures where the model's perception diverges from ground truth, and tests whether better prompting closes the gap. Results are explorable through an interactive dashboard.

Built for the *Data and Society* seminar at Saarland University.

**Authors:** Akash Chavan, Sina Elahi Manesh, Yassal Arif

---

## The question

LLMs are increasingly used as stand-ins for human judgement. But their view of the world is inherited from training data — which over-represents some regions and under-represents others.

So: **if you ask an LLM how happy people in a given country are, is it systematically wrong — and is it wrong in a patterned way?**

---

## Method

**1. Persona-based surveying.** The model is asked to answer as one of **20 personas** across **150+ countries**, rating life satisfaction on the Cantril ladder (the same scale the World Happiness Report uses).

**2. Four prompting approaches**, run and compared head-to-head:

| Approach | Description |
|---|---|
| Baseline | 7-question survey, detailed persona prompt |
| Few-shot | Same survey, plus calibration examples |
| Single-question (Gallup) | One question only, mirroring the original Gallup item |
| Structured personas | Single question + structured persona fields |

**3. Hyperparameter tuning.** Sweeps over sampling temperature (0.3 / 0.6 / 0.9 / 1.2) and chain-of-thought on/off, scored against ground truth.

**4. Prompt tuning.** Seven prompt variants tested — baseline, simplified, concise persona, JSON output, few-shot, explicit context, step-by-step reasoning.

**5. Statistical analysis.** LLM predictions vs. real scores, tested for significant bias:
- t-tests and ANOVA across groupings
- Weighted linear regression for driver importance
- Pearson / Spearman correlation
- Groupings: continent, region, development level, income tier

---

## What the dashboard shows

An interactive **Dash** app with six tabs:

- **World Map** — choropleth of happiness scores by year
- **Driver Analysis** — which factors (GDP, social support, freedom, …) actually predict happiness, by region
- **Trends** — how happiness moved 2011–2023, top movers, volatility
- **Group Comparisons** — regional and income-level gaps with significance tests
- **LLM Audit** — bias by group, key findings, and statistical significance, filterable by prompting approach
- **Overview** — summary statistics

---

## Repository structure

```
.
├── app.py                  # Dash application entry point
├── data_loader.py          # Loading and preprocessing of WHR data
├── driver_analysis.py      # Weighted regression on happiness drivers
├── trend_analysis.py       # Temporal trends
├── group_comparison.py     # Regional / income comparisons + tests
├── llm_audit.py            # Audit integration for the dashboard
├── dataset.xlsx            # World Happiness Report panel (2011–2024)
│
└── llm_audit_data/
    ├── personas.py                     # 20 persona definitions
    ├── analyzer.py                     # Statistical utilities
    ├── analyze_llm_vs_real.py          # LLM vs ground truth comparison
    ├── analyze_bias.py                 # Bias analysis by group
    ├── hyperparameter_tuning.py        # Temperature / CoT sweep
    ├── prompt_tuning.py                # Prompt strategy comparison
    │
    ├── initial_approach/               # Baseline 7-question survey
    ├── few_shot_approach/              # Few-shot calibrated
    ├── single_question_gallup_approach/
    └── structured_personas_approach/
```

---

## Setup

```bash
python3 -m venv venv
source venv/bin/activate          # Windows: venv\Scripts\activate
pip install -r requirements.txt
```

**API key** — the audit pipelines call the OpenAI API:

```bash
cp .env.example .env
# then add: OPENAI_API_KEY=sk-...
```

The `.env` file is gitignored and never committed.

---

## Running it

**Dashboard:**

```bash
./run.sh          # or: python app.py
```

Opens at `http://localhost:8050`. Works out of the box — if no audit results are present, it falls back to mock data.

**Audit pipelines:**

```bash
cd llm_audit_data
python initial_approach/run_audit.py       # baseline
python few_shot_approach/run_audit_improved.py
python hyperparameter_tuning.py            # temperature / CoT sweep
python prompt_tuning.py                    # prompt strategy comparison
python analyze_bias.py                     # bias by group
```

Results are written to `results/` as CSVs, with statistically significant findings (p < 0.05) extracted automatically.

**Full analysis notebook:** [`World_Happiness_Analysis_and_LLM_Audit.ipynb`](World_Happiness_Analysis_and_LLM_Audit.ipynb)

---

## Tech stack

`Python` · `OpenAI API` · `Pandas` · `NumPy` · `SciPy` · `Dash` · `Plotly` · `Jupyter`

---
