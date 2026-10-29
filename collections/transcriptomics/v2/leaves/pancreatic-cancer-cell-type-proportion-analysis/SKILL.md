---
name: pancreatic-cancer-cell-type-proportion-analysis
description: Use when sample-level Leiden clusters (e.g., the PDAC 11 control / 14
  Tumor 1 / 10 Tumor 2 split at resolution 0.3) exist and you must test which cell
  types' proportions or marker genes separate them.
when_to_use_negative:
- Samples from distinct technologies or cell-isolation methods (droplet vs. well-based,
  single-nucleus vs. single-cell) are mixed — the article flags cross-technology sample
  comparison as an open challenge
- The goal is cell-level (not sample-level) clustering or pseudotime analysis — PILOT's
  statistical tests here are defined on patient/sample clusters, not individual cells
- Only one patient cluster is available (no between-cluster contrast is possible for
  the Welch t-test or the cluster-vs-rest limma contrast)
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0092
tools:
- name: PILOT
  role: Computes Wasserstein distances between samples and performs Leiden clustering
    (select_best_sil, clustering_emd) plus the limma-based DE contrast
  repo: https://github.com/CostaLab/PILOT
- name: POT library
  role: Provides the emd2 optimal-transport solver underlying PILOT's sample distance
    computation
  repo: https://github.com/CostaLab/PILOT
- name: limma
  role: Empirical-Bayes differential-expression contrast of ductal cell 2 and stellate
    cells against other clusters
- name: scanpy
  role: Loads/reprocesses the PDAC AnnData (read_h5ad) and provides the PCA embedding
    used for the cost matrix
- name: pandas
  role: Assembles cluster assignments, t-test results, and DE tables into pdac_fig5_reanalysis_results.csv
- name: Seurat
  role: Referenced uniform pre-processing pipeline for normalization and clustering
    of structures
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s44320-023-00003-8
    title: Detection of PatIent-Level distances from single cell genomics and pathomics
      data with Optimal Transport (PILOT)
  source_content_hash: sha256:56a44a4c588ed2de8059cdd59469cd181501be522743acc40d13569eaf0ca943
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CostaLab/PILOT
  tools:
  - PILOT
  - POT library
  - Seurat
  - limma
  - pandas
  - scanpy
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# pancreatic-cancer-cell-type-proportion-analysis

## Summary

Downstream statistical analysis of PILOT patient clusters on pancreatic ductal adenocarcinoma (PDAC) scRNA-seq (GSA CRA001160): Welch t-tests of cell-type proportions between detected clusters and limma empirical-Bayes differential-expression contrasts to identify the cell types and genes driving tumor subgroups.

## When to use

Use when sample-level Leiden clusters (e.g., the PDAC 11 control / 14 Tumor 1 / 10 Tumor 2 split at resolution 0.3) exist and you must test which cell types' proportions or marker genes separate them.

## When NOT to use

- Samples from distinct technologies or cell-isolation methods (droplet vs. well-based, single-nucleus vs. single-cell) are mixed — the article flags cross-technology sample comparison as an open challenge
- The goal is cell-level (not sample-level) clustering or pseudotime analysis — PILOT's statistical tests here are defined on patient/sample clusters, not individual cells
- Only one patient cluster is available (no between-cluster contrast is possible for the Welch t-test or the cluster-vs-rest limma contrast)

## Inputs

- Reprocessed PDAC scRNA-seq AnnData (derived from GSA CRA001160) with PCA embedding, cell-type, patient, and disease-status columns
- PILOT sample-to-cluster assignments at the optimal Leiden resolution (0.3, 3 clusters)
- Per-sample cell-type proportion matrix

## Outputs

- Welch t-test table of cell-type proportions per cluster (p < 0.05 flagged; ductal cell 1, ductal cell 2, stellate expected significant)
- limma empirical-Bayes gene-level differential-expression statistics for ductal cell 2 and stellate vs. other clusters
- pdac_fig5_reanalysis_results.csv combining cluster assignments, t-test results, and DE tables
- Saved cluster heatmap from pl.pl.clustering_emd

## How to apply

First run PILOT's Wasserstein-distance clustering on the reprocessed PDAC AnnData (from GSA CRA001160) using the PCA embedding, cell-type, patient, and disease-status columns; use pl.pl.select_best_sil to pick the optimal Leiden resolution (0.3 in the paper) and pl.pl.clustering_emd to obtain the 3 patient clusters. Then apply a Welch t-test (unequal variances) to each cell type's proportion across clusters, recording ductal cell 1, ductal cell 2, and stellate as significantly different at p < 0.05. Finally run PILOT's limma empirical-Bayes differential-expression contrast for ductal cell 2 and stellate cells against the other clusters and export gene-level statistics. Assemble cluster assignments, t-test results, and DE tables into a single results table (pdac_fig5_reanalysis_results.csv) for validation against the reported 11/14/10 split.

## Related tools

- **PILOT** (Computes Wasserstein distances between samples and performs Leiden clustering (select_best_sil, clustering_emd) plus the limma-based DE contrast) — https://github.com/CostaLab/PILOT
- **POT library** (Provides the emd2 optimal-transport solver underlying PILOT's sample distance computation) — https://github.com/CostaLab/PILOT
- **limma** (Empirical-Bayes differential-expression contrast of ductal cell 2 and stellate cells against other clusters)
- **scanpy** (Loads/reprocesses the PDAC AnnData (read_h5ad) and provides the PCA embedding used for the cost matrix)
- **pandas** (Assembles cluster assignments, t-test results, and DE tables into pdac_fig5_reanalysis_results.csv)
- **Seurat** (Referenced uniform pre-processing pipeline for normalization and clustering of structures)

## Examples

```
from scipy.stats import ttest_ind; stat, p = ttest_ind(proportions_cluster1, proportions_cluster2, equal_var=False)
```

## Evaluation signals

- Leiden clustering at resolution 0.3 reproduces the reported 3-cluster patient split: 11 control, 14 Tumor 1, 10 Tumor 2 (35 PDAC samples total)
- select_best_sil Silhouette Score curve peaks at Leiden resolution 0.3
- Welch t-test identifies ductal cell 1, ductal cell 2, and stellate as significantly different (p < 0.05) between clusters
- limma DE tables for ductal cell 2 and stellate are exported with gene-level statistics (contrast: cluster vs. all other clusters)
- pdac_fig5_reanalysis_results.csv contains all three components: cluster assignments, t-test results, and DE tables

## Limitations

- The Leiden resolution (0.3) and cluster count are data-dependent; validation requires the Silhouette Score curve, and the reported 11/14/10 split must be reproduced exactly
- PILOT does not explicitly correct for potential technical artefacts; it only provides statistical tests to evaluate their influence, so batch effects in the PDAC data are not modeled away
- Package versions for pilotpy, scanpy, pandas, seaborn, and matplotlib are not specified in the tutorial, risking environment-dependent results

## Evidence

- [methods] optimal resolution and clusters: "pl.pl.select_best_sil(adata, start = 0.2)"
- [discussion] leiden clustering on W: "Clustering analysis can be performed by providing W as input to a Leiden clustering algorithm (Traag et al, 2019)."
- [discussion] limma DE approach: [evidence span withheld — anchor status: not_found]
- [abstract] statistical testing of cell populations: "PILOT provides a statistical approach to find changes in cell populations, gene expression, and tissue structures related to the trajectories or clusters"
- [readme] tutorial for sub-group detection: [evidence span withheld — anchor status: external_doc]
