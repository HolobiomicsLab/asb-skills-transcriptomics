---
name: single-cell-dimensionality-reduction-and-clustering
description: Use when raw scRNA-seq count matrices (e.g., ZF1_F3_DrImpute) must be
  converted into cluster labels and state lineages that downstream lineage-tree methods
  like LinRace NJ_asym_Dif require as inputs.
when_to_use_negative:
- Input is already a cluster label vector or state lineage set rather than a raw count/expression
  matrix — the reduction and clustering step is redundant.
- The system has synchronous, fast differentiation so ancestral cell states are absent
  from the current-day scRNA-seq data; k-means states would not capture intermediates.
- Data quality is too low (sparse lineage barcodes / poor expression coverage), as
  LinRace assumes high-diversity, high-coverage barcodes for reliable state assignment.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2229
tools:
- name: R
  role: Environment in which PCA (prcomp), k-means, UMAP, and Slingshot are executed
  repo: https://github.com/ZhangLabGT/LinRace
- name: LinRace
  role: Consumes the cluster_id and state_lineages outputs; installed via devtools::install_github
  repo: https://github.com/ZhangLabGT/LinRace
- name: umap
  role: Embeds the top 50 PCs into 2D with min_dist = 0.3 for trajectory inference
- name: slingshot
  role: Runs on the UMAP layout with clusterLabels = cluster_id to derive state lineages
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-023-44173-3
    title: 'LinRace: cell division history reconstruction of single cells using paired
      lineage barcode and gene expression data'
  source_content_hash: sha256:e432005831bac110b2a6125c6541a1d8725895eeb6e2a6ac1a2aea4f75f06cf7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ZhangLabGT/LinRace
  tools:
  - Cassiopeia
  - DCLEAR
  - LinRace
  - LinTIMaT
  - R
  - ggplot2
  - phangorn
  - slingshot
  - umap
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5333333333333333
n_claims: 15
---

# single-cell-dimensionality-reduction-and-clustering

## Summary

Preprocesses paired single-cell gene expression data (e.g., zebrafish scGESTALT ZF1_F3_DrImpute counts) by scaling, PCA to 50 PCs, k-means clustering into 20 cell states, UMAP embedding (min_dist = 0.3), and Slingshot trajectory inference to derive state lineages used by LinRace.

## When to use

Use when raw scRNA-seq count matrices (e.g., ZF1_F3_DrImpute) must be converted into cluster labels and state lineages that downstream lineage-tree methods like LinRace NJ_asym_Dif require as inputs.

## When NOT to use

- Input is already a cluster label vector or state lineage set rather than a raw count/expression matrix — the reduction and clustering step is redundant.
- The system has synchronous, fast differentiation so ancestral cell states are absent from the current-day scRNA-seq data; k-means states would not capture intermediates.
- Data quality is too low (sparse lineage barcodes / poor expression coverage), as LinRace assumes high-diversity, high-coverage barcodes for reliable state assignment.

## Inputs

- zebrafish scGESTALT count matrix ZF1_F3_DrImpute (750 cells) expression matrix
- lineage character matrix cm_ZF1_F3 (read from cm_ZF1_F3.txt, sep = "\t", row.names = 1)
- UMAP layout coordinates for Slingshot

## Outputs

- PCA scores (top 50 PCs) from prcomp on scaled expression
- k-means cluster_id vector assigning each of the 750 cells to one of 20 cell states
- UMAP embedding (custom.config$min_dist = 0.3)
- Slingshot state lineages derived from the UMAP layout and cluster labels

## How to apply

Load the expression matrix (expr_ZF) and lineage character matrix (cm_ZF1_F3), then run prcomp on the scaled expression (pca_res <- prcomp(expr_ZF, scale. = TRUE)) and take the top 50 PCs. Cluster cells with k-means using centers = 20 to obtain cluster_id, which defines cell states for LinRace's asymmetric division model. Embed the same 50 PCs with UMAP, setting custom.config$min_dist = 0.3, then run Slingshot on the UMAP layout with clusterLabels = cluster_id to derive state lineages. These cluster labels and state lineages, together with the character matrix, are passed to NJ_asym_Dif with max_Iter = 500 for tree reconstruction.

## Related tools

- **R** (Environment in which PCA (prcomp), k-means, UMAP, and Slingshot are executed) — https://github.com/ZhangLabGT/LinRace
- **LinRace** (Consumes the cluster_id and state_lineages outputs; installed via devtools::install_github) — https://github.com/ZhangLabGT/LinRace
- **umap** (Embeds the top 50 PCs into 2D with min_dist = 0.3 for trajectory inference)
- **slingshot** (Runs on the UMAP layout with clusterLabels = cluster_id to derive state lineages)

## Examples

```
pca_res <- prcomp(expr_ZF, scale. = TRUE); cluster_id <- kmeans(pca_res$x[,1:50], centers = 20)$cluster; custom.config <- umap.config; custom.config$min_dist <- 0.3; umap_res <- umap(pca_res$x[,1:50], custom.config); sce <- slingshot(data = umap_res$layout, clusterLabels = cluster_id)
```

## Evaluation signals

- prcomp runs with scale. = TRUE and exactly the first 50 PCs are retained (pca_res$x[,1:50]) for both k-means and UMAP.
- k-means returns exactly 20 clusters (centers = 20) covering all 750 ZF1-F3 cells, and cluster_id is a per-cell label vector.
- UMAP is computed on the same 50-PC matrix with custom.config$min_dist = 0.3.
- Slingshot output (state lineages) must be compatible with NJ_asym_Dif, which is called as NJ_asym_Dif(as.data.frame(cm_ZF), cluster_id, expr_ZF, state_lineages, max_Iter = 500).
- Downstream LinRace tree on ZF1-F3 should show greater maximum depth and more internal nodes than Cassiopeia and LinTIMaT trees (Fig. 4b ordering), indicating the states fed in were usable.

## Limitations

- LinRace assumes cell state transitions are irreversible; reversible transitions are not modeled, so k-means states may misrepresent such dynamics.
- The method assumes all cell states during development exist in current-day cells; ancestral states reconstructed are limited to the 20 k-means states present in the scRNA-seq data.
- Software versions for the R packages (umap, ggplot2, slingshot) are not stated, complicating exact reproduction.

## Evidence

- [methods] PCA on zebrafish expression data (top 50 PCs): "pca_res <- prcomp(expr_ZF,scale. = TRUE)"
- [methods] kmeans clustering into 20 centers: "cluster_id <- kmeans(pca_res$x[,1:50], centers = 20)$cluster"
- [methods] UMAP embedding: "custom.config$min_dist = 0.3; umap_res <-umap(pca_res$x[,1:50],custom.config)"
- [methods] Slingshot trajectory inference on UMAP layout: "sce <- slingshot(data = umap_res$layout,clusterLabels = cluster_id)"
- [readme] LinRace application on ZF1-F3 sample: [evidence span withheld — anchor status: external_doc]
