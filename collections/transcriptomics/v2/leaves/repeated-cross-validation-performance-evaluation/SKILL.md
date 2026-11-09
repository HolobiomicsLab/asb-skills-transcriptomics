---
name: repeated-cross-validation-performance-evaluation
description: 'Use when you need to quantify downstream benefit of an adjusted expression
  matrix: sample-level labels exist (e.g., Healthy / Mild-Moderate / Severe-Critical),
  and you must compare predictive accuracy between adjusted and raw data per cell
  type.'
when_to_use_negative:
- No per-sample phenotype labels (e.g., severity metadata) exist, since cross-validated
  pseudo-bulk classification cannot be performed
- The goal is batch correction/integration itself rather than downstream evaluation
  of an already-adjusted expression matrix
- Evaluation target is cell-level clustering or annotation quality rather than sample-level
  disease-state discrimination
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scMerge2
  role: Provides the adjusted expression matrix whose downstream prediction accuracy
    is being evaluated against the raw baseline
  repo: https://github.com/SydneyBioX/scMerge
- name: Seurat
  role: Used to handle the single-cell object and aggregate expression within each
    cell type and sample into pseudo-bulk signatures prior to SVM classification
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-023-39923-2
    title: Atlas-scale single-cell multi-sample multi-condition data integration using
      scMerge2
  source_content_hash: sha256:a1d007963160730efd3ee8013e0c14710dd86ab681fa29f608efdd652a91ea71
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
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 3
---

# repeated-cross-validation-performance-evaluation

## Summary

Evaluates whether scMerge2-adjusted cell-type-specific expression improves COVID-19 disease severity prediction by training an RBF-kernel SVM on sample-level pseudo-bulk signatures under repeated fivefold cross-validation, and comparing per-cell-type accuracy against the raw logcounts baseline.

## When to use

Use when you need to quantify downstream benefit of an adjusted expression matrix: sample-level labels exist (e.g., Healthy / Mild-Moderate / Severe-Critical), and you must compare predictive accuracy between adjusted and raw data per cell type.

## When NOT to use

- No per-sample phenotype labels (e.g., severity metadata) exist, since cross-validated pseudo-bulk classification cannot be performed
- The goal is batch correction/integration itself rather than downstream evaluation of an already-adjusted expression matrix
- Evaluation target is cell-level clustering or annotation quality rather than sample-level disease-state discrimination

## Inputs

- scMerge2-adjusted COVID-19 cell-type-specific expression matrix (SingleCellExperiment/Seurat object with adjusted logcounts)
- Consensus cell-type labels for all cells
- Per-sample severity metadata (Healthy / Mild-Moderate / Severe-Critical)
- Raw logcounts matrix for the unadjusted baseline

## Outputs

- Per-cell-type accuracy table (scMerge2-adjusted vs raw logcounts)
- Comparison boxplot of per-cell-type accuracies
- Average accuracy delta across cell types above 1% abundance

## How to apply

Load the scMerge2-adjusted COVID-19 expression matrix together with consensus cell-type labels and per-sample severity metadata, then aggregate expression within each cell type and sample to build sample-level pseudo-bulk signatures. Train an RBF-kernel SVM to predict the three severity classes and evaluate with repeated fivefold cross-validation, recording accuracy for each cell type with abundance above 1%. Repeat the identical pseudo-bulk aggregation and SVM cross-validation on the raw logcounts matrix to obtain the unadjusted baseline. Tabulate per-cell-type accuracies side by side and render a comparison boxplot; success is confirmed by the reported 3.2% average accuracy increase and the CD14 Monocyte rise from 81.3% to 83.6%. The rationale is that scMerge2 integration 'improves the performance of discriminating distinct cell states in COVID-19 patients with varying degrees of severity', so any benefit should surface as higher pseudo-bulk classification accuracy on adjusted vs raw data.

## Related tools

- **scMerge2** (Provides the adjusted expression matrix whose downstream prediction accuracy is being evaluated against the raw baseline) — https://github.com/SydneyBioX/scMerge
- **Seurat** (Used to handle the single-cell object and aggregate expression within each cell type and sample into pseudo-bulk signatures prior to SVM classification)

## Examples

```
fitControl <- trainControl(method = "repeatedcv", number = 5, repeats = 5); svmFit <- train(pseudo_bulk_expr, severity_labels, method = "svmRadial", trControl = fitControl)
```

## Evaluation signals

- Adjusted-data accuracy exceeds raw-data accuracy for all cell types above 1% abundance
- Average accuracy gain across qualifying cell types is approximately 3.2%
- CD14 Monocyte pseudo-bulk classification accuracy increases from 81.3% (raw) to 83.6% (adjusted)
- Cross-validation is repeated fivefold; per-cell-type accuracies are recorded for every fold/repetition, not a single split
- Identical pseudo-bulk aggregation and SVM settings are used for both adjusted and raw matrices, so the only differing factor is the expression adjustment

## Limitations

- Interpretation depends on the availability of an adjusted expression matrix: many rapid integration methods return only embeddings, which 'diminish their potency and generalizability' for sample-level analysis
- The severity classifier uses an RBF-kernel SVM on pseudo-bulk signatures; results are specific to this classifier and the three-class severity scheme (Healthy / Mild-Moderate / Severe-Critical)
- Only cell types above 1% abundance are evaluated, since rarer cell types lack sufficient pseudo-bulk replicates for stable cross-validated estimates

## Evidence

- [abstract] Discriminating disease progression accuracy: "reveals signatures derived from cell-type expression that are more accurate in discriminating disease progression"
- [intro] Integration improves severity discrimination: "the integration using scMerge2 improves the performance of discriminating distinct cell states in COVID-19 patients with varying degrees of severity"
- [intro] Adjusted expression matrices needed for sample-level analysis: "the lack of adjusted expression matrices restricts the utilisation of such integrative results and diminishes their potency and generalizability"
- [readme] scMerge2 package is the R implementation providing the adjusted matrix: "scMerge is a R package for merging and normalising single-cell RNA-Seq datasets."
