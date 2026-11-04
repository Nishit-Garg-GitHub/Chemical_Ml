# Chemical_Ml

Machine-learning workflow for the electrochemical oxidation of
5-hydroxymethylfurfural (HMF) to 2,5-furandicarboxylic acid (FDCA), adapted from
Li et al., *ACS Sustainable Chem. Eng.* 2025, 13, 13940
([doi:10.1021/acssuschemeng.5c04936](https://doi.org/10.1021/acssuschemeng.5c04936)).

A literature dataset of 154 electro-oxidation records from 99 publications is
turned into numerical descriptors. The workflow then classifies reported
catalytic performance, explains the models with SHAP and searches for promising
operating conditions with a four-objective NSGA-II genetic algorithm.

**Manuscript:** [`paper/main.pdf`](paper/main.pdf)

## Key results

Performance classes are cut from reported HMF conversion and FDCA yield
(`P = min(conversion, yield)`, 123 usable records). All scores are nested
cross-validation over five outer seeds; oversampling and every tuning step see
training folds only.

| Task | Publication-grouped split | Record-level split (reference-study protocol) |
|---|---|---|
| 2 classes (P ≥ 90 % vs rest) | macro-F1 **0.687** (0.728 with catalyst-name descriptors) | macro-F1 **0.774**, weighted-F1 **0.817** |
| 3 classes (P < 50, 50–80, ≥ 80 %) | macro-F1 0.517, weighted-F1 **0.834** | macro-F1 0.569, weighted-F1 **0.842** |

- Publication-grouped splits never put records of one paper in both training and
  test folds; they measure transfer to unseen studies and are the headline.
- Support-vector machines and neural networks beat random forests and decision
  trees under identical tuning (`results/family_comparison.csv`).
- Macro-F1 is capped by the two minority classes (9 and 7 records).
- NSGA-II best-compromise candidate: Ni–Cu, carbon-supported, nanostructured LDH
  anode, alkaline OH⁻ electrolyte, TEMPO, flow cell, 9.4 °C, 1.44 V vs RHE
  (predicted conversion 99.3 %, FDCA yield 96.3 %). The surrogate R² is
  negative on its validation split, so candidates are hypotheses for
  experiments.

Full tables are in [`RESULTS.md`](RESULTS.md).

## Repository layout

```
01_preprocess_data.py … 08_genetic algorithms.py   original workflow
09_f1_optimization.py      278-pipeline screen, nested CV (reference protocol)
10_f1_improved.py          final classifier pool (tuned decision rules, ensembles)
11_family_comparison.py    DT vs RF vs SVM vs NN under identical tuning
Nishith_Summary Table of different cell types.xlsx   raw literature table
HMF_Cleaned_Full_Dataset.xlsx, HMF_ML_Preprocessed.{xlsx,csv}   outputs of 01
results/                   classification results (CSV)
paper/                     manuscript source, figures and compiled PDF
outputs/                   figures and tables from 02–08 (created on run, not tracked)
```

## Setup

```bash
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

All scripts use paths relative to the repository folder, so they run from any
working directory.

## Scripts

| Script | What it does |
|---|---|
| `01_preprocess_data.py` | Cleans the raw literature table and writes `HMF_Cleaned_Full_Dataset.xlsx` and `HMF_ML_Preprocessed.{xlsx,csv}` |
| `02_kmeans.py` | K-means (K = 3–6) on standardized conversion and FDCA yield |
| `03_model_evaluation.py` | Learning curves for DT, SVR, RF and NN regressors on conversion, yield, FE and selectivity |
| `04_kmeans_oversampling.py` | SMOTE/ADASYN performance classes and K-means visualization |
| `05_learning_curves_oversamples.py` | F1 learning curves of oversampled neural-network classifiers |
| `06_shap_analysis.py` | SHAP feature importance for K = 3 and K = 5 |
| `07_shap_extra_figures.py` | SHAP beeswarm and impact-trajectory figures |
| `08_genetic algorithms.py` | Random-forest surrogates + four-objective NSGA-II |
| `09_f1_optimization.py` | Leakage-safe, publication-grouped, nested-CV screen of 278 pipelines (reference protocol) |
| `10_f1_improved.py` | Final classifier pool: macro-F1-tuned decision rules, regress-then-bin models and soft-vote ensembles |
| `11_family_comparison.py` | Decision tree, random forest, SVM and neural network under the same nested tuning |

## Reproducing the results

```bash
python 01_preprocess_data.py
python "08_genetic algorithms.py"
python 09_f1_optimization.py all                 # modes: stage1 | nested | all
python 10_f1_improved.py                         # publication-grouped, 73 descriptors
python 10_f1_improved.py --text                  # + catalyst-name descriptors
python 10_f1_improved.py --record --text         # record-level splits
python 11_family_comparison.py
```

`10_f1_improved.py` also accepts scheme names (`K2_high K3_paper K3_yield
K4_yield K5_paper`, and `KM3`–`KM5` for K-means classes) and `--rhe` (potential
reference flag). Each run writes `results/improved_nested_results*.csv`.

To rebuild the paper after changing results, update the metric block at the top
of `paper/main.tex` and run:

```bash
tectonic -X compile paper/main.tex
```

## Data notes

- Records from the same publication are correlated; use grouped splits when
  claiming generalization to new studies.
- The raw `Catalyst_Pt` flag mostly marks Pt counter electrodes (71 of 74
  records); `10_f1_improved.py` uses an anode-only Pt flag.
- Pressure and reaction time are not reported consistently in the
  electrochemical literature and are not imputed.
