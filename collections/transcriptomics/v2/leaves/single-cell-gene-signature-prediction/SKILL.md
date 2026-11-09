---
name: single-cell-gene-signature-prediction
description: Use when the analysis goal is to derive cell type-specific signature
  genes from a clustered SINCERA R S4 expression object, once cell types have been
  identified and marker genes can be registered or must be inferred de novo.
when_to_use_negative:
- Cell types have not yet been identified — this step sits downstream of SINCERA clustering
  and cell type identification and requires an object with assigned clusters/cell
  types.
- The aim is constructing transcriptional regulatory networks or identifying driving
  forces (key nodes) — those are later pipeline stages that follow gene signature
  prediction.
- Input is bulk RNA-Seq or already a feature/signature table rather than a single-cell
  SINCERA S4 object.
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: SINCERA
  role: R S4 pipeline providing setCellTypeMarkers, signature.prediction (logistic
    regression over four feature metrics or frequency/fold-change filters), plotHeatmap,
    and signature.validation for cell type-specific signature prediction
  repo: github.com/xu-lab/SINCERA
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1007/978-1-4939-7710-9_15
    title: Single-Cell Transcriptome Analysis Using SINCERA Pipeline
  source_content_hash: sha256:ae9aab2a6fc7c269861afe8a52c93dc1541d4a8ed8b7478727bbc44b9d22b803
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - SINCERA
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# single-cell-gene-signature-prediction

## Summary

Predicts cell type-specific gene signatures from single-cell RNA-Seq data within the SINCERA pipeline, after clustering and cell type identification and before transcriptional regulatory network construction. It integrates four feature metrics via logistic regression when markers are known, or applies frequency (>=30% cells) and fold change (>=1.5x vs. second-highest cluster) filters when markers are unavailable.

## When to use

Use when the analysis goal is to derive cell type-specific signature genes from a clustered SINCERA R S4 expression object, once cell types have been identified and marker genes can be registered or must be inferred de novo.

## When NOT to use

- Cell types have not yet been identified — this step sits downstream of SINCERA clustering and cell type identification and requires an object with assigned clusters/cell types.
- The aim is constructing transcriptional regulatory networks or identifying driving forces (key nodes) — those are later pipeline stages that follow gene signature prediction.
- Input is bulk RNA-Seq or already a feature/signature table rather than a single-cell SINCERA S4 object.

## Inputs

- SINCERA R S4 object (expression matrix with assigned cell clusters/cell types from the cell type identification step)
- Positive and negative marker genes per cell type (optional; enables the logistic regression ranking mode)

## Outputs

- Ranked/predicted signature genes per cell type
- signature_genes_table.csv (predicted signature genes per cell type)
- plotHeatmap visualization of signature gene expression across cell types
- signature.validation repeated random subsampling classification accuracy report

## How to apply

Load the SINCERA R S4 object (created by the construct function) carrying the expression matrix and cell clusters/cell types assigned in the upstream cell type identification step. Register positive and negative marker genes per cell type with setCellTypeMarkers, then run signature.prediction: if markers exist, it integrates the four feature metrics — common gene metric, unique gene metric, test statistic metric, and synthetic profile similarity — with a logistic regression model to rank candidate signature genes; if markers are unavailable (e.g., novel cell types), it instead applies a frequency filter (genes expressed in at least 30% of cells in the cluster) and a fold change filter (average expression enriched at least 1.5-fold versus the cluster with the second highest average) to refine differentially expressed genes. Visualize predicted signature genes across cell types with plotHeatmap to confirm selective expression, then run signature.validation, which uses repeated random subsampling to measure classification accuracy of the signature in separating the given cell type from others. Export the per-cell-type signature genes to a signature_genes_table.csv.

## Related tools

- **SINCERA** (R S4 pipeline providing setCellTypeMarkers, signature.prediction (logistic regression over four feature metrics or frequency/fold-change filters), plotHeatmap, and signature.validation for cell type-specific signature prediction) — github.com/xu-lab/SINCERA

## Examples

```
signature.prediction(eset); plotHeatmap(eset, "signature"); signature.validation(eset)
```

## Evaluation signals

- signature.validation passes: repeated random subsampling shows the predicted signature achieves high classification accuracy in separating cells of the given type from cells of other types.
- plotHeatmap confirms selective expression of signature genes in their defined clusters across cell types.
- For the marker-less mode, retained genes satisfy the frequency filter (expressed in at least 30% of cells in the cluster) and the fold change filter (>=1.5-fold enrichment versus the cluster with second highest average).
- signature_genes_table.csv contains signature genes for each mapped cell type with no missing cell types.
- The signature prediction stage precedes transcriptional regulatory network construction in the pipeline order.

## Limitations

- Marker-gene ranking mode depends on a precompiled cell type and gene association table built from EBI Expression Atlas experimental expression data; novel cell types without registered markers fall back to the frequency (30%) and fold change (1.5x) filters.
- Signature quality is contingent on upstream cell clustering and cell type identification, which 'directly influences all downstream analysis'.

## Evidence
