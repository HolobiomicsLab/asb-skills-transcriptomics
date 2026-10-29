---
name: leiden-sample-level-clustering-silhouette-selection
description: Use when you have a sample-level distance matrix (e.g., PILOT's Wasserstein
  W over scRNA-seq samples in an AnnData with patient/disease columns) and need to
  detect unsupervised patient sub-groups and pick the Leiden resolution.
when_to_use_negative:
- Samples already carry labels or a known trajectory — PILOT's unsupervised clustering
  is intended for unlabeled sample-level comparison; prior label-based methods (PhEMD,
  SCANCell, Multi-scale PHATE) 'require labels of patients for their analysis and
  cannot be used in the unsupervised analysis'.
- The data mix distinct technologies or isolation methods (single nucleus vs. single
  cell, droplet vs. well based) — the article flags cross-technology analysis as 'still
  an open challenge', and PILOT assumes uniformly collected tissue samples.
- The goal is cell-level rather than sample-level clustering — this skill operates
  on the sample distance matrix W, not on individual cells.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_2229
tools:
- name: PILOT
  role: Provides pl.tl.wasserstein_distance to compute the sample distance matrix
    W, pl.pl.select_best_sil to sweep Leiden resolutions via Silhouette Score, and
    pl.pl.clustering_emd to cluster patients
  repo: https://github.com/CostaLab/PILOT
- name: POT library
  role: Implements the emd2 optimal-transport solver underlying PILOT's Wasserstein
    distance computation
  repo: https://github.com/CostaLab/PILOT
- name: scanpy
  role: Loads and preprocesses the reprocessed AnnData (e.g., sc.read_h5ad) with the
    PCA embedding used in the cost matrix
- name: pandas
  role: Assembles cluster assignments and downstream test results into CSV outputs
    such as pdac_fig5_reanalysis_results.csv
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s44320-023-00003-8
    title: Detection of PatIent-Level distances from single cell genomics and pathomics
      data with Optimal Transport (PILOT)
  source_content_hash: sha256:3c3fc01d38a9455f1be0ffe7b54e8bb9e56bbe8d4bab047c1a46f9e4ba76f4bc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CostaLab/PILOT
  tools:
  - PILOT
  - POT library
  - Seurat
  - pandas
  - scanpy
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# leiden-sample-level-clustering-silhouette-selection

## Summary

Unsupervised Leiden clustering of single-cell samples on PILOT's Wasserstein-distance matrix (W), with the Leiden resolution chosen by maximizing the Silhouette Score via PILOT's select_best_sil plot, to detect patient-level subgroups such as the PDAC 11-control/14-Tumor1/10-Tumor2 split.

## When to use

