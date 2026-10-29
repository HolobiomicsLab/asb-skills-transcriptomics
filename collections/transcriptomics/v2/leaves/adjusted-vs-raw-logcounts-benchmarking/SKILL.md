---
name: adjusted-vs-raw-logcounts-benchmarking
description: Use when you have both an integration-adjusted expression matrix and
  the raw logcounts matrix with cell-type labels and per-sample metadata (e.g. severity),
  and need to quantify whether integration improves sample-level prediction.
when_to_use_negative:
- The integration method outputs only a low-dimensional embedding and no adjusted
  expression matrix — pseudo-bulk signatures cannot be constructed from embeddings
- The task is cell-type discovery or label transfer rather than quantifying whether
  integration improves sample-level prediction
- Cell types of interest fall below 1% abundance, where pseudo-bulk signatures are
  too sparse for reliable severity classification
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_0203
tools:
- name: scMerge2
  role: Produces the adjusted expression matrix and consensus cell-type labels that
    are benchmarked against raw logcounts
  repo: https://github.com/SydneyBioX/scMerge
- name: Seurat
  role: Used for handling single-cell expression objects and cell-type metadata in
    the benchmarking workflow
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-023-39923-2
    title: Atlas-scale single-cell multi-sample multi-condition data integration using
      scMerge2
  source_content_hash: sha256:c54c0e232134eae70f784d110efef1cc3dfd8bea29ea03a4283bd476463c19b2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SydneyBioX/scMerge
  tools:
  - Seurat
  - scMerge2
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# adjusted-vs-raw-logcounts-benchmarking

## Summary

Benchmarks whether scMerge2-adjusted expression improves downstream prediction over raw log-normalised data by building cell-type- and sample-level pseudo-bulk signatures and training an RBF-kernel SVM to predict COVID-19 disease severity under repeated fivefold cross-validation, comparing per-cell-type accuracy between adjusted and raw matrices.

## When to use

Use when you have both an integration-adjusted expression matrix and the raw logcounts matrix with cell-type labels and per-sample metadata (e.g. severity), and need to quantify whether integration improves sample-level prediction.

## When NOT to use

- The integration method outputs only a low-dimensional embedding and no adjusted expression matrix — pseudo-bulk signatures cannot be constructed from embeddings
- The task is cell-type discovery or label transfer rather than quantifying whether integration improves sample-level prediction
- Cell types of interest fall below 1% abundance, where pseudo-bulk signatures are too sparse for reliable severity classification

## Inputs

- scMerge2-adjusted COVID-19 expression matrix (adjusted logcounts)
- Raw log-normalised logcounts matrix (unadjusted baseline)
- Consensus cell-type labels per cell
- Per-sample severity metadata (Healthy / Mild-Moderate / Severe-Critical classes)

## Outputs

- Per-cell-type classification accuracy for the adjusted matrix
- Per-cell-type classification accuracy for the raw logcounts baseline
- Adjusted-vs-raw accuracy comparison (boxplot and tabulation)
- Average accuracy gain across cell types (reported: 3.2% average increase)

## How to apply

Load the adjusted expression matrix with consensus cell-type labels and per-sample severity metadata (Healthy / Mild-Moderate / Severe-Critical). Aggregate expression within each cell type and sample to construct sample-level pseudo-bulk signatures — mirroring scMerge2's pseudo-bulk building innovation that minimises processing demand — then train an RBF-kernel SVM classifier on those signatures. Evaluate with repeated fivefold cross-validation and record accuracy per cell type, restricting comparisons to cell types above 1% abundance. Repeat the identical pseudo-bulk aggregation and SVM cross-validation on the raw logcounts matrix to establish the unadjusted baseline. The rationale is that embedding-only integrations lack adjusted expression matrices, which 'restricts the utilisation of such integrative results and diminishes their potency and generalizability'; having an adjusted matrix enables this sample-level pseudo-bulk benchmark.

## Related tools

- **scMerge2** (Produces the adjusted expression matrix and consensus cell-type labels that are benchmarked against raw logcounts) — https://github.com/SydneyBioX/scMerge
- **Seurat** (Used for handling single-cell expression objects and cell-type metadata in the benchmarking workflow)

## Examples

```
BiocManager::install("scMerge"); ## then aggregate adjusted logcounts by cell type x sample into pseudo-bulk and run e1071::svm(severity ~ ., data = pseudobulk, kernel = "radial") under repeated 5-fold CV
```

## Evaluation signals

- Adjusted matrix should yield higher severity-prediction accuracy than raw logcounts for all cell types above 1% abundance
- Average accuracy increase across cell types should approximate the reported 3.2%
- CD14 Monocyte accuracy should rise from 81.3% (raw) to 83.6% (adjusted)
- Identical pseudo-bulk aggregation and cross-validation settings must be applied to both adjusted and raw matrices so the comparison is like-for-like
- Cross-validation must be repeated fivefold; accuracy should be reported per cell type, not only averaged

## Limitations

- Improvement is assessed only for cell types above 1% abundance; rare cell types are excluded from the comparison
- A low-dimensional embedding output alone 'increases interpretability challenges since a low-dimensional representation does not naturally lend itself to the development of interpretable features', so this benchmark requires an adjusted expression matrix
- Findings are specific to the COVID-19 data collection (~5 million cells, 963 donors, 1298 samples from 20 studies) and severity classes; transfer to other phenotypes is not established here

## Evidence

- [results] finding: severity prediction improvement: [evidence span withheld — anchor status: not_found]
- [abstract] signatures from cell-type expression: "reveals signatures derived from cell-type expression that are more accurate in discriminating disease progression"
- [intro] lack of adjusted expression matrices: "the lack of adjusted expression matrices restricts the utilisation of such integrative results and diminishes their potency and generalizability"
- [discussion] pseudo-bulk building: "three essential innovations with hierarchical integration, pseudo-bulk building to minimise processing demand, and pseudo-replication"
- [readme] scMerge2 R package: "`scMerge` is a R package for merging and normalising single-cell RNA-Seq datasets."
