# Chemical_Ml

Machine-learning workflow for the electrochemical oxidation of
5-hydroxymethylfurfural (HMF) to 2,5-furandicarboxylic acid (FDCA), adapted from
Li et al., *ACS Sustainable Chem. Eng.* 2025, 13, 13940
([doi:10.1021/acssuschemeng.5c04936](https://doi.org/10.1021/acssuschemeng.5c04936)).

The pipeline turns a literature dataset of 154 electro-oxidation records
(99 publications) into numerical descriptors, classifies reported catalytic
performance with oversampling and grouped cross-validation, explains the models
with SHAP and searches for promising operating conditions with NSGA-II.

## Setup

```bash
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

All scripts read and write paths relative to the repository folder. Figures and
tables from scripts 02–08 are written to `outputs/`.

## Scripts

| Script | What it does |
|---|---|
| `01_preprocess_data.py` | Cleans `Nishith_Summary Table of different cell types.xlsx` and writes `HMF_Cleaned_Full_Dataset.xlsx` and `HMF_ML_Preprocessed.{xlsx,csv}` |
| `02_kmeans.py` | K-means (K = 3–6) on standardized conversion and FDCA yield |
| `03_model_evaluation.py` | Learning curves for DT, SVR, RF and NN regressors on conversion, yield, FE and selectivity |
| `04_kmeans_oversampling.py` | SMOTE/ADASYN performance classes and K-means visualization |
| `05_learning_curves_oversamples.py` | F1 learning curves of oversampled neural-network classifiers |
| `06_shap_analysis.py` | SHAP feature importance for K = 3 and K = 5 |
| `07_shap_extra_figures.py` | SHAP beeswarm and impact-trajectory figures |
| `08_genetic algorithms.py` | Random-forest surrogates + four-objective NSGA-II |
| `09_f1_optimization.py` | Leakage-safe, publication-grouped, nested-CV F1 optimization (see `RESULTS.md`) |

Run them from any directory, for example:

```bash
python 01_preprocess_data.py
python "08_genetic algorithms.py"
python 09_f1_optimization.py all      # modes: stage1 | nested | all
```

## Results and paper

- `RESULTS.md` and `results/` hold the classification results of `09_f1_optimization.py`.
- `paper/` holds the manuscript (`main.tex`, `references.bib`, `figures/`) and the
  compiled `main.pdf`. Rebuild it with `tectonic -X compile paper/main.tex`.