Use when you have a sample-level distance matrix (e.g., PILOT's Wasserstein W over scRNA-seq samples in an AnnData with patient/disease columns) and need to detect unsupervised patient sub-groups and pick the Leiden resolution.

## When NOT to use

- Samples already carry labels or a known trajectory — PILOT's unsupervised clustering is intended for unlabeled sample-level comparison; prior label-based methods (PhEMD, SCANCell, Multi-scale PHATE) 'require labels of patients for their analysis and cannot be used in the unsupervised analysis'.
- The data mix distinct technologies or isolation methods (single nucleus vs. single cell, droplet vs. well based) — the article flags cross-technology analysis as 'still an open challenge', and PILOT assumes uniformly collected tissue samples.
- The goal is cell-level rather than sample-level clustering — this skill operates on the sample distance matrix W, not on individual cells.

## Inputs

- AnnData (.h5ad) single-cell object with a PCA embedding and columns for cell type, patient/sample ID, and disease status
- Sample-level Wasserstein distance matrix W computed by PILOT (pl.tl.wasserstein_distance)
- Range of Leiden resolutions to sweep (e.g., starting at 0.2)

## Outputs

- Silhouette Score curve over candidate Leiden resolutions with the optimal resolution identified (0.3 for PDAC)
- Cluster heatmap of sample clusters (pl.pl.clustering_emd)
- Sample/patient cluster assignments (e.g., 11 control / 14 Tumor 1 / 10 Tumor 2)
- Cluster assignment table exportable to CSV (e.g., pdac_fig5_reanalysis_results.csv)

## How to apply

First run PILOT's Wasserstein distance computation (pl.tl.wasserstein_distance) on the reprocessed AnnData, supplying the PCA embedding, cell-type, patient, and disease-status columns, to obtain the sample-distance matrix W. Then run pl.pl.select_best_sil (e.g., with start=0.2) to sweep Leiden resolutions and plot the Silhouette Score curve, selecting the resolution at the maximum — for the PDAC data (GSA CRA001160) this is 0.3. Provide W to Leiden clustering (pl.pl.clustering_emd) at that resolution; for PDAC this recovers 3 patient clusters with 11 control samples, 14 samples as Tumor 1, and 10 as Tumor 2. Finally, validate the split against the reported composition and optionally characterize clusters by testing cell-type proportions (Welch t-test, p < 0.05) and limma empirical-Bayes differential expression.

## Related tools

- **PILOT** (Provides pl.tl.wasserstein_distance to compute the sample distance matrix W, pl.pl.select_best_sil to sweep Leiden resolutions via Silhouette Score, and pl.pl.clustering_emd to cluster patients) — https://github.com/CostaLab/PILOT
- **POT library** (Implements the emd2 optimal-transport solver underlying PILOT's Wasserstein distance computation) — https://github.com/CostaLab/PILOT
- **scanpy** (Loads and preprocesses the reprocessed AnnData (e.g., sc.read_h5ad) with the PCA embedding used in the cost matrix)
- **pandas** (Assembles cluster assignments and downstream test results into CSV outputs such as pdac_fig5_reanalysis_results.csv)

## Examples

```
pl.pl.select_best_sil(adata, start = 0.2); pl.pl.clustering_emd(adata)  # PDAC: Silhouette peaks at resolution 0.3 -> 3 patient clusters (11 control / 14 Tumor 1 / 10 Tumor 2)
```

## Evaluation signals

- The Silhouette Score curve (select_best_sil) peaks at the expected resolution — 0.3 for the PDAC data.
- Leiden clustering at that resolution reproduces the reported cluster count (3) and patient split (11 control / 14 Tumor 1 / 10 Tumor 2).
- Cluster heatmap from pl.pl.clustering_emd renders coherent sample blocks rather than fragmented clusters.
- Downstream Welch t-tests on cell-type proportions between clusters yield significant differences (p < 0.05) for expected cell types (ductal cell 1, ductal cell 2, stellate), confirming biologically distinct clusters.
- Every patient/sample in the input AnnData receives exactly one cluster assignment in the exported results CSV.

## Limitations

- PILOT does not explicitly correct for potential technical artefacts; it only provides statistical tests to evaluate their influence, and it is unclear how such artefacts affect comparison of disease groups.
- The analysis assumes tissue samples were collected uniformly (similar tissue areas, same technology); comparing data across distinct technologies (single nucleus vs. single cell) or isolation methods (droplet vs. well based) is an open challenge.
- Software/package versions for pilotpy, scanpy, pandas, seaborn, and matplotlib are not specified in the tutorial text, which may affect exact reproducibility of the resolution sweep.

## Evidence

- [methods] select_best_sil sweep: [evidence span withheld — anchor status: not_found]
- [discussion] Leiden on W: "Clustering analysis can be performed by providing W as input to a Leiden clustering algorithm (Traag et al, 2019)."
- [readme] PILOT sample-level clustering purpose: "This allows us to perform unsupervised analysis at the sample level and uncover trajectories or cellular clusters associated with disease progression."
- [discussion] cost matrix construction: "we generate a median representation for each cluster (in the PCA space) and estimate the cosine distance between the median value of clusters"
- [discussion] technical artifact caveat: "While PILOT does not explicitly correct for such potential technical artefacts, it provides statistical tests to evaluate if technical factors might influence the predictions."
