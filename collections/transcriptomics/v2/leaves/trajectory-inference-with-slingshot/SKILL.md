---
name: trajectory-inference-with-slingshot
description: Use when paired lineage-barcode and scRNA-seq data need a cell-state
  tree (state_lineages) over k-means clusters of top-50 PCs, so a lineage-reconstruction
  method like LinRace can map mutations onto state transitions.
when_to_use_negative:
- Input lacks paired scRNA-seq expression data (barcode-only data), since the state
  lineages must be derived from expression clusters on the UMAP layout
- Cell states are expected to be transient/absent from the final sample (fast-changing
  systems), so ancestral states reconstructed from current-day clusters would be incomplete
- Differentiation is largely reversible — LinRace assumes irreversible cell state
  transitions, so Slingshot-ordered lineages would misrepresent reversible state changes
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: slingshot
  role: Trajectory inference over the k-means clusters on the UMAP layout to produce
    state_lineages
- name: umap
  role: R umap package embedding of the top 50 PCs (min_dist = 0.3) that provides
    the 2D layout Slingshot runs on
- name: LinRace
  role: 'Downstream consumer: NJ_asym_Dif uses the Slingshot-derived state lineages,
    cluster_id, character matrix and expression to reconstruct the lineage tree'
  repo: https://github.com/ZhangLabGT/LinRace
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-023-44173-3
    title: 'LinRace: cell division history reconstruction of single cells using paired
      lineage barcode and gene expression data'
  source_content_hash: sha256:6a998ed3fedc2dac0d73fa8abf42d95e5323a4341d65e1bd9b429325add92403
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
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 15
---

# trajectory-inference-with-slingshot

## Summary

Derives cell-state lineages (pseudotime trajectories over clusters) from a UMAP embedding of the zebrafish scGESTALT expression data using R/Slingshot; these state lineages become the state-transition prior that LinRace's NJ_asym_Dif uses when reconstructing the cell division tree.

## When to use

Use when paired lineage-barcode and scRNA-seq data need a cell-state tree (state_lineages) over k-means clusters of top-50 PCs, so a lineage-reconstruction method like LinRace can map mutations onto state transitions.

## When NOT to use

- Input lacks paired scRNA-seq expression data (barcode-only data), since the state lineages must be derived from expression clusters on the UMAP layout
- Cell states are expected to be transient/absent from the final sample (fast-changing systems), so ancestral states reconstructed from current-day clusters would be incomplete
- Differentiation is largely reversible — LinRace assumes irreversible cell state transitions, so Slingshot-ordered lineages would misrepresent reversible state changes

## Inputs

- ZF1_F3_DrImpute zebrafish scGESTALT gene expression count matrix (750 cells)
- cm_ZF1_F3 lineage character matrix
- PCA loadings (top 50 PCs of the scaled expression matrix)
- k-means cluster assignments (centers = 20)
- UMAP embedding of the top 50 PCs (min_dist = 0.3)

## Outputs

- Slingshot state lineages (ordered cluster lineages over the UMAP layout)
- cluster_id vector used as cell-state labels for the LinRace tree
- state_lineages input consumed by LinRace NJ_asym_Dif for tree reconstruction and ancestral state inference

## How to apply

Scale the ZF1_F3_DrImpute expression matrix and run PCA with prcomp(expr_ZF, scale. = TRUE), keeping the first 50 PCs. Cluster cells with kmeans(pca_res$x[,1:50], centers = 20)$cluster to obtain cluster_id, then embed the same top-50 PCs with umap using a custom.config with min_dist = 0.3. Run Slingshot on the UMAP layout with the k-means cluster labels as clusterLabels to infer Slingshot lineages over the cell states. Pass these state_lineages together with the cm_ZF1_F3 character matrix, cluster_id, and expression matrix into LinRace's NJ_asym_Dif (max_Iter = 500), so the lineage tree respects inferred state-transition paths rather than arbitrary cluster merges. The rationale is that Slingshot's cluster-ordering constraints encode differentiation directions that the asymmetric cell division model in LinRace can exploit to infer ancestral states.

## Related tools

- **slingshot** (Trajectory inference over the k-means clusters on the UMAP layout to produce state_lineages)
- **umap** (R umap package embedding of the top 50 PCs (min_dist = 0.3) that provides the 2D layout Slingshot runs on)
- **LinRace** (Downstream consumer: NJ_asym_Dif uses the Slingshot-derived state lineages, cluster_id, character matrix and expression to reconstruct the lineage tree) — https://github.com/ZhangLabGT/LinRace

## Examples

```
sce <- slingshot(data = umap_res$layout, clusterLabels = cluster_id)
```

## Evaluation signals

- clustering yields exactly 20 k-means clusters (centers = 20) on the top 50 PCs
- UMAP config has min_dist = 0.3 and uses the same pca_res$x[,1:50] as the clustering
- Slingshot is run on umap_res$layout with clusterLabels = cluster_id, producing lineage paths consistent with the observed cluster structure
- Downstream check: the restored-tips LinRace tree (NJ_asym_Dif, max_Iter = 500) shows greater maximum depth and more internal nodes than Cassiopeia and LinTIMaT trees on ZF1-F3 (Fig. 4b ordering)
- Ancestral states inferred on the tree fall only within the cell states present in the scRNA-seq data

## Limitations

- LinRace assumes irreversible cell state transitions; reversible transitions are not modeled
- The analysis assumes all cell states during development exist in current-day cells; LinRace is most applicable to systems with asynchronous differentiation
- Ancestral cell state reconstruction is restricted to cell states present in the scRNA-seq data; fast-changing systems may have uncaptured ancestral/intermediate states
- The number of k-means centers (20) and UMAP min_dist (0.3) are fixed pipeline choices; software versions for the R packages are not stated

## Evidence

- [methods] Slingshot trajectory inference on UMAP layout: "sce <- slingshot(data = umap_res$layout,clusterLabels = cluster_id)"
- [methods] UMAP embedding of top 50 PCs: "custom.config$min_dist = 0.3; umap_res <-umap(pca_res$x[,1:50],custom.config)"
- [methods] k-means clustering feeding cluster labels: "cluster_id <- kmeans(pca_res$x[,1:50], centers = 20)$cluster"
- [methods] LinRace consumes state lineages in tree building: "tree_linrace <- NJ_asym_Dif(as.data.frame(cm_ZF),cluster_id,expr_ZF,state_lineages,max_Iter = 500)"
- [readme] README confirms the ZF1-F3 example application: [evidence span withheld — anchor status: external_doc]
