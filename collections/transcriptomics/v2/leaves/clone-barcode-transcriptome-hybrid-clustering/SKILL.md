---
name: clone-barcode-transcriptome-hybrid-clustering
description: Use when single-cell count matrices are paired with clone-barcode assignments
  and you need clusters that weigh clonal origin alongside transcriptomic similarity,
  plus verification that cluster markers keep ROC AUC fidelity across α levels.
when_to_use_negative:
- Full-lineage tracing data (CRISPR-Cas9 genome editing or imaging-based family trees)
  — the study did not address these data types, though ClonoCluster could be adapted
  to weight distances by phylogenetic distance
- Systems where cell fate is determined extrinsically (e.g. cardiomyocyte-directed
  iPSCs from Jiang et al., low-dose vemurafenib) — high α reduces marker fidelity
  here, so clonal weighting is not beneficial
- Analysis goal is pure transcriptome-defined cell-type discovery with no barcode
  data available — ClonoCluster requires a table of cell IDs and assigned barcodes
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_0203
tools:
- name: ClonoCluster
  role: Core R package that linearly combines transcriptome nearest-neighbor edge
    weights with normalized clone-barcode weights via α to produce hybrid cluster
    assignments; provides Find_Markers_ROC for ROCR-based marker AUC analysis
  repo: https://github.com/leeprichman/ClonoCluster
- name: ClonoCluster_paper
  role: Raw data and analysis scripts (Run_full_analysis.R, Marker_analysis.R, Marker_turnover_hm.R)
    reproducing the AUC fidelity analysis across α levels
  repo: https://github.com/leeprichman/ClonoCluster_paper
- name: ROCR
  role: Computes ROC AUC of top cluster marker genes used for marker fidelity assessment
    at each α level
- name: pheatmap
  role: Renders heatmaps of top cluster marker AUC at each α level (marker fidelity
    heatmap)
- name: data.table
  role: Loads count matrices and barcode tables (fread) and manipulates cluster-assignment
    tables; dt2m converts data.table to matrix
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.xgen.2022.100247
    title: 'ClonoCluster: A method for using clonal origin to inform transcriptome
      clustering'
  source_content_hash: sha256:ac391b0f62497e4592efafb4dac1925aede8313996f48a4488529b9247070d3d
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
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# clone-barcode-transcriptome-hybrid-clustering

## Summary

Generates hybrid single-cell clusters by linearly combining transcriptome nearest-neighbor edge weights with clone-barcode weights using ClonoCluster's tunable parameter α (α=0 purely transcriptome, α=1 purely clone), then assesses whether hybrid clusters retain equivalent marker fidelity (median top-marker ROC AUC) versus transcriptome-only clustering.

## When to use

Use when single-cell count matrices are paired with clone-barcode assignments and you need clusters that weigh clonal origin alongside transcriptomic similarity, plus verification that cluster markers keep ROC AUC fidelity across α levels.

## When NOT to use

- Full-lineage tracing data (CRISPR-Cas9 genome editing or imaging-based family trees) — the study did not address these data types, though ClonoCluster could be adapted to weight distances by phylogenetic distance
- Systems where cell fate is determined extrinsically (e.g. cardiomyocyte-directed iPSCs from Jiang et al., low-dose vemurafenib) — high α reduces marker fidelity here, so clonal weighting is not beneficial
- Analysis goal is pure transcriptome-defined cell-type discovery with no barcode data available — ClonoCluster requires a table of cell IDs and assigned barcodes

## Inputs

- Normalized or scaled count matrix of barcoded cells (gene-by-cell, e.g. mycountmatrix.tsv from Data_genes/)
- Barcode table of unique cell IDs and assigned barcodes (e.g. mybarcodetable.tsv from Data_barcodes/)
- Cluster-assignment table from clonocluster run at each α value (e.g. *_cluster_assignments.txt)

## Outputs

- Hybrid cluster assignments per cell at every α value
- ROC-derived AUC values for top cluster markers at each α level (e.g. *_auc_linclust.txt / marker_fidelity_auc.tsv)
- Marker fidelity heatmap of top cluster marker AUC across α levels (pheatmap, as in Marker_turnover_hm.R / Figure S5A)

## How to apply

