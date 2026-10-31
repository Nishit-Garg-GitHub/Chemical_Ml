# F1 optimisation results (`09_f1_optimization.py`)

Objective (Li et al., ACS Sustain. Chem. Eng. 2025, 13, 13940): predict the
catalytic performance class of an HMF -> FDCA experiment from catalyst and
reaction descriptors and maximise F1, with SMOTE/ADASYN oversampling.

## Protocol
- Rows: 123 experiments that report a yield. `perf = min(conversion, yield)`;
  when only yield is reported `perf = yield` (yield cannot exceed conversion).
- Inputs: 72 descriptors (preprocessed columns + flags from Catalysts /
  Electrolyte / Cell Type text + ratios). Conversion, yield, FE and
  selectivity are never inputs; the outcome-describing text columns are unused.
- Oversampling only inside training folds; rows of the same paper never
  straddle train/test (StratifiedGroupKFold on paper title).
- Stage 1: 278 pipelines (12+ model families x 5 samplers x feature-count)
  scored with 5 x 5-fold grouped CV. The best of 278 is optimistic.
- Stage 2 (headline): nested CV. The pipeline is chosen from all 278 using
  only the outer-training data, then scored on the held-out fold.

## Class definitions (all reported, none cherry-picked)
| scheme | classes | counts |
|---|---|---|
| K2_high | perf >= 90 vs rest | 34 / 89 |
| K3_paper | perf <50, 50-80, >=80 (paper text) | 9 / 7 / 107 |
| K3_yield | yield <80, 80-95, >=95 | 16 / 38 / 69 |
| K4_yield | yield <50, 50-80, 80-95, >=95 | 9 / 7 / 38 / 69 |
| K5_paper | perf <50, 50-80, 80-90, 90-95, >=95 | 9 / 7 / 18 / 22 / 67 |

## Honest (nested-CV) F1
| scheme | macro-F1 | weighted-F1 | majority-class macro-F1 |
|---|---|---|---|
| K2_high | 0.610 | 0.685 | 0.420 |
| K3_paper | 0.460 | 0.824 | 0.310 |
| K3_yield | 0.445 | 0.516 | 0.240 |
| K4_yield | 0.367 | 0.459 | 0.180 |
| K5_paper | 0.327 | 0.426 | 0.141 |

## Paper's four models, fixed settings (no tuning), SMOTE/ADASYN, macro-F1
| scheme | DT | SVM | RF | NN |
|---|---|---|---|---|
| K2_high | 0.611 | 0.543 | 0.627 | 0.597 |
| K3_paper | 0.516 | 0.305 | 0.473 | 0.526 |
| K3_yield | 0.422 | 0.371 | 0.477 | 0.442 |
| K4_yield | 0.420 | 0.239 | 0.436 | 0.486 |
| K5_paper | 0.282 | 0.236 | 0.398 | 0.372 |

Best single pipeline in the stage-1 screen (optimistic, selected on the same
folds): K2_high 0.763 (logistic regression + ADASYN, top-20 features),
K3_paper 0.581 macro / 0.868 weighted (ExtraTrees + random oversampling).

## Reading the numbers
- The paper reports F1 about 0.8 on 379 thermocatalytic records with pressure,
  time and alkali descriptors. This dataset has 123 usable electro-oxidation
  rows, about 87% of them in the top class, and no pressure/time. Class
  definitions differ, so the figures are not comparable with the paper's.
- Picking the best of 278 pipelines loses 0.15 macro-F1 on K2_high
  (0.763 -> 0.610) once selection is nested. Use the nested column.
- More classes means lower F1: classes of 7-9 rows are not learnable from
  descriptors alone here.
