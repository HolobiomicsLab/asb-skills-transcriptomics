---
name: batch-effect-diagnostic-assessment
description: Use when scRNA-Seq expression data are collected across multiple batches
  or replicates and batch effects must be diagnosed before z-score normalization and
  clustering.
when_to_use_negative:
- When the dataset contains a single batch or replicate, there are no cross-batch
  distributions to compare and batch.analysis is unnecessary
- When the goal is downstream cell clustering or gene signature prediction rather
  than diagnosing confounding batch effects in preprocessing
edam_operation: http://edamontology.org/operation_2940
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: SINCERA
  role: R package whose batch.analysis function performs the batch-effect diagnostic
    assessment during preprocessing
  repo: https://github.com/xu-lab/SINCERA
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1007/978-1-4939-7710-9_15
    title: Single-Cell Transcriptome Analysis Using SINCERA Pipeline
  source_content_hash: sha256:827d705ffd1a68cd650c0a43278950bcad38b1bb96099e68228cd703315dcb92
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/xu-lab/SINCERA
  tools:
  - SINCERA
  edam_operation: http://edamontology.org/operation_2940
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# Batch-effect diagnostic assessment

## Summary

A SINCERA preprocessing skill that uses the batch.analysis function to detect and quantify batch differences in scRNA-Seq expression data by comparing per-cell expression quantiles, MA plots, Q-Q plots, and cell correlation/distance measures across batches before downstream analyses.

## When to use

Use when scRNA-Seq expression data are collected across multiple batches or replicates and batch effects must be diagnosed before z-score normalization and clustering.

## When NOT to use

- When the dataset contains a single batch or replicate, there are no cross-batch distributions to compare and batch.analysis is unnecessary
- When the goal is downstream cell clustering or gene signature prediction rather than diagnosing confounding batch effects in preprocessing

## Inputs

- SINCERA S4 object built from a genes-by-cells expression matrix (exprfile: rows = genes, columns = cells, FPKM/TPM values) and a single-column sample table (samplefile)
- Batch/replicate labels associated with each cell (from the sample table)
- Filtered expression matrix surviving filterLowQualityCells and prefilterGenes steps

## Outputs

- Per-cell expression quantile plots across batches
- MA plots and Q-Q plots comparing batch distributions
- Cell correlation/distance measures across batches
- Batch diagnostics recorded in a per-cell QC summary alongside counts of cells removed by each filter

## How to apply

After constructing the SINCERA S4 object (construct on the genes-by-cells expression matrix in FPKM/TPM), removing low-quality cells (filterLowQualityCells with min.expression = 1 FPKM/TPM and min.genes = 500) and prefiltering genes (prefilterGenes; expr.minimum with 0.01 FPKM/TPM), run batch.analysis to plot expression quantiles per cell across batches and to compare batch distributions with MA plots, Q-Q plots, and cell correlation/distance measures. Record the resulting batch diagnostic plots and numbers (e.g., cells removed by each filter plus batch comparisons) in a per-cell QC summary. The rationale is that batch effects, alongside cell cycle effects, are named confounding factors in scRNA-Seq studies, and quantifying them before normalization.zscore informs whether within-sample per-sample z-score transformation is adequate (when cell type distribution is comparable across replicates) or a global transformation is needed. The batch diagnostics justify and document the normalization choice prior to clustering and cell type identification.

## Related tools

- **SINCERA** (R package whose batch.analysis function performs the batch-effect diagnostic assessment during preprocessing) — https://github.com/xu-lab/SINCERA

## Examples

```
library(SINCERA); batch.analysis(sc)
```

## Evaluation signals

- batch.analysis runs without error on the SINCERA S4 object and returns the quantile-per-cell plots plus MA, Q-Q, and cell correlation/distance comparisons
- The QC summary documents batch diagnostic plots together with every cell removed by filterLowQualityCells and filterContaminatedCells
- The downstream normalization.zscore choice (within-sample vs. global transformation) is explicitly justified by the batch diagnostics and cell type distribution comparability across replicates
- Quantile distributions across batches are plotted on the filtered matrix, i.e., only cells/genes surviving prefilterGenes (genes >5 FPKM/TPM in at least two cells) and expr.minimum (0.01 FPKM/TPM) are represented

## Limitations

- batch.analysis identifies batch differences but does not itself correct them; correction is implicit in the subsequent normalization.zscore choice
- Deciding between within-sample and global z-score transformation requires comparable cell type distribution across replicates; when this assumption fails, batch diagnostics alone are insufficient
- Batches must be labeled in the sample table; unlabeled data cannot be stratified for cross-batch comparison

## Evidence

- [methods] batch analysis quantiles: "Run batch.analysis function to identify batch differences."
- [intro] confounding factors: "handling the confounding factors such as batch and cell cycle effects"
- [methods] z-score normalization decision: "normalization.zscore function is applied to each gene expression profile for per-sample z-score transformation"
- [methods] upstream filtering thresholds: "Use expr.minimum function to set a minimum expression value."
- [intro] pipeline scope: "to conduct data filtering, normalization, clustering, cell type identification, gene signature prediction"
