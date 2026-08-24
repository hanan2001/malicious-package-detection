# Metadata Compressibility and Evaluation Bias in Malicious Package Detection for NPM and PyPI

Replication package for the manuscript submitted to *Journal of Cybersecurity and Privacy*
(MDPI), manuscript ID **jcp-4466772**.

This repository contains the frozen dataset, the complete pipeline, and every table and figure
reported in the manuscript and its Supplementary Materials. All results are reproducible from
the frozen dataset without re-contacting the package registries.

---

## What this study measures

The paper is an empirical characterisation, not a new detection method. It reports three
measurements:

1. **Metadata compressibility.** After the a priori exclusion of 416 attributes that are
   constant within one ecosystem, 18 of the remaining 40 attain a validation AU-PR of 0.9955
   against 0.9972 for the full admissible set. Within NPM, where no attribute is
   ecosystem-constant by construction, the reduction is from the full 456 point-in-time
   attributes to 18 and is entirely data-driven.
2. **Evaluation bias.** A naive and a group-aware evaluation protocol differ by 5.7 accuracy
   points on the same package-version inventory. The contribution of each control is reported
   separately in `tables/tableS_leakage_block1_grouping.csv` and
   `tables/tableS_leakage_block2_fixed_test.csv`.
3. **Operational utility.** At an assumed registry prevalence of 0.1 %, the positive predictive
   value is 4.3 % and recall at the selected threshold is 0.537.

The six-stage selection pipeline is the *instrument* of these measurements, not the
contribution: at a matched budget it ranks fourth of five selectors, 0.0012 AU-PR below Random
Forest importance selection (`tables/table07_matched_budget.csv`).

---

## Repository layout

```
.
├── README.md
├── LICENSE
├── requirements.txt
├── pipeline.ipynb     # pipeline, notebook form
├── pipeline.py        # same pipeline, script form
├── data/
│   ├── dataset_versions_point_in_time.csv   # frozen dataset, 3,330 × 472
│   └── partition_assignment.csv             # package version → group_id → split
├── results/
│   ├── revision2_results.json        # every number reported in the paper
│   ├── hyperparameters.json          # canonical fit
│   ├── feature_inventory.csv         # 464 attributes with family and cardinality
│   ├── shap_npm.csv                  # SHAP ranking, primary NPM analysis
│   ├── cumulative_stage_curve.csv
│   ├── leave_one_stage_out.csv
│   ├── stage_ordering_before_budget.csv
│   ├── stage_ordering_after_budget.csv
│   └── evasion_curve.csv
├── tables/                           # one CSV per table in the paper
└── figures/                          # PNG figures and their underlying data
```

---

## The frozen dataset

`data/dataset_versions_point_in_time.csv` contains **3,330 package versions × 472 columns**:
8 metadata columns (`package`, `version`, `ecosystem`, `t_p`, `maintainer`, `label`, `canon`,
`group_id`) and 464 extracted attributes.

Every attribute is reconstructed **at the publication timestamp of the version** (`t_p`), not
at collection time. The 8 attributes prefixed `cur_` describe the current registry state and
are excluded from all point-in-time analyses, leaving 456 candidates.

| | Versions | Malicious | Presumed benign | Composite groups |
|---|---|---|---|---|
| NPM | 2,505 | 2,092 | 413 | 767 |
| PyPI | 825 | 79 | 746 | 723 |
| **Total** | **3,330** | **2,171** | **1,159** | **1,490** |

The benign class is **presumed benign**: the absence of an advisory record is not evidence that
a package is not malicious.

### Composite groups

`group_id` is the union of three relations, computed by union-find:

- package identity, after canonicalisation (lowercase, alphanumeric only);
- maintainer identity;
- name-similarity family: two canonicalised names *a* and *b* are grouped when
  `1 − levenshtein(a,b) / max(|a|,|b|) ≥ 0.85`.

The 3,330 versions belong to 1,814 packages and 1,490 groups. The largest group holds 137
versions. All versions of a group reside in a single partition, which is why the effective
sample size for inference is the number of groups, not the number of versions.

