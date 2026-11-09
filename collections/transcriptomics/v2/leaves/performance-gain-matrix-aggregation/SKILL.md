---
name: performance-gain-matrix-aggregation
description: Use when many inferred GRNs (one per cell type) have each been scored
  with a random-walk disease-subnetwork framework and you must aggregate gains into
  a cell-type-by-disease matrix for benchmarking methods like SCING vs GRNBOOST2,
  ppcor, PIDC.
when_to_use_negative:
- The GRNs have not yet been scored against degree-preserved random null networks
  with the Huang et al. random walk framework — aggregation would combine unvalidated
  or incomparable values.
- The input networks come from only a single cell type or tissue rather than the multi-cell-type
  MCA panel (e.g., 33 tissues / 106 cell types), so a cell-type-by-disease matrix
  has no meaningful rows to aggregate.
- The evaluation targets are not DisGeNET disease gene sets (e.g., perturb-seq perturbation
  target prediction or cosine-similarity held-out gene expression), where other validation
  metrics from the SCING pipeline apply instead.
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: SCING
  role: Primary GRN inference method whose random-walk disease-subnetwork performance
    gains are aggregated; outperformed the three benchmark methods across tissues
    and disease types
  repo: github.com/XiaYangLabOrg/SCING
- name: GRNBOOST2
  role: Benchmark GRN inference method (gradient boosting) evaluated against SCING
    to populate the performance-gain matrix
- name: ppcor
  role: Benchmark GRN inference method (partial correlation) evaluated against SCING
    to populate the performance-gain matrix
- name: PIDC
  role: Benchmark GRN inference method (conditional mutual information) evaluated
    against SCING to populate the performance-gain matrix
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.isci.2023.107124
    title: 'SCING: Inference of robust, interpretable gene regulatory networks from
      single cell and spatial transcriptomics'
  source_content_hash: sha256:cbd6bf233e8a46b88ab35b2d01a4e58ff66fbad39d83b05820bc202532b6c5cc
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GRNBOOST2
  - PIDC
  - SCING
  - conda
  - ppcor
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# performance-gain-matrix-aggregation

## Summary

Aggregates per-cell-type random-walk disease-subnetwork performance gains (real GRN vs. degree-preserved random GRNs) into a cell-type-by-disease-category matrix (e.g., disease_subnetwork_performance_gain.csv) spanning 273 Mouse Cell Atlas cell-type networks (33 tissues, 106 cell types) across Immune, Metabolic, and Neuronal DisGeNET disease gene sets.

## When to use

Use when many inferred GRNs (one per cell type) have each been scored with a random-walk disease-subnetwork framework and you must aggregate gains into a cell-type-by-disease matrix for benchmarking methods like SCING vs GRNBOOST2, ppcor, PIDC.

## When NOT to use

- The GRNs have not yet been scored against degree-preserved random null networks with the Huang et al. random walk framework — aggregation would combine unvalidated or incomparable values.
- The input networks come from only a single cell type or tissue rather than the multi-cell-type MCA panel (e.g., 33 tissues / 106 cell types), so a cell-type-by-disease matrix has no meaningful rows to aggregate.
- The evaluation targets are not DisGeNET disease gene sets (e.g., perturb-seq perturbation target prediction or cosine-similarity held-out gene expression), where other validation metrics from the SCING pipeline apply instead.

## Inputs

- Per-cell-type random-walk performance gains of inferred GRNs over degree-preserved random GRNs (Huang et al. framework) for Immune, Metabolic, and Neuronal DisGeNET disease gene sets
- Inferred GRNs for 273 Mouse Cell Atlas cell-type networks (33 tissues, 106 cell types), from SCING, GRNBOOST2, ppcor, and PIDC
- Disease gene sets from the DisGeNET database (Immune, Metabolic, Neuronal categories)

## Outputs

- disease_subnetwork_performance_gain.csv: a cell-type-by-disease-category performance-gain matrix
- Aggregated benchmarking evidence that SCING achieves higher random-walk performance gain than GRNBOOST2, ppcor, and PIDC across Immune, Metabolic, and Neuronal disease categories

