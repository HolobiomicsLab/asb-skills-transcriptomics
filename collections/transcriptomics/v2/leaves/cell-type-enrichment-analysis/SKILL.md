---
name: cell-type-enrichment-analysis
description: Use when SINCERA hierarchical clustering has produced cell clusters and
  each cluster must be annotated with a predicted cell type using differentially expressed
  genes and a Fisher's exact test against the EBI Expression Atlas annotation table.
when_to_use_negative:
- Input lacks cluster-level differentially expressed genes — enrichment requires marker
  gene sets from cluster.diffgenes, not raw expression matrices
- The cell types of interest are not represented in the EBI Expression Atlas-derived
  annotation table (e.g. novel or highly specialized cell types)
- Cells have not yet been clustered via cluster.assignment; cell type prediction operates
  on defined clusters, not individual cells
edam_operation: http://edamontology.org/operation_3501
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: SINCERA
  role: Implements celltype.enrichment (one-tailed Fisher's exact test against the
    EBI Expression Atlas cell type-gene table) and celltype.validation for cluster
    annotation
  repo: github.com/xu-lab/SINCERA
provenance:
  source_task_ids:
  - task_006
  - task_005
  source_papers:
  - doi: 10.1007/978-1-4939-7710-9_15
    title: Single-Cell Transcriptome Analysis Using SINCERA Pipeline
  source_content_hash: sha256:4d3661a559e51e55a734aae03a90e0c05988f5e4feb857ad574b654c789b51b3
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - SINCERA
  edam_operation: http://edamontology.org/operation_3501
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# cell-type-enrichment-analysis

## Summary

Assigns a predicted cell type to each SINCERA cluster by testing cluster marker genes for enrichment against a precompiled cell type-gene association table built from EBI Expression Atlas data, then cross-validates assignments with a rank-aggregation-based check.

## When to use

Use when SINCERA hierarchical clustering has produced cell clusters and each cluster must be annotated with a predicted cell type using differentially expressed genes and a Fisher's exact test against the EBI Expression Atlas annotation table.

## When NOT to use

- Input lacks cluster-level differentially expressed genes — enrichment requires marker gene sets from cluster.diffgenes, not raw expression matrices
- The cell types of interest are not represented in the EBI Expression Atlas-derived annotation table (e.g. novel or highly specialized cell types)
- Cells have not yet been clustered via cluster.assignment; cell type prediction operates on defined clusters, not individual cells

## Inputs

- SINCERA S4 object with assigned cell clusters
- Per-cluster differentially expressed genes from cluster.diffgenes (p < 0.05)
- SINCERA precompiled cell type-gene association table built from EBI Expression Atlas (https://www.ebi.ac.uk/gxa)

## Outputs

- Predicted cell type assignment per cluster
- Per-cell cluster and predicted cell-type mapping table (e.g. celltype_assignments.csv)
- celltype.validation consistency assessment of mapped cell types versus known marker expression

## How to apply

After cluster.assignment groups cells and cluster.diffgenes identifies differentially expressed genes per cluster (one-tailed Welch's t test or Wilcoxon test, p-value threshold 0.05), run celltype.enrichment, which performs a one-tailed Fisher's exact test of each cluster's gene set against SINCERA's precompiled cell type and gene association table derived from EBI Expression Atlas experimental expression data. Assign the predicted cell type per cluster from the enrichment results, then validate with celltype.validation, a rank-aggregation-based check of consistency between mapped cell types and known marker expression patterns, and inspect assignments visually with plotMarkers. In the demo datasets this recovers Epcam selectively expressed in the epithelial cluster and Pecam1 in the endothelial cluster, reproducing the major cell types reported in Figure 3.

## Related tools

- **SINCERA** (Implements celltype.enrichment (one-tailed Fisher's exact test against the EBI Expression Atlas cell type-gene table) and celltype.validation for cluster annotation) — github.com/xu-lab/SINCERA

## Examples

```
library(SINCERA); predicted <- celltype.enrichment(object); validated <- celltype.validation(object); write.csv(celltype.assignments(object), "celltype_assignments.csv")
```

## Evaluation signals

- Each predicted cell type is consistent with selectively expressed known marker genes (e.g. Epcam in epithelial, Pecam1 in endothelial clusters)
- celltype.validation shows consistency between mapped cell types and known marker expression patterns via rank aggregation
- plotMarkers confirms selective marker expression in the expected clusters, reproducing major cell types reported in Figure 3
- Enrichment significance derives from the one-tailed Fisher's exact test; per-cluster differential genes passed the p < 0.05 threshold from cluster.diffgenes
- Exported per-cell table contains both cluster membership and predicted cell type for every cell

## Limitations

- Annotation depends entirely on the precompiled cell type and gene association table from EBI Expression Atlas; cell types absent from that reference cannot be assigned
- Assignments are only as reliable as the upstream clustering and differential gene calls; errors in cluster.assignment or cluster.diffgenes propagate
- Housekeeping-like unselectively expressed genes must first be dropped via cluster.geneSelection (specificity threshold 0.7) or they may confound enrichment

## Evidence

- [methods] SINCERA has built a precompiled cell type and gene association table using experimental expression data obtained from EBI expression atlas: "SINCERA has built a precompiled cell type and gene association table using experimental expression data obtained from EBI expression atlas (https://www.ebi.ac.uk/gxa)."
- [methods] Run cluster.geneSelection function to select genes with a certain level of expression specificity for cell type identification.: "Run cluster.geneSelection function to select genes with a certain level of expression specificity for cell type identification."
- [readme] A demonstration of using SINCERA to analysis Human IPF and normal epithelial single cell data: "A demonstration of using SINCERA to analysis Human IPF and normal epithelial single cell data can be found at https://github.com/minzheguo/SINCERA/blob/master/demo/humanIPF.R."
- [abstract] The pipeline supports the analysis for the identification of major cell types, cell type-specific gene signatures: "The pipeline supports the analysis for the identification of major cell types, cell type-specific gene signatures, and driving forces of given cell types."
- [methods] Cell clustering and cell type identification is a key step in the pipeline and directly influences all downstream analysis.: "Cell clustering and cell type identification is a key step in the pipeline and directly influences all downstream analysis."