### Partitions

`data/partition_assignment.csv` fixes the split used in the paper:

| Split | Versions | Groups |
|---|---|---|
| train | 1,713 | 893 |
| val | 649 | 224 |
| test | 968 | 373 |

The test partition contains 691 malicious and 277 presumed-benign versions; the 277 benign
versions belong to 253 independent groups. The largest test group holds 115 versions, 11.9 % of
the partition.

---

## Reproducing the results

```bash
git clone <repository-url>
cd <repository>
pip install -r requirements.txt
python pipeline.py
```

The script looks for `data/dataset_versions_point_in_time.csv` and
`data/partition_assignment.csv`. When both are present it loads them exclusively and skips
collection, temporal windowing and grouping, which are already frozen in those files.

**Collection is disabled by default.** If the frozen files are missing, the script stops with
an explanatory message rather than silently re-collecting. A fresh collection can be enabled
with `CFG['allow_collection'] = True`, but it will produce a **different sample**: advisory
records are added continuously and malicious packages are withdrawn from registries after
detection, so a later collection cannot reproduce the figures in the paper. The frozen dataset
is the reference.

Runtime on a standard Colab CPU instance with the frozen dataset: **about 2 h 45**, dominated
by the 200 grouped bootstrap resamples of the selection-stability analysis (`n_stability_runs`).
Setting it to 50 reduces the total to roughly 1 h 30 and changes the mean Jaccard index in the
third decimal.

The script ends with a consistency assertion that halts execution if the locked test evaluation
and the model-performance table disagree. If it passes, every table in the paper derives from
the same canonical model fit.

---

## Canonical fit

All figures in the model-performance, matched-budget, family-ablation and stage-ablation tables
derive from a single fit, exported to `results/hyperparameters.json`:

- **CatBoost**: `depth=6`, `learning_rate=0.05`, `iterations=500`, `l2_leaf_reg=1`,
  `auto_class_weights="Balanced"`, `thread_count=1`, `random_state=42`
- **Random Forest**: `n_estimators=100`, `max_depth=10`, `min_samples_split=2`,
  `max_features="sqrt"`, `class_weight="balanced"`, `random_state=42`
- **Meta-learner**: `LogisticRegression(C=1.0, penalty="l2", max_iter=1000)` on out-of-fold
  probabilities generated under `StratifiedGroupKFold(5)` on the same `group_id`

Hyperparameters are re-tuned inside the primary NPM analysis and are reported separately in
`results/revision2_results.json` under `primary_npm.hyperparameters`.

`thread_count=1` is set deliberately: an earlier version of this pipeline fitted the ablation
tables with CatBoost library defaults while the main table used the grid-searched values, and
that discrepancy — not any stochastic component — accounted for a 0.0006 AU-PR difference
between two tables reporting the same configuration.

---

## Mapping: paper → files

### Main manuscript

| Element | File |
|---|---|
| Table 1 — dataset composition | `data/dataset_versions_point_in_time.csv` |
| Table 3 — selection phases | `tables/table03_selection_phases.csv` |
| Table 4 — 18 pooled attributes with SHAP | `tables/table04_selected_features_shap.csv` |
| Table 5a — NPM primary performance | `tables/table05_model_performance_npm.csv` |
| Table 5b — pooled performance | `tables/table05_model_performance.csv` |
| Table 6 — grouped paired tests | `tables/table06_grouped_paired_tests.csv` |
| Table 7 — matched-budget selectors | `tables/table07_matched_budget.csv` |
| Table 8 — robustness | `tables/table09_robustness.csv` |
| Table 9 — operational prevalence | `tables/table12_operational_prevalence_group_level.csv` |
| Figure 3 — confusion matrix | `figures/figure03_confusion_matrix.png` |
| Figure 4 — budget sweep | `figures/figure04_budget_sweep.png` + `_data.csv` |
| Figure 5 — SHAP, primary NPM | `figures/figure05_shap_npm.png`, `results/shap_npm.csv` |
| Figure 6 — reliability diagram | `figures/figure06_calibration.png` + `_data.csv` |

