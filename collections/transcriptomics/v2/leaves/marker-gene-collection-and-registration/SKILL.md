---
name: marker-gene-collection-and-registration
description: Use when SINCERA cell types have been identified for a scRNA-Seq dataset
  and known positive/negative markers are needed as inputs to signature.prediction
  for cell type-specific signature prediction.
when_to_use_negative:
- Cell types are novel with no known markers available — in that case skip registration
  and rely on SINCERA's frequency (>=30% of cells) and fold change (>=1.5) filters
  on differentially expressed genes instead
- The SINCERA object has not yet passed cell clustering and cell type identification,
  since cell type identification directly influences all downstream analysis
- The analysis goal is a generic marker listing rather than predicting a cell type-specific
  gene signature for later validation with signature.validation
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: SINCERA
  role: Provides the setCellTypeMarkers function for registering positive/negative
    marker genes on the SINCERA R S4 object, and the precompiled cell type and gene
    association table sourced from EBI Expression Atlas
  repo: https://github.com/xu-lab/SINCERA
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1007/978-1-4939-7710-9_15
    title: Single-Cell Transcriptome Analysis Using SINCERA Pipeline
  source_content_hash: sha256:00516037c01b3a308d667a687386b8141edd6cf8ae0606fc4dd525a0de5e0425
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/xu-lab/SINCERA
  tools:
  - SINCERA
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# marker-gene-collection-and-registration

## Summary

Collects known positive and negative marker genes for each cell type mapped in a SINCERA analysis and registers them on the SINCERA R S4 object with setCellTypeMarkers, enabling downstream gene signature prediction, visualization, and validation.

## When to use

Use when SINCERA cell types have been identified for a scRNA-Seq dataset and known positive/negative markers are needed as inputs to signature.prediction for cell type-specific signature prediction.

## When NOT to use

- Cell types are novel with no known markers available — in that case skip registration and rely on SINCERA's frequency (>=30% of cells) and fold change (>=1.5) filters on differentially expressed genes instead
- The SINCERA object has not yet passed cell clustering and cell type identification, since cell type identification directly influences all downstream analysis
- The analysis goal is a generic marker listing rather than predicting a cell type-specific gene signature for later validation with signature.validation

## Inputs

- SINCERA R S4 object from the construct function holding the expression matrix
- Assigned cell clusters and mapped cell types from the upstream cell type identification step
- Known positive and negative marker gene lists per cell type (e.g., curated from the EBI Expression Atlas association table)

## Outputs

- SINCERA S4 object annotated with per-cell-type positive and negative marker gene sets
- Marker-registered object ready for signature.prediction and downstream signature_genes_table.csv export

## How to apply

After the upstream cell type identification step, load the SINCERA R S4 object (created by construct) that holds the expression matrix and the assigned cell clusters/cell types. For each mapped cell type, gather known positive markers (e.g., from a precompiled cell type and gene association table built from EBI Expression Atlas experimental expression data) and negative markers, then register both sets on the object using the setCellTypeMarkers function. Marker availability is a decision point: when markers are registered, signature.prediction integrates the common gene metric, unique gene metric, test statistic metric, and synthetic profile similarity via a logistic regression model to rank signature genes; when markers are unavailable (e.g., novel cell types), the pipeline instead applies a frequency filter (genes expressed in at least 30% of cells in the cluster) and a fold change filter (at least 1.5-fold average expression enrichment versus the cluster with second highest average). Verify registration by inspecting that each cell type on the SINCERA object has its marker sets attached before proceeding to signature prediction.

## Related tools

- **SINCERA** (Provides the setCellTypeMarkers function for registering positive/negative marker genes on the SINCERA R S4 object, and the precompiled cell type and gene association table sourced from EBI Expression Atlas) — https://github.com/xu-lab/SINCERA

## Examples

```
library(SINCERA); obj <- setCellTypeMarkers(obj, markers.positive = pos_markers, markers.negative = neg_markers, cell.type = "AT2")
```

## Evaluation signals

- Each mapped cell type on the SINCERA S4 object has both positive and negative marker sets registered via setCellTypeMarkers before signature.prediction is run
- Downstream signature.prediction uses the logistic regression over the four feature metrics (common gene, unique gene, test statistic, synthetic profile similarity) rather than falling back to the frequency/fold-change filter path, confirming markers were found
- plotHeatmap shows selective expression of predicted signature genes in their defined clusters
- signature.validation repeated random subsampling shows the predicted signature achieves high classification accuracy separating the given cell type from other types
- Predicted signature genes per cell type export cleanly to signature_genes_table.csv

## Limitations

- Marker-based signature prediction depends on marker availability; for novel cell types with no registered markers the pipeline must fall back to the frequency filter (>=30% of cells) and fold change filter (>=1.5 vs. second-highest cluster), which may miss subtle signatures
- SINCERA is designed for scRNA-Seq analysis on standard desktop/laptop computers, so very large datasets may be constrained by memory
- Registered markers are only as good as their source; SINCERA's precompiled association table derives from EBI Expression Atlas experimental data and may not cover all cell types

## Evidence

- [methods] Collect positive and negative marker genes for each mapped cell type and register them with the setCellTypeMarkers function.: [evidence span withheld — anchor status: not_found]
