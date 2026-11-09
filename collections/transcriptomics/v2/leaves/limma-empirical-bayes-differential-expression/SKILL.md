---
name: limma-empirical-bayes-differential-expression
description: Use when PILOT has detected sample-level sub-groups (e.g. 3 PDAC clusters
  at resolution 0.3) and you need cell-type-specific genes distinguishing one cluster
  from the others, as a gene-level follow-up to proportion testing.
when_to_use_negative:
- You need a trajectory (pseudotime) analysis rather than cluster contrasts — PILOT
  uses Wald statistics on fitted curves along diffusion-map/EIPLGraph pseudotime for
  that setting
- Input is pathomics/tissue-structure data with no gene-expression matrix, so limma
  gene contrasts are not applicable
- You have not yet validated the sample clustering (e.g. the 11/14/10 patient split
  at resolution 0.3) — DE on unstable clusters is meaningless
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: limma
  role: Empirical-Bayes moderated differential-expression testing of cell-type-specific
    genes between one patient cluster and all other clusters
  repo: https://github.com/CostaLab/PILOT
- name: PILOT
  role: Computes the sample-level Wasserstein distances and Leiden clusters whose
    contrasts are tested with limma
  repo: https://github.com/CostaLab/PILOT
- name: pandas
  role: Assembles cluster assignments, t-test results and DE tables into the results
    CSV
- name: scanpy
  role: Handles the AnnData input and PCA embedding used by PILOT's Wasserstein distance
    step
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s44320-023-00003-8
    title: Detection of PatIent-Level distances from single cell genomics and pathomics
      data with Optimal Transport (PILOT)
  source_content_hash: sha256:601795a21a5b65bfa6f0df33fd28999da711a704f3873ee65066176161aaadd2
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
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# limma-empirical-bayes-differential-expression

## Summary

PILOT's marker-gene step: after Leiden clustering of samples on the Wasserstein-distance matrix, limma's empirical-Bayes framework (Ritchie et al, 2015) is used to contrast gene expression of a chosen cell type in one patient cluster against all other clusters. In the PDAC case study this yields gene-level statistics for ductal cell 2 and stellate cells across the 3 clusters found at Leiden resolution 0.3.

## When to use

Use when PILOT has detected sample-level sub-groups (e.g. 3 PDAC clusters at resolution 0.3) and you need cell-type-specific genes distinguishing one cluster from the others, as a gene-level follow-up to proportion testing.

## When NOT to use

- You need a trajectory (pseudotime) analysis rather than cluster contrasts — PILOT uses Wald statistics on fitted curves along diffusion-map/EIPLGraph pseudotime for that setting
- Input is pathomics/tissue-structure data with no gene-expression matrix, so limma gene contrasts are not applicable
- You have not yet validated the sample clustering (e.g. the 11/14/10 patient split at resolution 0.3) — DE on unstable clusters is meaningless

## Inputs

- Reprocessed scRNA-seq AnnData (PDAC data derived from GSA CRA001160) with PCA embedding, cell-type annotation, patient/sample and disease-status columns
- Sample-cluster assignments from PILOT Leiden clustering of the Wasserstein-distance matrix W
- Raw/normalized expression matrix per cell type for the limma contrast

## Outputs

- Gene-level differential-expression statistics (moderated logFC, p-values) per cell type for cluster-vs-other-cluster contrasts
- Exported DE tables and combined results CSV (cluster assignments + t-test results + DE tables)
- Saved cluster heatmap from pl.pl.clustering_emd

## How to apply

First run PILOT's sample-level pipeline: compute Wasserstein distances (pl.tl.wasserstein_distance) using the PCA embedding, cell-type, patient and disease-status columns, pick the Leiden resolution with pl.pl.select_best_sil (0.3 for the PDAC data), and obtain clusters with pl.pl.clustering_emd (expected 3 clusters: 11 control / 14 Tumor 1 / 10 Tumor 2). Then, per cell type of interest (ductal cell 2 and stellate in the PDAC study), fit limma's empirical-Bayes model contrasting that cluster's cells against cells from all other clusters and export the gene-level statistics (log fold-change and moderated p-values). Optionally pair the DE results with a Welch t-test on cell-type proportions between clusters (ductal cell 1, ductal cell 2, stellate differ at p < 0.05) to interpret which populations drive the cluster separation. Assemble cluster assignments, proportion-test results and DE tables into a single results file (e.g. pdac_fig5_reanalysis_results.csv).

## Related tools

- **limma** (Empirical-Bayes moderated differential-expression testing of cell-type-specific genes between one patient cluster and all other clusters) — https://github.com/CostaLab/PILOT
- **PILOT** (Computes the sample-level Wasserstein distances and Leiden clusters whose contrasts are tested with limma) — https://github.com/CostaLab/PILOT
- **pandas** (Assembles cluster assignments, t-test results and DE tables into the results CSV)
- **scanpy** (Handles the AnnData input and PCA embedding used by PILOT's Wasserstein distance step)

## Evaluation signals

- Leiden clustering at resolution 0.3 must reproduce the reported 3-cluster, 35-sample split: 11 control, 14 Tumor 1, 10 Tumor 2 before running the DE contrast
- Ductal cell 1, ductal cell 2 and stellate cell proportions must differ significantly between clusters (Welch t-test, p < 0.05), consistent with these being the DE-relevant populations
- DE tables must contain gene-level statistics (moderated statistics from limma) for ductal cell 2 and stellate cells in each cluster-vs-others contrast
- All cluster assignments, proportion-test results and DE tables must be assembled into pdac_fig5_reanalysis_results.csv

## Limitations

- PILOT does not explicitly correct for technical artefacts; it only provides statistical tests to evaluate their influence, so limma results may reflect batch/technology effects (e.g. across droplet- vs well-based isolation)
- Analysis assumes tissue samples were collected uniformly (similar tissue areas and same technology); cross-technology comparison is an open challenge
- Software/package versions for pilotpy, scanpy, pandas and related packages are not specified, complicating exact reproduction

## Evidence

- [discussion] PILOT uses limma's empirical Bayes approach for cell-type cluster contrast: [evidence span withheld — anchor status: not_found]
- [abstract] PILOT's statistical approach for cell population and gene expression changes related to clusters: "PILOT provides a statistical approach to find changes in cell populations, gene expression, and tissue structures related to the trajectories or clusters"
- [discussion] Leiden clustering on the sample distance matrix W provides the clusters used for contrasts: "Clustering analysis can be performed by providing W as input to a Leiden clustering algorithm (Traag et al, 2019)."
- [methods] Silhouette-based resolution selection precedes clustering in the tutorial workflow: "pl.pl.select_best_sil(adata, start = 0.2)"