### Supplementary Materials

| Element | File |
|---|---|
| Table S1 — composite groups by partition | `tables/tableS_group_counts.csv` |
| Table S2 — leakage decomposition, blocks 1 and 2 | `tables/tableS_leakage_block1_grouping.csv`, `tables/tableS_leakage_block2_fixed_test.csv` |
| Table S3 — evasion by source field | `results/evasion_curve.csv` |
| Table S4 — 18 NPM attributes with SHAP | `results/shap_npm.csv` |
| Table S5 — stage ordering | `results/stage_ordering_before_budget.csv` |
| Table S6 — family ablation | `tables/table08_family_ablation.csv` |
| Table S7 — leave-one-stage-out | `results/leave_one_stage_out.csv` |
| Figure S1 — SHAP, pooled secondary | `figures/figure05_shap_pooled.png` |

`tables/table10_operational_prevalence.csv` holds the version-level prevalence table, retained
for comparison. The table published in the paper is the **group-level** one, in which both the
numerator and the denominator of the false-positive bound are counted in composite groups.

---

## Notes on interpretation

**AU-PR baselines differ between analyses.** The pooled test partition is 71.4 % malicious
(AU-PR baseline 0.7138); the NPM test partition is 87.6 % malicious (baseline 0.8755). Reported
AU-PR values must be read against the corresponding prior, not against 0.5.

**Confidence intervals are cluster bootstrap intervals** resampled at the composite-group level
(2,000 resamples). For CatBoost on the pooled test partition the AU-PR interval is
[0.9771, 0.9972], width 0.0201 — 3.7 times wider than a version-level interval (0.0055). At
that scale none of the differences between feature subsets, stage orderings or metadata
families reported in the paper is distinguishable from sampling variation.

**Exploratory comparisons use the validation partition.** The budget sweep, stage ordering,
matched-budget selector comparison, family ablation and leave-one-stage-out ablation are all
estimated on validation. A single locked configuration is evaluated once on the test partition.

**Cross-ecosystem transfer is source-only.** Feature selection and hyperparameter tuning are
repeated inside the source ecosystem for each direction, so no target-ecosystem label
influences the feature set. NPM → PyPI attains AUC-ROC 0.5527 with a cluster-bootstrap interval
of [0.446, 0.677], which contains 0.5.

---

## Known limitations of this package

- **The dataset collection flow is not fully reconstructible.** Intermediate collection counts
  were recorded at collection time but were overwritten on a subsequent cache reload. The
  frozen dataset itself is complete and verifiable; the intermediate exclusion counts are not.
- **A fresh collection will not reproduce this sample.** See "Reproducing the results".
- **No family ablation was computed within NPM.** The attribution-versus-ablation comparison in
  the paper is reported for the pooled configuration only.
- **No equivalence test.** Differences are reported alongside the cluster-bootstrap interval
  width; a clustered paired non-inferiority test with a pre-declared margin has not been
  conducted.
- **Nested group-aware cross-validation** of the compressibility result has not been conducted.

---

## Data sources

Malicious package advisories come from the [OSSF Malicious Packages
repository](https://github.com/ossf/malicious-packages). Package metadata is retrieved from the
public NPM registry API and the PyPI JSON API. Both are used in accordance with their terms of
service; no package artefact is redistributed here, only derived numerical attributes.

## Citation

```bibtex
@article{moufid2026metadata,
  title   = {Metadata Compressibility and Evaluation Bias in Malicious Package
             Detection for NPM and PyPI},
  author  = {Moufid, Hanan and El Ghazouani, Mohamed and El Kiram, Moulay Ahmed},
  journal = {Journal of Cybersecurity and Privacy},
  year    = {2026},
  note    = {Manuscript jcp-4466772, under review}
}
```

## License

Code released under the MIT License. Derived data released under CC BY 4.0.

## Contact

Hanan Moufid — LISI Laboratory, Faculty of Sciences Semlalia, Cadi Ayyad University,
Marrakesh — `h.moufid.ced@uca.ac.ma`
