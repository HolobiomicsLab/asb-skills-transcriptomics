---
name: marker-fidelity-roc-analysis
description: Use when you have hybrid cluster assignments from ClonoCluster at multiple
  α values and need to verify that cluster markers keep equivalent fidelity (median
  top-marker AUC) versus the α = 0 transcriptome-only baseline.
when_to_use_negative:
- You only need standard transcriptome-only cluster markers with no clonal barcode
  data or α comparison — use Seurat/scanpy marker finding directly.
- Your goal is to identify reorganization markers between paired transcriptome and
  hybrid clusters (a different analysis computed as *_auc_barcodes.txt), not top-marker
  AUC fidelity.
- Cell fate is not barcoded and no clone weight information exists, so hybrid clustering
  at varying α cannot be defined.
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_2229
tools:
- name: ClonoCluster
  role: Provides the hybrid cluster assignments across α levels whose top markers
    are scored for AUC fidelity; includes Find_Markers_ROC for cluster marker analysis
    with ROCR
  repo: https://github.com/leeprichman/ClonoCluster
- name: ROCR
  role: Computes ROC AUC of the top marker gene for each cluster within Find_Markers_ROC
- name: pheatmap
  role: Renders the marker fidelity heatmap of top cluster marker AUC at each α level
- name: data.table
  role: Loads the count matrix and barcode tables (fread) and manages the cluster-assignment
    tables
- name: ClonoCluster_paper
  role: Analysis scripts and raw data reproducing the marker fidelity figures (Marker_analysis.R,
    Marker_turnover_hm.R)
  repo: https://github.com/arjunrajlaboratory/ClonoCluster_paper
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.xgen.2022.100247
    title: 'ClonoCluster: A method for using clonal origin to inform transcriptome
      clustering'
  source_content_hash: sha256:4eaa7826fe359d1b156880f5398a7116ab57b414338af3a5631a9bea2435c904
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/leeprichman/ClonoCluster
  tools:
  - ClonoCluster
  - ClonoCluster_paper
  - R (version >=)
  - ROCR
  - data.table
  - pheatmap
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# marker-fidelity-roc-analysis

## Summary

Quantifies whether top markers of ClonoCluster hybrid clusters retain the sensitivity/specificity (ROC AUC) of transcriptome-only cluster markers as α shifts weight toward clonal origin, using Find_Markers_ROC and ROCR across α = 0 to 1.

## When to use

Use when you have hybrid cluster assignments from ClonoCluster at multiple α values and need to verify that cluster markers keep equivalent fidelity (median top-marker AUC) versus the α = 0 transcriptome-only baseline.

## When NOT to use

- You only need standard transcriptome-only cluster markers with no clonal barcode data or α comparison — use Seurat/scanpy marker finding directly.
- Your goal is to identify reorganization markers between paired transcriptome and hybrid clusters (a different analysis computed as *_auc_barcodes.txt), not top-marker AUC fidelity.
- Cell fate is not barcoded and no clone weight information exists, so hybrid clustering at varying α cannot be defined.

## Inputs

- normalized/scaled gene-by-cell count matrix (e.g. mycountmatrix.tsv, from Data_genes/ in ClonoCluster_paper release)
- barcode assignment table mapping unique cell IDs to barcodes (e.g. mybarcodetable.tsv)
- cluster-assignment table from clonocluster() containing one cluster per cell at every α value (e.g. *_cluster_assignments.txt)

## Outputs

- per-cluster ROC AUC values for top marker genes at each α level (*_auc_linclust.txt)
- per-α median top-marker AUC table (marker_fidelity_auc.tsv)
- marker fidelity heatmap of top cluster marker AUC across α levels (pheatmap, cf. Turnover_hm plots)

## How to apply

Load the gene-by-cell count matrix (e.g. mycountmatrix.tsv) and barcode table (mybarcodetable.tsv) with data.table::fread, confirming cell IDs match, and load cluster assignments produced by clonocluster(pca, bt, alpha = seq(0, 1, by = 0.1), beta = 0.1, res = 1). For each dataset and each α level (α = 0, low α ≈ 0.1–0.3, high α ≈ 0.6–0.9, α = 1), run Find_Markers_ROC(dl, cm), which uses ROCR to compute the ROC AUC of the top marker gene for each cluster, then compute the median top-marker AUC per α level. Compare these medians to the reported values (e.g. cardiomyocyte-directed iPSC 0.69/0.79 vs 0.94; low-dose vemurafenib 0.80/0.84 vs 0.90), flagging decreased fidelity in extrinsically determined systems. Write per-cluster AUCs and per-α medians to marker_fidelity_auc.tsv and render a marker fidelity heatmap with pheatmap (as in the paper's Marker_turnover_hm.R Figure S5A-style output).

## Related tools

- **ClonoCluster** (Provides the hybrid cluster assignments across α levels whose top markers are scored for AUC fidelity; includes Find_Markers_ROC for cluster marker analysis with ROCR) — https://github.com/leeprichman/ClonoCluster
- **ROCR** (Computes ROC AUC of the top marker gene for each cluster within Find_Markers_ROC)
- **pheatmap** (Renders the marker fidelity heatmap of top cluster marker AUC at each α level)
- **data.table** (Loads the count matrix and barcode tables (fread) and manages the cluster-assignment tables)
- **ClonoCluster_paper** (Analysis scripts and raw data reproducing the marker fidelity figures (Marker_analysis.R, Marker_turnover_hm.R)) — https://github.com/arjunrajlaboratory/ClonoCluster_paper

## Examples

```
clust <- clonocluster(pca, bt, alpha = seq(0, 1, by = 0.1), beta = 0.1, res = 1); auc <- Find_Markers_ROC(dl, cm)
```

## Evaluation signals

- All computed AUC values fall within [0.5, 1]
- Median top-marker AUC at hybrid α values is comparable to the transcriptome-only (α = 0) baseline for most datasets
- Cardiomyocyte-directed iPSC (CJ, Jiang et al.) dataset shows decreased marker fidelity at high α/clonal levels (0.69 and 0.79 vs 0.94), and low-dose vemurafenib shows 0.80/0.84 vs 0.90
- Marker fidelity heatmaps reproduce the pattern in Figure S5A: AUC of the union of top cluster markers remains stable across α except in extrinsically determined systems
- Cell IDs in the barcode table exactly match those in the count matrix before scoring

## Limitations

- Marker fidelity is decreased for systems where cell fate is largely extrinsically determined, e.g. cardiomyocyte-directed iPSCs (Jiang et al.) and low-dose vemurafenib samples
- Further testing of homogeneity of functional properties within each cluster is required to determine which clustering methods best match biological distinctions
- No reliable method was found to identify which experimental systems have intrinsic determinants of cell fate and would benefit from clonal information

## Evidence

- [results] Median AUC fidelity across α levels: [evidence span withheld — anchor status: not_found]
- [results] Decreased fidelity exception: [evidence span withheld — anchor status: not_found]
- [methods] Find_Markers_ROC tool: "Cluster marker analysis with `Find_Markers_ROC`"
- [methods] clonocluster call: "clust <- clonocluster(pca, bt, alpha = seq(0, 1, by = 0.1), beta = 0.1, res = 1)"
- [discussion] Confirm α with marker fidelity: "Analysis of marker fidelity or comparison to ground truth data can be used to confirm that the α value produces hybrid clusters that are biologically meaningful"
- [abstract] Equivalent fidelity finding: "The markers of these hybrid clusters were different but had equivalent fidelity to transcriptome-only clusters."
