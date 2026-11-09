---
name: marker-gene-expression-visualization
description: Use when scRNA-Seq clusters or predicted cell type-specific signatures
  need quality assessment via known marker gene expression patterns, e.g. after cluster.assignment
  or signature.prediction in SINCERA.
when_to_use_negative:
- Input is a raw count matrix that has not yet been filtered, normalized, or z-score
  transformed — preprocessing must precede clustering and marker visualization
- No known marker genes are available for the tissue or cell types, so plotMarkers
  cannot discriminate cluster quality
- The goal is de novo cluster discovery rather than visual quality assessment of existing
  SINCERA cluster assignments or signatures
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: SINCERA
  role: R S4 pipeline providing plotMarkers and plotHeatmap for marker and signature
    expression visualization on standard desktop/laptop computers
  repo: github.com/xu-lab/SINCERA
provenance:
  source_task_ids:
  - task_002
  - task_003
  source_papers:
  - doi: 10.1007/978-1-4939-7710-9_15
    title: Single-Cell Transcriptome Analysis Using SINCERA Pipeline
  source_content_hash: sha256:da09b738520b5a168e0211ddd8c4c935a781e10f89df65553e5687111b419faf
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - SINCERA
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# marker-gene-expression-visualization

## Summary

A SINCERA-based visualization skill that renders expression patterns of known marker genes across cell clusters (plotMarkers) and predicted cell type-specific signature genes across cell types (plotHeatmap), using the visual pattern of selective versus scattered expression to accept or reject clustering and signature results.

## When to use

Use when scRNA-Seq clusters or predicted cell type-specific signatures need quality assessment via known marker gene expression patterns, e.g. after cluster.assignment or signature.prediction in SINCERA.

## When NOT to use

- Input is a raw count matrix that has not yet been filtered, normalized, or z-score transformed — preprocessing must precede clustering and marker visualization
- No known marker genes are available for the tissue or cell types, so plotMarkers cannot discriminate cluster quality
- The goal is de novo cluster discovery rather than visual quality assessment of existing SINCERA cluster assignments or signatures

## Inputs

- SINCERA R S4 object holding the z-score transformed expression matrix of cell type-selective genes (from cluster.geneSelection)
- Assigned cluster labels or cell types per cell from cluster.assignment / cell type identification
- Positive and negative marker gene lists registered per cell type (setCellTypeMarkers)
- Predicted signature genes per cell type from signature.prediction

## Outputs

- plotMarkers expression pattern plots of known marker genes across clusters for each clustering condition
- plotHeatmap of predicted signature genes across cell types
- Visual accept/reject decision per clustering condition (feeds cluster_assignments.csv and signature_genes_table.csv)

## How to apply

Load the preprocessed, z-score transformed SINCERA R S4 object and, after running cluster.assignment under each clustering condition (average-linkage hierarchical with Pearson correlation distance, ward linkage, consensus, and tight clustering), call plotMarkers to inspect expression patterns of known marker genes across clusters; scattered or overlapping marker expression indicates a low-quality clustering scheme, so iterate with different parameter settings. Accept a clustering condition only when the cluster membership permutation analysis (cluster.permutation.analysis) supports cluster significance and markers show selective (non-overlapping) expression. After signature.prediction produces candidate signature genes per cell type, run plotHeatmap to confirm their selective expression in the defined clusters before exporting the signature gene table. Rationale: cell clustering and cell type identification directly influence all downstream SINCERA analysis, so visual marker evidence is the primary acceptance criterion.

## Related tools

- **SINCERA** (R S4 pipeline providing plotMarkers and plotHeatmap for marker and signature expression visualization on standard desktop/laptop computers) — github.com/xu-lab/SINCERA

## Examples

```
devtools::install_github("xu-lab/SINCERA"); library(SINCERA)
```

## Evaluation signals

- plotMarkers output shows known marker genes selectively (non-overlapping) expressed across defined clusters; scattered or overlapping patterns indicate the clustering scheme should be rejected
- plotHeatmap confirms selective expression of predicted signature genes in their defined cell type clusters
- Each accepted clustering condition is also supported by cluster.permutation.analysis (cluster membership permutation significance)
- Predicted signatures additionally pass signature.validation repeated random subsampling with high classification accuracy separating the given cell type from others
- Exported tables (cluster_assignments.csv, signature_genes_table.csv) contain one row per cell/cell type with identifiers matching the input S4 object

## Limitations

- Marker-based visual assessment is subjective: scattered marker expression forces iteration over clustering parameters rather than giving an automatic decision
- When marker genes are unavailable for novel cell types, signature genes must instead be refined by the frequency filter (expression in at least 30% of cells in the cluster) and the fold change filter (at least 1.5-fold average expression enrichment versus the second-highest cluster), which the heatmap alone cannot replace

## Evidence
