---
name: clonal-weighted-hybrid-clustering
description: Use when single-cell count data have paired clone-barcode assignments
  (cell ID to barcode table) and the goal is to test how weighting clonal origin reshapes
  transcriptome-only clusters, e.g. reducing entropy of known cell-type distributions
  across clusters.
when_to_use_negative:
- Cells lack clone-barcode information (transcriptome-only data), since α weighting
  of clone edges requires paired barcode assignments
- The experimental system's cell fate is determined by extrinsic rather than intrinsic
  (clonal) factors, where clonal weighting may not be biologically meaningful — no
  reliable method exists to identify which systems benefit from clonal information
- Full-lineage-tracing data (CRISPR-Cas9 genome editing or imaging-based family trees)
  is the input, since the study did not address phylogenetic-distance weighting, though
  ClonoCluster could be adapted
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: ClonoCluster
  role: Core R function (clonocluster) that linearly combines transcriptome and clone
    edge weights with tunable α (with beta = 0.1, res = 1) to produce hybrid clusters
  repo: https://github.com/leeprichman/ClonoCluster
- name: data.table
  role: Loads the count matrix and barcode table with fread, matching cell IDs between
    tables
- name: dt2m / irlba_wrap
  role: Converts the data.table count matrix to a matrix and computes PCA with npc
    = 25 as input to clonocluster
  repo: https://github.com/leeprichman/ClonoCluster
- name: cast_confusion
  role: Cross-tabulates known cell-type labels against cluster assignments at each
    α for concordance/entropy analysis
  repo: https://github.com/leeprichman/ClonoCluster
- name: entropy (R package)
  role: Computes Shannon entropy of cell-type label distributions within clusters
    for the Figure S7 comparison
- name: devtools
  role: Installs the ClonoCluster package via devtools::install_github("leeprichman/ClonoCluster")
  repo: https://github.com/leeprichman/ClonoCluster
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.xgen.2022.100247
    title: 'ClonoCluster: A method for using clonal origin to inform transcriptome
      clustering'
  source_content_hash: sha256:b907ca689de13ffeb485309cac6f88df27ec63d55a9ce3756dde0fd32114f338
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/leeprichman/ClonoCluster
  tools:
  - ClonoCluster
  - R (version >=)
  - cast_confusion
  - data.table
  - devtools
  - dt2m / irlba_wrap
  - entropy (R package)
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# clonal-weighted-hybrid-clustering

## Summary

Generates hybrid single-cell clusters by linearly combining transcriptome nearest-neighbor edge weights with clone-barcode weights using the tunable parameter α (α = 0 purely transcriptome, α = 1 purely clone) via the ClonoCluster R package, then assesses cluster purity/entropy against known cell-type labels.

## When to use

Use when single-cell count data have paired clone-barcode assignments (cell ID to barcode table) and the goal is to test how weighting clonal origin reshapes transcriptome-only clusters, e.g. reducing entropy of known cell-type distributions across clusters.

## When NOT to use

- Cells lack clone-barcode information (transcriptome-only data), since α weighting of clone edges requires paired barcode assignments
- The experimental system's cell fate is determined by extrinsic rather than intrinsic (clonal) factors, where clonal weighting may not be biologically meaningful — no reliable method exists to identify which systems benefit from clonal information
- Full-lineage-tracing data (CRISPR-Cas9 genome editing or imaging-based family trees) is the input, since the study did not address phylogenetic-distance weighting, though ClonoCluster could be adapted

## Inputs

- Normalized or scaled gene-by-cell count matrix (e.g. retrieved from a Seurat object)
- Table of unique cell IDs with assigned clone barcodes
- Optional known cell-type labels for validation (e.g. Weinreb et al. in vitro hematopoiesis types)

## Outputs

- Hybrid cluster assignments at α = 0 and low α (e.g. *_cluster_assignments.txt)
- Cluster-by-cell-type confusion matrix (cast_confusion output / Sankey alluvia)
- Shannon entropy values per cluster and mean entropy reduction vs α = 0 (e.g. entropy_comparison.csv)

## How to apply

