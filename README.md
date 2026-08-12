# Metadata Compressibility and Evaluation Bias in Malicious Package Detection for NPM and PyPI

Replication package for the manuscript *Metadata Compressibility and Evaluation Bias in Malicious Package Detection for NPM and PyPI*, submitted to the **Journal of Cybersecurity and Privacy** (manuscript ID jcp-4466772).

## What this repository contains

The pipeline, the feature inventory, the partition assignment for every package, and the raw results file from which every figure and table in the paper is derived.
.
├── README.md
├── 01_Main_Pipeline.ipynb # Data collection, protocols A & B, core results
├── 02_Ablation_Experiments.ipynb # 9 ablation experiments (budget, order, stability, etc.)
├── DATA/
│ ├── dataset.csv                                  # raw collected data
│ ├── dataset_versions_point_in_time.csv           # 3,330 versions x 464 columns
│ ├── feature_inventory.csv                        # all 464 candidate features with definitions
│ └── partition_assignment.csv                     # package -> group_id -> train/val/test
├── FIGURES/
│ ├── figure03_confusion_matrix.png
│ ├── figure04_budget_sweep.svg
│ ├── figure04_budget_sweep_data.csv
│ ├── figure05_shap_summary.png
│ ├── figure06_calibration.png
│ └── figure06_calibration_data.csv
├── RESULTS/
│ ├── all_results.json                              # every reported number originates here
│ ├── budget_sweep_full_space.csv 
│ ├── cumulative_stage_curve.csv
│ ├── evasion_curve.csv
│ ├── feature_name_mapping.csv
│ ├── leave_one_stage_out.csv
│ ├── NOTE_full_space_curve.md                      # explanation of the full-space curve
│ ├── stage_ordering_after_budget.csv
│ ├── stage_ordering_before_budget.csv
│ ├── table03_selection_phases.csv
│ ├── table04_selected_features.csv
│ ├── table05_model_performance.csv
│ ├── table07_matched_budget.csv
│ ├── table08_family_ablation.csv
│ ├── table09_robustness.csv
│ └── table10_operational_prevalence.csv

## Run the main pipeline
Open and execute 01_Main_Pipeline.ipynb in Jupyter (or Google Colab).

This notebook:

Collects metadata from NPM and PyPI registries (first run: ~3 hours; subsequent runs: ~40 minutes using cache)

Extracts 464 features from package manifests at the publication timestamp

Implements the two evaluation protocols:

Protocol A: reproduces the initial submission (all features, random split, selection on full dataset)

Protocol B: corrected protocol (point-in-time features only, group-aware split, selection on training partition only)

Computes model performance (CatBoost, Random Forest, Stacking)

Generates SHAP explanations and calibration plots

Exports all core results to RESULTS/

## Run the ablation experiments
Open and execute 02_Ablation_Experiments.ipynb after the main pipeline has completed.

This notebook runs the 9 ablation experiments reported in the paper:

A. Cumulative stage curve – performance after each selection stage

B. Leave-one-stage-out – marginal contribution of each selection stage

C. Variance on the full space – effect of the variance threshold when applied before the ecosystem pre-filter

D. Stage ordering – six alternative orderings, measured before and after the budget stage

E. Matched-budget comparison – hybrid pipeline vs simpler selectors (L1-logistic, importance-based, random, PCA)

F. Budget sweep – performance vs feature-set size

G. Selection stability – Jaccard index across bootstrap resamples

H. Family ablation – contribution of each metadata family (authorship, release history, description, etc.)

I. Ecosystem pre-filter sensitivity – with vs without removal of ecosystem-proxy features

Collection notes
First run: data collection takes approximately 3 hours due to registry API rate limits. The dataset is cached as DATA/dataset_v5.csv after collection.

Subsequent runs: the cache is reused, and the pipeline completes in about 40 minutes.

Reproducibility over time: packages are removed from registries after detection, so a later run will retrieve fewer malicious versions than we did. DATA/dataset_versions_point_in_time.csv is the exact dataset used in the paper – we recommend using it to reproduce our numbers rather than collecting anew.

Random seed: all stochastic components are seeded with 42 for reproducibility.

Two evaluation protocols
The paper reports results under two protocols applied to the same collected data. The difference between them is the central result.

Protocol A reproduces the protocol of the initial submission. The accuracy gap (5.9 percentage points) quantifies the inflation attributable to package-family leakage, maintainer leakage, post-hoc metadata accumulation, and selection on the full dataset.

Key results
Under the corrected protocol (Protocol B), on 3,330 package versions with group-aware splitting:

456 point-in-time features reduce to 18; AU-PR 0.991 against 0.991 for the full set

Performance saturates by 5 features; all sizes from 5 to 40 are indistinguishable

Selection matters: 18 features drawn at random from the 456 candidates reach AU-PR 0.905

The six-stage pipeline does not: it is within 0.0016 AU-PR of L1-regularised logistic selection at a matched budget

63.8% of discriminative power above chance is attributable to ecosystem identity

Cross-ecosystem transfer degrades to AUC-ROC 0.65

At 0.1% operational prevalence, positive predictive value is 0.047

Ablation highlight
02_Ablation_Experiments.ipynb demonstrates that the six-stage hybrid pipeline confers no measurable advantage over simpler embedded selectors at a matched budget of 18 features. The best alternative (L1-regularised logistic selection) reaches AU-PR 0.9895 against the pipeline's 0.9912 – a difference of 0.0016, smaller than the 95% bootstrap interval width of 0.0055. Practitioners can obtain equivalent performance with simpler feature selection approaches.

Data sources
Malicious package advisories: OpenSSF Malicious Packages repository, which aggregates GitHub Security Advisories, Snyk, and registry-issued notices in OSV format.

Benign package metadata: public NPM and PyPI registry APIs.

No credentials are required; no private data are used.

Personally Identifiable Information (PII) is removed at extraction: email local parts are discarded and only an ordinal category of the domain is retained.

Repository contents
The complete feature inventory with definitions, all hyperparameter grids and selected values, selection thresholds, partition assignments, and preprocessing and evaluation scripts are available in this repository.

Notebook: 

01_Main_Pipeline.ipynb: Sections 2 (Data Collection, Feature Engineering), 3.1 (Feature Selection Performance), 3.2 (Model Performance), 3.3 (Explainability), 3.4 (Comparative Analysis), 3.6 (Alert Quality)
02_Ablation_Experiments.ipynb: Sections 3.2 (Ablations: cumulative, leave-one-stage-out, ordering, matched-budget, stability, family ablation, ecosystem pre-filter)

Contact
Hanan Moufid
LISI Laboratory, Faculty of Sciences Semlalia
Cadi Ayyad University, Marrakech, Morocco
h.moufid.ced@uca.ac.ma