---
name: signature-classification-accuracy-validation
description: Use when a predicted cell type-specific signature from single-cell RNA-Seq
  data (e.g., SINCERA signature.prediction output) must be validated for its ability
  to separate cells of the given type from all other types before export or reuse.
when_to_use_negative:
- Cell types have not yet been defined by upstream clustering and cell type identification
  — validation presupposes mapped cell types
- The goal is differential expression between conditions rather than validating a
  cell type signature's discriminative classification power
- Input is bulk RNA-Seq or already a validated signature table; the SINCERA validation
  operates on the single-cell S4 object with per-cell expression
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: SINCERA
  role: R S4 pipeline providing signature.prediction, signature.validation, setCellTypeMarkers,
    and plotHeatmap used to generate and validate the cell type-specific signature
  repo: github.com/xu-lab/SINCERA
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1007/978-1-4939-7710-9_15
    title: Single-Cell Transcriptome Analysis Using SINCERA Pipeline
  source_content_hash: sha256:0bc5ad325f8f5a132ac27556ff610280feae2f7bb41c845bf723410ae85cca92
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - SINCERA
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# signature-classification-accuracy-validation

## Summary

SINCERA's signature.validation step assesses whether a predicted cell type-specific gene signature actually discriminates that cell type, using a repeated random subsampling approach to estimate classification accuracy against other cell types.

## When to use

Use when a predicted cell type-specific signature from single-cell RNA-Seq data (e.g., SINCERA signature.prediction output) must be validated for its ability to separate cells of the given type from all other types before export or reuse.

## When NOT to use

- Cell types have not yet been defined by upstream clustering and cell type identification — validation presupposes mapped cell types
- The goal is differential expression between conditions rather than validating a cell type signature's discriminative classification power
- Input is bulk RNA-Seq or already a validated signature table; the SINCERA validation operates on the single-cell S4 object with per-cell expression

## Inputs

- SINCERA R S4 object with expression matrix, assigned cell clusters/cell types, and predicted signature genes from signature.prediction
- Positive and negative marker genes per cell type registered via setCellTypeMarkers (optional; triggers the four-metric logistic ranking path)

## Outputs

- Classification accuracy estimate for the predicted signature in distinguishing the given cell type from other types (repeated random subsampling result)
- signature_genes_table.csv with predicted signature genes per cell type (after validation passes)

## How to apply

After signature.prediction has ranked candidate signature genes for a mapped cell type (using the logistic regression model over the four feature metrics, or the frequency filter of ≥30% of cells plus fold change filter of ≥1.5 versus the second-highest cluster when markers are unavailable), run signature.validation on the SINCERA S4 object. The function applies a repeated random subsampling approach: it repeatedly splits cells of the target type versus cells of other types, trains/tests the signature's ability to classify held-out cells, and aggregates accuracy across repetitions. Acceptance requires that the predicted signature achieves high classification accuracy in separating cells of the given type from other types; pair this with plotHeatmap confirmation that signature genes show selective expression in their defined clusters. If validation fails, revisit the marker registration (setCellTypeMarkers) or the filtering thresholds before exporting signature_genes_table.csv.

## Related tools

- **SINCERA** (R S4 pipeline providing signature.prediction, signature.validation, setCellTypeMarkers, and plotHeatmap used to generate and validate the cell type-specific signature) — github.com/xu-lab/SINCERA

## Examples

```
library(SINCERA); sc <- signature.validation(sc)  # after signature.prediction on the S4 object; then plotHeatmap(sc)
```

## Evaluation signals

- signature.validation repeated random subsampling reports high classification accuracy for separating cells of the given type from other types
- plotHeatmap shows selective expression of the predicted signature genes in their defined clusters
- Marker-driven path: signature.prediction integrated all four feature metrics (common gene metric, unique gene metric, test statistic metric, synthetic profile similarity) via logistic regression
- Marker-less path: retained genes pass the ≥30% frequency filter and the ≥1.5 fold change filter versus the cluster with second highest average expression
- signature_genes_table.csv contains signature genes for each cell type after validation acceptance

## Limitations

- When marker genes are unavailable for novel cell types, signature ranking falls back on filtering heuristics (30% frequency, 1.5 fold change), which may be less specific than the four-metric logistic model
- Validation accuracy depends on the quality of upstream cell type identification; SINCERA notes that cell clustering and cell type identification 'directly influences all downstream analysis'

## Evidence
