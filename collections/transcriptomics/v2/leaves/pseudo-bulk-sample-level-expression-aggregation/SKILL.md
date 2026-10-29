---
name: pseudo-bulk-sample-level-expression-aggregation
description: Use when a single-cell atlas (e.g. the scMerge2-integrated COVID-19 collection
  of ~5 million cells, 963 donors, 1298 samples) must be reduced to per-sample, per-cell-type
  expression signatures for sample-level prediction or comparison of adjusted vs raw
  logcounts.
when_to_use_negative:
- When only a low-dimensional embedding is available (e.g. outputs of embedding-only
  integration methods like deepMNN/BBKNN/scVI) — embeddings do not support gene-expression
  pseudo-bulk construction and 'an embedding output by itself increases interpretability
  challenges'
- When the goal is per-cell rather than per-sample analysis — pseudo-bulk aggregation
  deliberately collapses cells within each cell type and sample
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: scMerge2
  role: Supplies the adjusted expression matrix whose cell-type-specific pseudo-bulk
    aggregates improve severity prediction over raw logcounts
  repo: https://github.com/SydneyBioX/scMerge
- name: Seurat
  role: Single-cell data handling for cell-type labels and expression aggregation
    into pseudo-bulk signatures
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-023-39923-2
    title: Atlas-scale single-cell multi-sample multi-condition data integration using
      scMerge2
  source_content_hash: sha256:2966ea112a8c4c71176a17acf9d18a7f005ef585bdaeeb5fdf5c3cc0f45f7c4a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SydneyBioX/scMerge
  tools:
  - Seurat
  - scMerge2
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# pseudo-bulk-sample-level-expression-aggregation

## Summary

Aggregates single-cell expression within each cell type and sample into sample-level pseudo-bulk signatures so that downstream models (e.g. an RBF-kernel SVM predicting Healthy / Mild-Moderate / Severe-Critical severity) can operate on the adjusted expression matrix returned by scMerge2.

## When to use

Use when a single-cell atlas (e.g. the scMerge2-integrated COVID-19 collection of ~5 million cells, 963 donors, 1298 samples) must be reduced to per-sample, per-cell-type expression signatures for sample-level prediction or comparison of adjusted vs raw logcounts.

## When NOT to use

- When only a low-dimensional embedding is available (e.g. outputs of embedding-only integration methods like deepMNN/BBKNN/scVI) — embeddings do not support gene-expression pseudo-bulk construction and 'an embedding output by itself increases interpretability challenges'
- When the goal is per-cell rather than per-sample analysis — pseudo-bulk aggregation deliberately collapses cells within each cell type and sample

## Inputs

- scMerge2-adjusted COVID-19 expression matrix (adjusted logcounts)
- Raw log-normalised logcounts matrix (unadjusted baseline)
- Consensus cell-type labels (e.g. CD14 Monocytes and other annotated types)
- Per-sample severity metadata (Healthy / Mild-Moderate / Severe-Critical)

## Outputs

- Sample-level pseudo-bulk expression signature per cell type (cell-type x sample matrix)
- Per-cell-type classification accuracies from repeated fivefold cross-validation for adjusted vs raw logcounts
- Comparison boxplot of per-cell-type accuracies

## How to apply

Load the scMerge2-adjusted expression matrix together with consensus cell-type labels and per-sample severity metadata, then sum (aggregate) expression within each cell type and sample to construct a pseudo-bulk signature per cell-type-per-sample; restrict analyses to cell types above 1% abundance. Rationale: scMerge2 was designed to produce adjusted expression matrices precisely because 'the lack of adjusted expression matrices restricts the utilisation of such integrative results and diminishes their potency and generalizability' for sample-level analysis. Train an RBF-kernel SVM on the pseudo-bulk signatures to predict the three severity classes with repeated fivefold cross-validation, recording accuracy per cell type. Repeat the identical aggregation and cross-validation on the raw logcounts matrix to obtain the unadjusted baseline, and tabulate per-cell-type accuracies (adjusted vs raw) to quantify the integration benefit.

## Related tools

- **scMerge2** (Supplies the adjusted expression matrix whose cell-type-specific pseudo-bulk aggregates improve severity prediction over raw logcounts) — https://github.com/SydneyBioX/scMerge
- **Seurat** (Single-cell data handling for cell-type labels and expression aggregation into pseudo-bulk signatures)

## Evaluation signals

- Average accuracy across cell types above 1% abundance should increase by ~3.2% for scMerge2-adjusted vs raw logcounts
- CD14 Monocyte pseudo-bulk classifier accuracy should rise from 81.3% (raw) to 83.6% (adjusted)
- Pseudo-bulk matrix should have one signature per (cell type, sample) pair — verify dimensions against 1298 samples and the annotated cell-type set
- Adjusted and raw baselines must use the identical aggregation and repeated fivefold CV protocol so the comparison is apples-to-apples
- Prediction task should discriminate the three severity classes (Healthy / Mild-Moderate / Severe-Critical)

## Limitations

- Pseudo-bulk building exists 'to minimise processing demand' of atlas-scale data, so it trades cell-level resolution for scalability
- Benefits depend on the integration method returning an adjusted expression matrix; embedding-only outputs restrict sample-level analysis
- The concatenated integration approach is more computationally intensive than the balanced-tree approach when generating the adjusted data being aggregated

## Evidence

- [discussion] pseudo-bulk building to minimise processing demand: "This was achieved via the use of three essential innovations with hierarchical integration, pseudo-bulk building to minimise processing demand, and pseudo-replication"
- [abstract] cell-type expression signatures discriminate disease progression: "reveals signatures derived from cell-type expression that are more accurate in discriminating disease progression"
- [intro] adjusted expression matrices needed for sample-level analysis: "the lack of adjusted expression matrices restricts the utilisation of such integrative results and diminishes their potency and generalizability"
- [readme] accessing count data for aggregation: [evidence span withheld — anchor status: external_doc]
