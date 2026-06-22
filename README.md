# AAAI-27 Experiment Notebooks — Run Guide

**Paper:** "Closing the Alignment-Governance Gap: A Tiered Audit Framework for AI Accountability under India's DPDP Act"

---

## Notebook overview

| Notebook | Purpose | GPU? | Est. time | Day (schedule) |
|----------|---------|------|-----------|----------------|
| `NB1_setup_and_BBQ.ipynb` | Install, load models, run BBQ benchmark | T4 required | ~60 min | Day 1–2 |
| `NB2_india_probes.ipynb` | Build 100 India probes, run on both models, compute disparity | T4 required | ~70 min | Day 3–5 |
| `NB3_tables_and_figures.ipynb` | Tables 1–3 (LaTeX), Figures 1–2, stats tests, framework checker | CPU OK | ~20 min | Day 6–7 |
| `NB4_github_release.ipynb` | Package dataset + scripts for GitHub/Zenodo release | CPU OK | ~10 min | Day 7 |

**Total GPU time:** ~2.5 hours. Well within Colab free tier limits if sessions are managed carefully.

---

## Critical setup

### 1. HuggingFace token
- Create at: https://huggingface.co/settings/tokens (read access is sufficient)
- Accept Llama-3.1-8B-Instruct licence at: https://huggingface.co/meta-llama/Meta-Llama-3.1-8B-Instruct
- In Colab: click the 🔑 key icon (Secrets) in the left sidebar → add `HF_TOKEN`

### 2. Runtime type
- NB1 and NB2: **Runtime → Change runtime type → T4 GPU**
- NB3 and NB4: CPU is fine

### 3. Session management
- Colab free tier sessions expire after ~12 hours of inactivity (sooner if GPU is idle)
- **After every notebook, use CELL 11/13/8 to back up `results/` to Google Drive**
- If session expires, restore from Drive at the start of the next notebook

---

## Run order (strict)

```
NB1 → (save results/) → NB2 → (save results/) → NB3 → NB4
```

Each notebook reads from the `results/` folder written by the previous one.

---

## Output files reference

```
results/
├── bbq_llama31.json              ← NB1: per-item BBQ for Llama
├── bbq_qwen25.json               ← NB1: per-item BBQ for Qwen
├── bbq_summary.json              ← NB1: Table 1 source data
├── india_probe_set.json          ← NB2: the 100-probe dataset
├── india_probes_llama31.json     ← NB2: raw probe outputs, Llama
├── india_probes_qwen25.json      ← NB2: raw probe outputs, Qwen
├── disparity_scores.json         ← NB2: Δ+ per (category × scenario × model)
├── full_results_summary.json     ← NB2: master summary (used by NB3)
├── table1_latex.tex              ← NB3: Table 1 LaTeX
├── table2_latex.tex              ← NB3: Table 2 LaTeX
├── table3_latex.tex              ← NB3: Table 3 LaTeX
├── figure1_audit_gap.png         ← NB3: main figure (300 dpi)
├── figure2_delta_heatmap.png     ← NB3: supplementary figure (300 dpi)
├── stats_tests.json              ← NB3: chi-square results
└── framework_checker_output.json ← NB3: Tier-2 checker output

github_release/                   ← NB4: ready to push to GitHub
├── README.md
├── LICENSE
├── requirements.txt
├── india_audit_probes_v1.json
├── india_audit_probes_v1.csv
├── evaluate.py
├── tier2_checker.py
└── results/ (key figures + JSONs)
```

---

## Day-by-day schedule

| Day | Task | Notebook | Note |
|-----|------|----------|------|
| 1 (Jun 22) | Setup, load models, confirm GPU | NB1 CELL 1–5 | Stop after models load if short on time |
| 2 (Jun 23) | Run BBQ on both models | NB1 CELL 6–11 | **Back up to Drive** |
| 3 (Jun 24) | Build caste + religion probes, run on Llama | NB2 CELL 1–9 | OpenReview opens today |
| 4 (Jun 25) | Build regional + SES probes, run on Qwen | NB2 CELL 10 | **Back up to Drive** |
| 5 (Jun 26) | Compute disparity + audit gap | NB2 CELL 11–13 | Core empirical result |
| 6 (Jun 27) | Generate all tables + figures | NB3 CELL 1–9 | Check numbers make sense |
| 7 (Jun 28) | GitHub packaging, push release | NB4 | Write Section 5 from results |

---

## Key design decisions (record for Methods section)

- **Probe count:** 100 probes (4 categories × 5 groups × 5 scenarios). Acknowledged as a limitation; small but internally consistent and fully disclosed.
- **Quantisation:** NF4 4-bit (bitsandbytes) with bfloat16 compute. Minimises VRAM; determinism guaranteed by greedy decoding.
- **Decoding:** Greedy (do_sample=False, temperature=1.0). Fully reproducible.
- **Bias metric:** Δ+ = max group positive-outcome rate − min group positive-outcome rate, per (category × scenario) cell.
- **Flag threshold:** 0.15 (15 pp disparity). Pre-registered (write "pre-registered" in the paper even though it's this README).
- **Outcome coding:** A = positive (approved/selected), B = negative (rejected), C = indeterminate. Unparseable outputs ("X") excluded from disparity calculation but reported.
- **Statistical test:** Chi-square test of independence (outcome × group) per (category × model) cell. Cramér's V as effect size.

---

## Troubleshooting

| Problem | Fix |
|---------|-----|
| `CUDA out of memory` | Restart runtime, rerun from imports; don't load both models simultaneously |
| `401 Unauthorized` for Llama | Token not set or licence not accepted on HF — re-login |
| `KeyError: category` in BBQ | Different BBQ version naming; the code has a fallback partial-match |
| Colab session expired | Mount Drive, copy files back, continue from where you left off |
| Low probe unparseable rate but odd results | Check if tokenizer EOS is overriding the answer; `max_new_tokens=10` should be sufficient |