Load a normalized/scaled gene-by-cell count matrix (e.g. via data.table::fread, converting with dt2m) and a table of unique cell IDs with assigned barcodes, verifying cell IDs match between the two. Compute PCA (e.g. irlba_wrap with npc = 25) and run ClonoCluster across a gradient of α values — clonocluster(pca, bt, alpha = seq(0, 1, by = 0.1), beta = 0.1, res = 1) — where clone weights are 1/0 for shared/different barcodes normalized by barcode cell count, then linearly combined with transcriptome edge weights by α. Select a resolution that returns the desired transcriptome-only cluster number, and plot cluster count vs α to pick the maximum tolerable α. For each α level, run Find_Markers_ROC (ROCR-based) on the top marker gene of each cluster, compute per-α median top-marker AUC, and compare against the α = 0 transcriptome-only baseline; flag decreases such as cardiomyocyte-directed iPSCs (0.69/0.79 vs 0.94) and low-dose vemurafenib samples, where fate is extrinsically determined. Write per-cluster AUCs and per-α medians (e.g. marker_fidelity_auc.tsv) and render AUC heatmaps with pheatmap; analysis of marker fidelity is the recommended way to confirm the chosen α produces biologically meaningful hybrid clusters.

## Related tools

- **ClonoCluster** (Core R package that linearly combines transcriptome nearest-neighbor edge weights with normalized clone-barcode weights via α to produce hybrid cluster assignments; provides Find_Markers_ROC for ROCR-based marker AUC analysis) — https://github.com/leeprichman/ClonoCluster
- **ClonoCluster_paper** (Raw data and analysis scripts (Run_full_analysis.R, Marker_analysis.R, Marker_turnover_hm.R) reproducing the AUC fidelity analysis across α levels) — https://github.com/leeprichman/ClonoCluster_paper
- **ROCR** (Computes ROC AUC of top cluster marker genes used for marker fidelity assessment at each α level)
- **pheatmap** (Renders heatmaps of top cluster marker AUC at each α level (marker fidelity heatmap))
- **data.table** (Loads count matrices and barcode tables (fread) and manipulates cluster-assignment tables; dt2m converts data.table to matrix)

## Examples

```
clust <- clonocluster(pca, bt, alpha = seq(0, 1, by = 0.1), beta = 0.1, res = 1)
```

## Evaluation signals

- All computed ROC AUC values fall within [0.5, 1]
- Median top-marker AUC at hybrid α values is not significantly different from the transcriptome-only (α = 0) baseline for most datasets
- Expected exceptions reproduce: cardiomyocyte-directed iPSC (Jiang et al.) shows decreased fidelity at high α (0.69/0.79 vs 0.94), and low-dose vemurafenib samples show reduced fidelity
- Cell IDs in the barcode table exactly match those in the count matrix before clustering
- Cluster count vs α plot identifies a maximum tolerable α before cluster fragmentation; chosen α is confirmed by marker fidelity analysis

## Limitations

- No reliable method was found to identify which experimental systems have intrinsic determinants of cell fate and would benefit from clonal information
- In systems with extrinsically determined fate (cardiomyocyte-directed iPSCs, low-dose vemurafenib), high α values decrease marker fidelity
- Full-lineage tracing data (CRISPR genome editing, imaging-based family trees) were not addressed, though ClonoCluster could be adapted to weight by phylogenetic distance
- Further testing of homogeneity of functional properties within clusters is required to determine which clustering methods best match biological distinctions
- Only R version >= 3.6 is specified; version numbers for other software are not stated

## Evidence

- [abstract] tunable α weighting: "We developed ClonoCluster, a computational method that combines both clone and transcriptome information to create hybrid clusters that weight both kinds of data with a tunable parameter."
- [intro] α linear combination: "We linearly combined transcriptome and clone weights using α such that it returned purely transcriptome weights for α = 0 and purely clone weights for α = 1"
- [intro] clone weight definition: "For each edge between cells, we also created a “clone weight” of 1 or 0 based on whether the cells have the same or different barcodes."
- [abstract] marker fidelity equivalence: "The markers of these hybrid clusters were different but had equivalent fidelity to transcriptome-only clusters."
- [methods] ClonoCluster call: "clust <- clonocluster(pca, bt, alpha = seq(0, 1, by = 0.1), beta = 0.1, res = 1)"
- [discussion] marker fidelity for α selection: "Analysis of marker fidelity or comparison to ground truth data can be used to confirm that the α value produces hybrid clusters that are biologically meaningful"
- [discussion] extrinsic fate exception: "in the case of cardiomyocyte-directed iPSCs from Jiang et al.,18 in which we know that cell fate is largely determined by extrinsic factors"
- [readme] installation: "ClonoCluster is easily installed with the help of the `devtools` package"
