---
name: gene-expression-zscore-normalization
description: Use when an FPKM/TPM expression matrix (genes x cells) from scRNA-Seq
  has been quality-filtered and expression-floored (expr.minimum = 0.01) and a scale-free
  gene profile is needed before correlation-distance-based cell clustering.
when_to_use_negative:
- Input is raw unfiltered UMI counts rather than FPKM/TPM values — the z-score step
  assumes prior SINCERA filtering and the 0.01 expression floor have been applied
- The downstream method requires absolute expression magnitudes (e.g., differential
  expression testing such as cluster.diffgenes one-tailed Welch's t test on expression
  values) rather than standardized profiles
- Batch differences have not yet been assessed — run batch.analysis (MA plots, Q-Q
  plots, cell correlation/distance measures) first, since normalization decisions
  depend on batch diagnostics
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: SINCERA
  role: R S4 pipeline whose normalization.zscore function performs the per-sample
    z-score transformation of each gene expression profile
  repo: https://github.com/xu-lab/SINCERA
provenance:
  source_task_ids:
  - task_001
  - task_005
  - task_002
  source_papers:
  - doi: 10.1007/978-1-4939-7710-9_15
    title: Single-Cell Transcriptome Analysis Using SINCERA Pipeline
  source_content_hash: sha256:f22316b869f2bfcb371a36318586b5758f1aae69255a820c5c90cf8d517cdae4
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/xu-lab/SINCERA
  tools:
  - SINCERA
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# Gene-expression z-score normalization

## Summary

SINCERA's gene-level normalization step (normalization.zscore) applies per-sample z-score transformation to each gene's expression profile (FPKM/TPM values) so that clustering of cells is driven by relative expression patterns rather than absolute expression magnitudes.

## When to use

Use when an FPKM/TPM expression matrix (genes x cells) from scRNA-Seq has been quality-filtered and expression-floored (expr.minimum = 0.01) and a scale-free gene profile is needed before correlation-distance-based cell clustering.

## When NOT to use

- Input is raw unfiltered UMI counts rather than FPKM/TPM values — the z-score step assumes prior SINCERA filtering and the 0.01 expression floor have been applied
- The downstream method requires absolute expression magnitudes (e.g., differential expression testing such as cluster.diffgenes one-tailed Welch's t test on expression values) rather than standardized profiles
- Batch differences have not yet been assessed — run batch.analysis (MA plots, Q-Q plots, cell correlation/distance measures) first, since normalization decisions depend on batch diagnostics

## Inputs

- Filtered SINCERA S4 object or genes x cells expression matrix (FPKM/TPM) surviving filterLowQualityCells (min.expression = 1, min.genes = 500), prefilterGenes, and expr.minimum (0.01 floor)
- Single-column sample table mapping cells to samples (for per-sample transformation and choosing within-sample vs global mode)

## Outputs

- Z-score transformed genes x cells expression matrix (e.g., normalized_expression_matrix.tsv)
- Choice and rationale for within-sample vs global transformation recorded in the QC summary

## How to apply

After SINCERA preprocessing (filterLowQualityCells with min.expression = 1 FPKM/TPM and min.genes = 500, prefilterGenes, and expr.minimum flooring values at 0.01), run the batch.analysis diagnostics first, then call normalization.zscore on each gene expression profile in the SINCERA S4 object for per-sample z-score transformation. Choose within-sample transformation when the cell type distribution is comparable across replicates; otherwise use global transformation so that cell type composition differences across samples are not distorted. The z-scored matrix feeds directly into cluster.geneSelection (specificity threshold 0.7) and cluster.assignment, which uses Pearson correlation-based distance, so z-scoring is essential to make gene profiles comparable. Write the filtered, normalized matrix to normalized_expression_matrix.tsv and verify per-gene values are z-score transformed.

## Related tools

- **SINCERA** (R S4 pipeline whose normalization.zscore function performs the per-sample z-score transformation of each gene expression profile) — https://github.com/xu-lab/SINCERA

## Examples

```
library(SINCERA); obj <- construct(exprfile, samplefile); obj <- normalization.zscore(obj)
```

## Evaluation signals

- Per-gene values in normalized_expression_matrix.tsv have mean approximately 0 and standard deviation approximately 1 across cells
- The output matrix contains only cells and genes that survived filterLowQualityCells, filterContaminatedCells, prefilterGenes, and expr.minimum
- The transformation mode (within-sample vs global) is documented and matches the comparability of cell type distribution across replicates
- Downstream cluster.assignment with Pearson correlation-based distance and cluster.permutation.analysis yields significant clusters whose marker genes show selective (non-overlapping) expression
- The per-cell QC summary accounts for every removed cell and includes batch diagnostic plots from batch.analysis

## Limitations

- Within-sample z-score transformation can misrepresent profiles when cell type distribution differs across replicates; global transformation must be used instead in that case
- Z-scoring is applied to FPKM/TPM values floored at 0.01 FPKM/TPM, so very lowly expressed genes still carry floor-induced artifacts
- The paper does not report a dedicated batch-correction normalization; batch differences are only diagnosed via batch.analysis, not removed by z-scoring

## Evidence
