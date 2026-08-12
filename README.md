# Metadata-Based Detection of Malicious Packages in NPM and PyPI: Feature Reduction and Evaluation Bias

Replication package for the manuscript *Metadata-Based Detection of Malicious Packages in NPM and PyPI: Feature Reduction and Evaluation Bias*, submitted to the **Journal of Cybersecurity and Privacy** (manuscript ID jcp-4466772).

## What this repository contains 

The pipeline, the feature inventory, the partition assignment for every package, and the raw results file from which every figure and table in the paper is derived.

.
├── README.md
├── Requirements.txt
├── Pipeline.py                                    # single entry point, runs end to end
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

## Reproducing the results

```bash
pip install -r Requirements.txt
python Pipeline.py
```
The script runs unattended. First execution collects metadata from the NPM and PyPI registries (approximately three hours) and caches the dataset; subsequent runs read the cache and complete in about forty minutes. Every stochastic component is seeded with 42.

Collection is not perfectly reproducible over time. Packages are removed from registries after detection, so a later run will retrieve fewer malicious versions than we did. DATA/dataset_versions_point_in_time.csv is the exact dataset used in the paper; point the CACHE variable at it to reproduce our numbers rather than collect anew.

Two evaluation protocols

The paper reports results under two protocols applied to the same collected data. The difference between them is the central result.

Protocol A	Protocol B
Features	all 464, including current registry state	456 point-in-time features, followed by ecosystem-proxy removal
Split	random stratified	group-aware (package, maintainer, name-similarity family)
Selection and scaling	fitted on the full dataset	fitted on the training partition only
Test accuracy (CatBoost)	0.981	0.918
Protocol A reproduces the protocol of the initial submission. The accuracy gap quantifies the inflation attributable to package-family leakage, maintainer leakage, post-hoc metadata accumulation and selection on the full dataset.

Ablation experiments
Ten experiments, all in Pipeline.py.
budget_sweep_full_space.csv is deliberately not used in the paper. It is computed on the full 456-feature space before removing ecosystem-proxy features. At k = 2, the selector picks ecosystem indicators rather than informative metadata, producing a curve that artificially rises due to ecosystem imbalance. Figure 4 in the paper uses the post-filter curve (Section F), where saturation occurs at k = 5. We publish this file for transparency and to demonstrate why ecosystem-proxy removal is essential in the corrected protocol. See NOTE_full_space_curve.md for details.

Headline results
Under the corrected protocol, on 3,330 package versions (with group-aware splitting):

- 456 point-in-time features reduce to 18; AU-PR 0.991 against 0.991 for the full set
- Performance saturates by five features; all sizes from 5 to 40 are indistinguishable
- Selection matters: 18 features drawn at random from the 456 candidates reach 0.905
- The six-stage pipeline does not: it is indistinguishable from L1-regularised logistic selection at a matched budget
- 63.8% of discriminative power above chance is attributable to ecosystem identity
- Cross-ecosystem transfer degrades to AUC-ROC 0.65
- At 0.1% operational prevalence, positive predictive value is 0.047

Data sources
Malicious package advisories come from the OpenSSF Malicious Packages repository, which aggregates GitHub Security Advisories, Snyk and registry-issued notices in OSV format. Benign package metadata come from the public NPM and PyPI registry APIs. No credentials are required and no private data are used.

Personally identifiable information is removed at extraction: email local parts are discarded and only an ordinal category of the domain is retained.

Repository

The complete feature inventory with definitions, all hyperparameter grids and selected values, selection thresholds, partition assignments, and preprocessing and evaluation scripts are available in this repository.

Contact
h.moufid.ced@uca.ac.ma — LISI Laboratory, Faculty of Sciences Semlalia, Cadi Ayyad University, Marrakech, Morocco.