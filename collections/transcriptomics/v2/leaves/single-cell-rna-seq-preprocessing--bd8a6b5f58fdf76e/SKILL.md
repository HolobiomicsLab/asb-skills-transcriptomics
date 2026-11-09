---
name: single-cell-rna-seq-preprocessing
description: Use when a raw single-cell RNA-Seq expression matrix (genes x cells,
  FPKM/TPM) plus a per-cell sample table must be quality-filtered and normalized on
  a desktop/laptop before cell clustering and cell type identification.
when_to_use_negative:
- Input is already a filtered and normalized expression matrix — re-running prefilterGenes
  and normalization.zscore would double-transform the data
- The analysis goal is downstream cell clustering, differential expression, or cell
  type assignment rather than preparing the data (use cluster.assignment, cluster.diffgenes,
  celltype.enrichment instead)
- Data are bulk RNA-Seq or already aggregated per sample — per-cell quality filtering
  (min.genes = 500) and per-sample z-score transformation are not applicable
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: SINCERA
  role: R S4 tool set providing construct, filterLowQualityCells, filterContaminatedCells,
    prefilterGenes, expr.minimum, batch.analysis, and normalization.zscore functions
    used for the preprocessing stage
  repo: https://github.com/xu-lab/SINCERA
provenance:
  source_task_ids:
  - task_006
  - task_001
  - task_005
  source_papers:
  - doi: 10.1007/978-1-4939-7710-9_15
    title: Single-Cell Transcriptome Analysis Using SINCERA Pipeline
  source_content_hash: sha256:55247a098742a176cf39969d1215b47876e3d6799d8d85e54e484d9b18e49b2c
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/xu-lab/SINCERA
  tools:
  - SINCERA
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# single-cell-rna-seq-preprocessing

## Summary

The SINCERA preprocessing stage for scRNA-Seq: construct an R S4 object from a genes x cells expression matrix plus per-cell sample table, then run low-quality-cell filtering, contaminated-cell removal, low-expression gene prefiltering, minimum-expression flooring, batch diagnostics, and per-sample z-score gene normalization before clustering.

## When to use

Use when a raw single-cell RNA-Seq expression matrix (genes x cells, FPKM/TPM) plus a per-cell sample table must be quality-filtered and normalized on a desktop/laptop before cell clustering and cell type identification.

## When NOT to use

- Input is already a filtered and normalized expression matrix — re-running prefilterGenes and normalization.zscore would double-transform the data
- The analysis goal is downstream cell clustering, differential expression, or cell type assignment rather than preparing the data (use cluster.assignment, cluster.diffgenes, celltype.enrichment instead)
- Data are bulk RNA-Seq or already aggregated per sample — per-cell quality filtering (min.genes = 500) and per-sample z-score transformation are not applicable

## Inputs

- exprfile: genes x cells expression matrix (FPKM/TPM)
- samplefile: single-column per-cell sample information table
- Optional: markers.1 and markers.2 (highly specific marker genes, e.g., Epcam and Pecam1) for filterContaminatedCells

## Outputs

- SINCERA S4 object containing filtered, z-score normalized expression data
- Normalized expression matrix (only cells/genes surviving the filters, per-gene z-score transformed)
- Per-cell QC summary: cells removed by each filter and batch diagnostic plots (quantile plots, MA plot, Q-Q plot, cell correlation/distance measures)

## How to apply

Install SINCERA from GitHub (devtools::install_github("xu-lab/SINCERA")) and run the construct function on exprfile (rows = genes, columns = cells) and samplefile (single-column per-cell sample information) to create the S4 object. Remove low-quality cells with filterLowQualityCells using min.expression = 1 FPKM/TPM and min.genes = 500, and optionally remove contaminated cells with filterContaminatedCells by supplying known markers (e.g., Epcam vs Pecam1) via markers.1 and markers.2. Then prefilterGenes to drop genes expressed >5 FPKM/TPM in fewer than two cells, and apply expr.minimum to replace FPKM/TPM values <= 0.01 with 0.01. Run batch.analysis to plot per-cell expression quantiles across batches and compare batches with MA plots, Q-Q plots, and cell correlation/distance measures. Finally run normalization.zscore for per-sample z-score transformation of each gene expression profile (within-sample transformation when cell type distribution is comparable across replicates, otherwise global transformation); the z-scored matrix is the input to downstream cluster.geneSelection (specificity threshold 0.7) and cluster.assignment.

## Related tools

- **SINCERA** (R S4 tool set providing construct, filterLowQualityCells, filterContaminatedCells, prefilterGenes, expr.minimum, batch.analysis, and normalization.zscore functions used for the preprocessing stage) — https://github.com/xu-lab/SINCERA

## Examples

```
devtools::install_github("xu-lab/SINCERA"); library(SINCERA); obj <- construct(exprfile = expr.matrix, samplefile = sample.info); obj <- normalization.zscore(expr.minimum(prefilterGenes(filterLowQualityCells(obj))))
```

## Evaluation signals

- Output matrix contains only cells/genes surviving the filters: genes expressed >5 FPKM/TPM in fewer than two cells are gone, and every retained cell passes min.expression = 1 FPKM/TPM with min.genes = 500
- Per-gene values in the normalized matrix are z-score transformed (mean 0, unit variance per gene within sample or globally)
- QC summary accounts for every removed cell, attributing each removal to filterLowQualityCells, filterContaminatedCells, or prefilterGenes
- expr.minimum floor is respected: no FPKM/TPM value below 0.01 remains (values <= 0.01 set to 0.01)
- Batch diagnostics recorded: expression quantile plots across batches plus MA plot, Q-Q plot, and cell correlation/distance comparisons completed

## Limitations

- Preprocessing assumes FPKM/TPM-scaled expression data (thresholds of 1 and 5 FPKM/TPM and a 0.01 floor are expressed in those units), not raw UMI counts
- filterContaminatedCells requires prior knowledge of highly specific marker genes (e.g., Epcam for epithelial, Pecam1 for endothelial) to detect co-expression contamination
- Choice of z-score transformation mode depends on experimental design: within-sample transformation is valid only when cell type distribution is comparable across replicates; otherwise a global transformation must be used

## Evidence