## How to apply

First ensure every GRN was evaluated with the Huang et al. random walk framework against DisGeNET Immune, Metabolic, and Neuronal disease gene sets, where each real GRN is compared against degree-preserved random GRNs with similar node attributes; record the performance gain (real network minus random null) per cell type and disease category. Next, extend evaluation across the full mouse cell atlas panel — 33 tissues and 106 cell types — so that each of the 273 cell-type networks contributes one row per disease category. Aggregate these per-cell-type, per-disease-category gains into a two-dimensional matrix with cell types as rows and disease categories as columns, and save it as disease_subnetwork_performance_gain.csv. Use the matrix as a validation artifact: for the benchmark task, SCING should show higher random-walk performance gain than GRNBOOST2, ppcor, and PIDC across all three disease categories and tissues such as bone marrow, brain, and liver, consistent with the paper's claimed improved disease modeling accuracy. When interpreting, remember the gains quantify improvement over nulls with similar node degree characteristics, not absolute disease-prediction accuracy.

## Related tools

- **SCING** (Primary GRN inference method whose random-walk disease-subnetwork performance gains are aggregated; outperformed the three benchmark methods across tissues and disease types) — github.com/XiaYangLabOrg/SCING
- **GRNBOOST2** (Benchmark GRN inference method (gradient boosting) evaluated against SCING to populate the performance-gain matrix)
- **ppcor** (Benchmark GRN inference method (partial correlation) evaluated against SCING to populate the performance-gain matrix)
- **PIDC** (Benchmark GRN inference method (conditional mutual information) evaluated against SCING to populate the performance-gain matrix)

## Evaluation signals

- The aggregated matrix disease_subnetwork_performance_gain.csv has rows for 273 MCA cell-type networks (33 tissues, 106 cell types) and columns for Immune, Metabolic, and Neuronal DisGeNET disease categories.
- For every tissue benchmarked (bone marrow, brain, liver) and every disease category, SCING's performance gain over degree-preserved random networks exceeds that of GRNBOOST2, ppcor, and PIDC.
- Each matrix entry reflects a gain relative to random GRNs with similar node degree/node attributes, not an absolute AUROC — verify the null-generation step (degree-preserved random GRNs) preceded aggregation.
- Per-entry values are consistent with the paper's finding that the mouse cell atlas combined with DisGeNET demonstrates improved accuracy of SCING over existing methods.

## Limitations

- Performance gains are relative to degree-preserved random networks with similar node attributes, so they measure disease-subnetwork modeling improvement over nulls rather than absolute predictive accuracy.
- Many hyperparameters in the upstream GRN pipeline require selection, and while the paper shows robustness across a range of hyperparameters, selection can be dataset dependent.
- Inferred GRN links are not necessarily causal or directional, which limits biological interpretation of aggregated disease-subnetwork gains.

## Evidence

- [methods] We utilize a random walk framework from Huang et al. to determine the increase in performance of a GRN to model disease subnetworks versus random GRNs: "We utilize a random walk framework from Huang et al. to determine the increase in performance of a GRN to model disease subnetworks versus random GRNs"
- [readme] We utilize a random walk framework from Huang et al. to determine the increase in performance of a GRN to model disease subnetworks versus random GRNs with similar node attributes: "We utilize a random walk framework from Huang et al. to determine the increase in performance of a GRN to model disease subnetworks versus random GRNs with similar node attributes"
- [abstract] Performance evaluation using Perturb-seq datasets, held-out data, and the mouse cell atlas combined with the DisGeNET database demonstrates the improved accuracy and biological interpretability of: "Performance evaluation using Perturb-seq datasets, held-out data, and the mouse cell atlas combined with the DisGeNET database demonstrates the improved accuracy and biological interpretability of"
- [methods] and to the mouse cell atlas in 33 tissues and 106 cell types: "and to the mouse cell atlas in 33 tissues and 106 cell types"
