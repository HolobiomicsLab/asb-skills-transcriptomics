---
name: logistic-regression-gene-ranking
description: Use when cell clusters and cell types are already assigned in a SINCERA
  single-cell RNA-Seq analysis and marker genes per cell type are available, so gene
  signatures can be ranked and validated.
when_to_use_negative:
- Cell clustering and cell type identification have not yet been performed, since
  the article states this step directly influences all downstream analysis and signature
  prediction follows cell type identification
- The analysis goal is transcriptional regulatory network construction or identification
  of driving forces (key nodes), which in the SINCERA pipeline comes after signature
  prediction
- Input is bulk RNA-Seq data rather than single-cell RNA-Seq, since SINCERA is designed
  for scRNA-Seq profiling analysis
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: SINCERA
  role: Provides setCellTypeMarkers, signature.prediction, signature.validation, and
    plotHeatmap functions for logistic regression-based gene signature ranking and
    validation
  repo: https://github.com/xu-lab/SINCERA
- name: EBI Expression Atlas
  role: Source of the precompiled cell type and gene association table used by SINCERA
    for cell type mapping
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1007/978-1-4939-7710-9_15
    title: Single-Cell Transcriptome Analysis Using SINCERA Pipeline
  source_content_hash: sha256:f49a45a0ec8f72c30f46ed5b474d6b21e7160d9346b4f5778b77d0c496b6fbc9
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/xu-lab/SINCERA
  tools:
  - EBI Expression Atlas
  - SINCERA
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# Logistic regression gene ranking

## Summary

SINCERA's signature.prediction step ranks cell type-specific signature genes by integrating four feature metrics (common gene metric, unique gene metric, test statistic metric, synthetic profile similarity) with a logistic regression model, following cell type identification and preceding transcriptional regulatory network construction.

## When to use

Use when cell clusters and cell types are already assigned in a SINCERA single-cell RNA-Seq analysis and marker genes per cell type are available, so gene signatures can be ranked and validated.

## When NOT to use

- Cell clustering and cell type identification have not yet been performed, since the article states this step directly influences all downstream analysis and signature prediction follows cell type identification
- The analysis goal is transcriptional regulatory network construction or identification of driving forces (key nodes), which in the SINCERA pipeline comes after signature prediction
- Input is bulk RNA-Seq data rather than single-cell RNA-Seq, since SINCERA is designed for scRNA-Seq profiling analysis

## Inputs

- SINCERA R S4 object containing the expression matrix and cell clusters/cell types from the upstream cell type identification step
- Positive and negative marker genes for each mapped cell type (registered via setCellTypeMarkers)

## Outputs

- Ranked predicted signature genes per cell type
- signature_genes_table.csv
- Heatmap of signature gene expression across cell types (via plotHeatmap)
- Signature validation classification accuracy estimate (via signature.validation)

## How to apply

Load the SINCERA R S4 object holding the expression matrix and assigned cell clusters/cell types from the upstream cell type identification step. Register positive and negative marker genes for each mapped cell type with the setCellTypeMarkers function. Run signature.prediction, which uses a logistic regression model to integrate the four feature metrics — common gene metric, unique gene metric, test statistic metric, and synthetic profile similarity — into a ranking of prediction; for novel cell types without markers, fall back to a frequency filter (genes expressed in at least 30% of cells in the cluster) and a fold change filter (average expression enrichment of at least 1.5 versus the cluster with second highest average). Visualize predicted signature genes across cell types with plotHeatmap to confirm selective expression, and assess classification accuracy with signature.validation using a repeated random subsampling approach. Export the predicted signature genes per cell type to signature_genes_table.csv.

## Related tools

- **SINCERA** (Provides setCellTypeMarkers, signature.prediction, signature.validation, and plotHeatmap functions for logistic regression-based gene signature ranking and validation) — https://github.com/xu-lab/SINCERA
- **EBI Expression Atlas** (Source of the precompiled cell type and gene association table used by SINCERA for cell type mapping)

## Examples

```
library(SINCERA); es <- setCellTypeMarkers(es, markers = markers); es <- signature.prediction(es)
```

## Evaluation signals

- signature.validation with repeated random subsampling achieves high classification accuracy in separating cells of the given type from cells of other types
- plotHeatmap confirms selective expression of signature genes in their defined clusters
- Marker-less fallback genes pass the frequency filter (expressed in at least 30% of cells in the cluster) and the fold change filter (at least 1.5 enrichment versus the cluster with second highest average)
- Each cell type with registered markers is ranked via the logistic regression model over all four feature metrics (common gene metric, unique gene metric, test statistic metric, synthetic profile similarity)
- Output file signature_genes_table.csv contains the predicted signature genes per cell type

## Limitations

- When marker genes are unavailable for novel cell types, the logistic regression ranking cannot be applied and only the frequency (30%) and fold change (1.5) filters refine differentially expressed genes
- Signature quality depends directly on the upstream cell clustering and cell type identification step, which the article notes directly influences all downstream analysis

## Evidence
