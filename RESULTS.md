# F1 optimisation results

## Current best: `10_f1_improved.py`

Same data, class schemes, publication-grouped splits and nested protocol as
`09_f1_optimization.py` (below), plus decision rules tuned for macro-F1 inside
the inner folds (per-class probability weights for classifiers, cut-offs for
regress-then-bin models), a soft-vote ensemble of the three best classifiers,
anode-only Pt and reporting flags (73 features, 44 candidates). Outer test
folds never influence model choice, weights or cut-offs.

Nested CV, five outer seeds (`results/improved_nested_results.csv`):

| scheme | macro-F1 | weighted-F1 | majority-class macro-F1 |
|---|---|---|---|
| K2_high | 0.687 +- 0.042 | 0.741 | 0.420 |
| K3_paper | 0.517 +- 0.040 | 0.834 | 0.310 |
| K3_yield | 0.459 +- 0.027 | 0.503 | 0.240 |
| K4_yield | 0.398 +- 0.038 | 0.491 | 0.180 |
| K5_paper | 0.355 +- 0.036 | 0.466 | 0.141 |

Same outer splits as `09` (seeds 100-101), macro-F1: K2_high 0.610 -> 0.693,
K3_paper 0.460 -> 0.502, K3_yield 0.445 -> 0.482, K4_yield 0.367 -> 0.388,
K5_paper 0.327 -> 0.335. Weighted-F1 also rises on every scheme. The
seed-to-seed s.d. (0.03-0.04) is as large as the K4/K5 gains.

Most-selected candidates over 125 outer folds: soft-vote ensemble 40, neural
network 21, class-weighted extra trees 19, class-weighted logistic regression
12, regress-then-bin models 16.

### Split protocol and feature set

Macro-F1 (weighted-F1), five outer seeds. Base: 73 descriptors. `+text`: 12
character-n-gram components of the anode catalyst name (85 descriptors).
Record-level splits follow the reference study and allow records of one paper
on both sides of a split.

| scheme | grouped, base | grouped, +text | record, base | record, +text |
|---|---|---|---|---|
| K2_high | 0.687 (0.741) | 0.728 (0.779) | 0.767 (0.815) | 0.774 (0.817) |
| K3_paper | 0.517 (0.834) | 0.462 (0.807) | 0.554 (0.833) | 0.569 (0.842) |
| K3_yield | 0.459 (0.503) | 0.416 (0.477) | 0.492 (0.546) | 0.512 (0.568) |
| K4_yield | 0.398 (0.491) | 0.388 (0.497) | 0.447 (0.552) | 0.472 (0.583) |
| K5_paper | 0.355 (0.466) | 0.346 (0.469) | 0.433 (0.535) | 0.394 (0.499) |

Files: `improved_nested_results.csv`, `_text.csv`, `_record.csv`,
`_record_text.csv`. The `*_K2_high_K3_paper.csv` files are exploratory runs
on two schemes (including `--rhe`).

K-means classes (`KM3`-`KM5`) are not learnable here: K = 3 gives clusters of
3 / 9 / 90 records and K >= 4 gives single-record clusters.

### Model families under identical tuning (`11_family_comparison.py`)

Macro-F1 (weighted-F1), five outer seeds, +text descriptors.

| family | grouped K2 | grouped K3_paper | record K2 | record K3_paper |
|---|---|---|---|---|
| Decision tree | 0.634 (0.707) | 0.431 (0.786) | 0.674 (0.735) | 0.488 (0.814) |
| Random forest | 0.703 (0.757) | 0.478 (0.823) | 0.743 (0.792) | 0.556 (0.839) |
| SVM | **0.716** (0.763) | **0.503** (0.824) | **0.756** (**0.801**) | **0.602** (0.843) |
| Neural network | 0.714 (**0.769**) | 0.481 (**0.828**) | 0.737 (0.786) | 0.585 (**0.845**) |

---

## Reference protocol: `09_f1_optimization.py`

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
