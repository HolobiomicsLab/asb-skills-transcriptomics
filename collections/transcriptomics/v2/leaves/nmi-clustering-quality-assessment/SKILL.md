---
name: nmi-clustering-quality-assessment
description: Use when comparing unsupervised feature selection methods (e.g., DELVE,
  Hotspot, Laplacian Score) on single-cell data and you need a clustering-based score
  of how well each top-p feature set recapitulates known cell states or phases.
when_to_use_negative:
- When ground truth labels are unavailable — NMI requires reference annotations, so
  unsupervised-only settings need other evaluation criteria
- When the task is regression of continuous progression (e.g., cell cycle age in hours)
  — use SVM regressor RMSE or pseudotime Kendall correlation instead of NMI
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: delve_benchmark
  role: Provides the fs class with evaluate_select() to perform feature selection
    and score it via NMI clustering evaluation
  repo: https://github.com/jranek/delve_benchmark
- name: DELVE
  role: Feature selection method whose top-30 feature set is evaluated with the NMI
    clustering metric; expected to rank highest
  repo: https://github.com/jranek/delve
- name: Python
  role: Implementation language for the benchmarking pipeline (scikit-learn KMeans++
    and NMI computation)
  repo: https://github.com/jranek/delve
provenance:
  source_task_ids:
  - task_008
  - task_003
  source_papers:
  - doi: 10.1038/s41467-024-46773-z
    title: 'DELVE: feature selection for preserving biological trajectories in single-cell
      data'
  source_content_hash: sha256:e9d169d8af0f6eea4785ba6a3979bd48c0cea196856a5a681e4c569d21c7978e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jranek/delve_benchmark
  tools:
  - DELVE
  - Hotspot
  - Python
  - Slingshot
  - UMAP (uniform manifold approximation and projection)
  - delve_benchmark
  - scikit-learn
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# nmi-clustering-quality-assessment

## Summary

Evaluates feature selection methods by clustering cells with KMeans++ on the selected feature set and scoring agreement with ground truth labels (e.g., cell cycle phase) using normalized mutual information (NMI), as used in the DELVE benchmark for trajectory-preserving feature selection.

## When to use

Use when comparing unsupervised feature selection methods (e.g., DELVE, Hotspot, Laplacian Score) on single-cell data and you need a clustering-based score of how well each top-p feature set recapitulates known cell states or phases.

## When NOT to use

- When ground truth labels are unavailable — NMI requires reference annotations, so unsupervised-only settings need other evaluation criteria
- When the task is regression of continuous progression (e.g., cell cycle age in hours) — use SVM regressor RMSE or pseudotime Kendall correlation instead of NMI

## Inputs

- Preprocessed single-cell AnnData (.h5ad) object, cells x features (e.g., adata_RPE.h5ad with 2759 cells x 241 features, or PDAC 4i files with 253 features)
- Ground truth labels in adata.obs (e.g., cell cycle phase: G0, G1, S, G2, M)
- Ranked selected feature list per feature selection method (top p=30 features)

## Outputs

- NMI score per feature selection method between KMeans++ cluster labels and ground truth phase annotations
- Median NMI aggregated across methods and cell lines in a summary dataframe/CSV (e.g., NMI 0.599 for DELVE on RPE 4i)

## How to apply

After selecting the top p=30 features per method from the preprocessed .h5ad single-cell data (e.g., RPE 4i or nine PDAC 4i cell lines), cluster cells with KMeans++ using c centroids equal to the number of ground truth classes (c=4 for RPE cell cycle phases, c=5 for PDAC phases G0, G1, S, G2, M) and repeat with multiple random initializations (25 for RPE) to reduce seed sensitivity. Compute the normalized mutual information (NMI) between the KMeans cluster labels and the ground truth annotations in adata.obs (labels_key). Aggregate the median NMI per feature selection method across cell lines and trials, and rank methods: in the DELVE benchmark, similarity-based methods (DELVE, Hotspot) achieved the highest NMI, so a correct run should show them at or near the top. Use the delve_benchmark package's fs class, whose evaluate_select() method performs the selection-then-evaluation step and returns a score dataframe containing the NMI alongside other metrics (classification accuracy, precision@30, pseudotime correlation).

## Related tools

- **delve_benchmark** (Provides the fs class with evaluate_select() to perform feature selection and score it via NMI clustering evaluation) — https://github.com/jranek/delve_benchmark
- **DELVE** (Feature selection method whose top-30 feature set is evaluated with the NMI clustering metric; expected to rank highest) — https://github.com/jranek/delve
- **Python** (Implementation language for the benchmarking pipeline (scikit-learn KMeans++ and NMI computation)) — https://github.com/jranek/delve

## Examples

```
from delve_benchmark import fs; model = fs(adata=adata_RPE, fs_method=run_delve_fs, feature_threshold=30, labels_key='cell_cycle_phase'); predicted_features, score_df = model.evaluate_select()
```

## Evaluation signals

- NMI is in [0, 1]; a valid evaluation reports per-method NMI values with median aggregation across random KMeans initializations (e.g., 25 inits for RPE)
- DELVE and Hotspot should achieve the highest (or near-highest) NMI among the twelve methods; DELVE on RPE 4i should reproduce NMI ≈ 0.599
- Random features / all features baselines should score below DELVE and Hotspot; if not, check the labels_key and cluster count c
- Number of KMeans centroids c must equal the number of ground truth classes (c=4 for RPE phases, c=5 for PDAC phases)

## Limitations

- NMI reflects agreement with discrete phase labels only; it does not capture progression along trajectories, so it should be paired with pseudotime correlation and precision@30 metrics
- Feature selection can bias interpretation of the underlying cellular trajectory, so scores must be interpreted with care
- Results depend on ground truth annotation quality (e.g., manual gating of core cell cycle markers for PDAC 4i)

## Evidence

- [readme] evaluate_select() which can be used to evaluate a feature selection method's performance on a trajectory task: [evidence span withheld — anchor status: external_doc]
- [discussion] similarity-based methods achieved highest classification accuracy: [evidence span withheld — anchor status: not_found]
- [abstract] DELVE abstract claim: "we present DELVE, an unsupervised feature selection method for identifying a representative subset of molecular features which robustly recapitulate cellular trajectories"