Load the single-cell count matrix and barcode table with data.table::fread, ensuring cell IDs match, convert with dt2m, and compute PCA with irlba_wrap(cm, npc = 25). Run clonocluster(pca, bt, alpha = seq(0, 1, by = 0.1), beta = 0.1, res = 1), retaining assignments at α = 0 (purely transcriptome weights) and at a low α (hybrid weights). First identify the resolution value that returns the desired number of clusters from standard transcriptome-only clustering, then plot the number of clusters returned with increasing α to determine the maximum tolerable α. Cross-tabulate known cell-type labels (e.g. basophils, erythroid progenitors, lymphoid progenitors, mast cells, megakaryocytes, migratory and plasmacytoid dendritic cells in the in vitro hematopoiesis dataset) against cluster assignments using cast_confusion, compute Shannon entropy of the cell-type label distribution within each cluster at each α, and confirm marker fidelity or ground-truth agreement for the chosen α.

## Related tools

- **ClonoCluster** (Core R function (clonocluster) that linearly combines transcriptome and clone edge weights with tunable α (with beta = 0.1, res = 1) to produce hybrid clusters) — https://github.com/leeprichman/ClonoCluster
- **data.table** (Loads the count matrix and barcode table with fread, matching cell IDs between tables)
- **dt2m / irlba_wrap** (Converts the data.table count matrix to a matrix and computes PCA with npc = 25 as input to clonocluster) — https://github.com/leeprichman/ClonoCluster
- **cast_confusion** (Cross-tabulates known cell-type labels against cluster assignments at each α for concordance/entropy analysis) — https://github.com/leeprichman/ClonoCluster
- **entropy (R package)** (Computes Shannon entropy of cell-type label distributions within clusters for the Figure S7 comparison)
- **devtools** (Installs the ClonoCluster package via devtools::install_github("leeprichman/ClonoCluster")) — https://github.com/leeprichman/ClonoCluster

## Examples

```
devtools::install_github("leeprichman/ClonoCluster"); pca <- irlba_wrap(cm, npc = 25); clust <- clonocluster(pca, bt, alpha = seq(0, 1, by = 0.1), beta = 0.1, res = 1)
```

## Evaluation signals

- Every cluster at low α shows Shannon entropy no greater than its α = 0 counterpart, consistent with the low-α purity increase reported in Figure S7
- Entropy reduction is observed for several cell types (basophils, erythroid progenitors, lymphoid progenitors, mast cells, megakaryocytes, migratory and plasmacytoid dendritic cells) relative to transcriptome-only clustering
- The number of clusters returned at the chosen α is checked by plotting cluster number vs increasing α to identify the maximum tolerable α before excessive fragmentation
- Marker fidelity (ROC AUC via Find_Markers_ROC or Cohen's kappa vs α) confirms hybrid clusters are biologically meaningful at the chosen α
- Cell IDs match exactly between the count matrix and barcode table before clustering; entropy values are written to entropy_comparison.csv and are finite/non-negative

## Limitations

- Further testing of homogeneity of functional properties of cells within each cluster is required to determine which clustering methods most closely match biological distinctions
- No reliable method was found to identify which experimental systems have intrinsic determinants of cell fate and would benefit from clonal information; the appropriate transcriptome-vs-clone weighting may differ between endpoint and transitional systems and requires case-by-case evaluation
- Full-lineage-tracing data (CRISPR-Cas9 genome editing or imaging-based family trees) was not addressed, though ClonoCluster could be adapted to weight distances by phylogenetic distance; 'twin' analysis (barcoded twin cells in different environments) is a recommended critical control that was not part of the presented analyses

## Evidence

- [intro] We linearly combined transcriptome and clone weights using α such that it returned purely transcriptome weights for α = 0 and purely clone weights for α = 1: "We linearly combined transcriptome and clone weights using α such that it returned purely transcriptome weights for α = 0 and purely clone weights for α = 1"
- [methods] clonocluster call with npc 25 PCA: "pca <- irlba_wrap(cm, npc = 25)"
- [methods] clonocluster function invocation: "clust <- clonocluster(pca, bt, alpha = seq(0, 1, by = 0.1), beta = 0.1, res = 1)"
- [discussion] optimal alpha selection via cluster count and fidelity: "first identifying the resolution value that returns the desired number of clusters from standard transcriptome-only clustering"
- [readme] README installation and input requirements: "To get started, you will need:  1. A normalized or scaled count matrix of your barcoded cells  2. A table of unique cell IDs and their assigned barcodes"
- [readme] entropy figure S7 analysis script: [evidence span withheld — anchor status: external_doc]
