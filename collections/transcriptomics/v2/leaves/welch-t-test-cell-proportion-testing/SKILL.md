---
name: welch-t-test-cell-proportion-testing
description: Use when PILOT's Wasserstein-distance-based Leiden clustering has assigned
  patient samples to clusters (e.g., the PDAC 11 control / 14 Tumor 1 / 10 Tumor 2
  split) and you need to test which cell-type proportions differ between clusters.
when_to_use_negative:
- The goal is differential gene expression between clusters rather than cell-population
  changes — use PILOT's limma empirical-Bayes or Wald-statistic testing instead.
- 'Clusters have not yet been defined: the Welch t-test requires patient-level cluster
  labels (e.g., from select_best_sil followed by clustering_emd), not raw per-cell
  data.'
- Each cluster contains only one or very few samples, so between-cluster variance
  in proportions cannot be estimated reliably.
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_2222
tools:
- name: PILOT
  role: Provides the patient-level Wasserstein distance matrix, Leiden patient clusters,
    and per-cluster cell-population summaries that the Welch t-test is applied to
  repo: https://github.com/CostaLab/PILOT
- name: pandas
  role: Assembles cluster assignments, t-test results, and DE tables into the exported
    CSV of reanalysis results
- name: limma
  role: Downstream empirical-Bayes differential-expression contrast for the significant
    cell types identified by the proportion test (e.g., ductal cell 2, stellate)
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s44320-023-00003-8
    title: Detection of PatIent-Level distances from single cell genomics and pathomics
      data with Optimal Transport (PILOT)
  source_content_hash: sha256:efcc6d30c87dedef2993cee634d503180c63c389d3efbd2ecc4d195e284b7a0e
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
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_2222
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# welch-t-test-cell-proportion-testing

## Summary

Statistically compares cell-type composition between PILOT-detected patient clusters by applying a Welch t-test to per-cluster cell-type proportions, identifying which populations (e.g., ductal cell 1, ductal cell 2, stellate) differ significantly (p < 0.05) across patient subgroups.

## When to use

Use when PILOT's Wasserstein-distance-based Leiden clustering has assigned patient samples to clusters (e.g., the PDAC 11 control / 14 Tumor 1 / 10 Tumor 2 split) and you need to test which cell-type proportions differ between clusters.

## When NOT to use

- The goal is differential gene expression between clusters rather than cell-population changes — use PILOT's limma empirical-Bayes or Wald-statistic testing instead.
- Clusters have not yet been defined: the Welch t-test requires patient-level cluster labels (e.g., from select_best_sil followed by clustering_emd), not raw per-cell data.
- Each cluster contains only one or very few samples, so between-cluster variance in proportions cannot be estimated reliably.

## Inputs

- PILOT-annotated AnnData (.h5ad) with cell-type, patient, and disease-status columns
- Patient/sample cluster assignments from pl.pl.clustering_emd (Leiden clustering on the Wasserstein distance matrix W)
- Per-sample cell-type proportion (composition) matrix derived from the cell-type annotations

## Outputs

- Per-cell-type Welch t-test statistics and p-values comparing proportions between patient clusters
- List of significantly varying cell types (p < 0.05), e.g., ductal cell 1, ductal cell 2, stellate in PDAC
- Combined results table such as pdac_fig5_reanalysis_results.csv (cluster assignments + t-test results + DE tables)

## How to apply

After running pl.pl.clustering_emd to obtain patient clusters (in the PDAC case, three clusters at Leiden resolution 0.3 with 11 control / 14 Tumor 1 / 10 Tumor 2 samples), compute per-sample cell-type proportion vectors from the AnnData's cell-type column, then apply a Welch t-test (unequal variances) per cell type between samples in one cluster versus the others. Record which cell types pass p < 0.05 — in the PDAC reanalysis these were ductal cell 1, ductal cell 2, and stellate cells. This cell-population-level test complements gene-level contrast via limma's empirical-Bayes approach, which PILOT applies separately for significant cell types such as ductal cell 2 and stellate. Assemble the proportion test results with cluster assignments and DE tables into a single results table (e.g., pdac_fig5_reanalysis_results.csv) for downstream interpretation.

## Related tools

- **PILOT** (Provides the patient-level Wasserstein distance matrix, Leiden patient clusters, and per-cluster cell-population summaries that the Welch t-test is applied to) — https://github.com/CostaLab/PILOT
- **pandas** (Assembles cluster assignments, t-test results, and DE tables into the exported CSV of reanalysis results)
- **limma** (Downstream empirical-Bayes differential-expression contrast for the significant cell types identified by the proportion test (e.g., ductal cell 2, stellate))

## Evaluation signals

- The Leiden clustering at resolution 0.3 reproduces the reported 11 control / 14 Tumor 1 / 10 Tumor 2 patient split across the 3 clusters before testing proportions.
- Per-sample cell-type proportions sum to 1 within each patient sample before the Welch t-test is applied.
- Significant cell types must pass p < 0.05; in the PDAC case study these are ductal cell 1, ductal cell 2, and stellate cells.
- The Welch (unequal-variance) variant is used, not a Student t-test with pooled variance.
- The t-test results are coherently assembled with cluster assignments and limma DE tables in pdac_fig5_reanalysis_results.csv.

## Limitations

- PILOT does not explicitly correct for potential technical artefacts; it only provides statistical tests to evaluate their influence, so significant proportion differences may reflect batch effects.
- The analysis assumes tissue samples were collected uniformly (similar tissue areas, same technology); mixing technologies or isolation methods is an open challenge.
- The Welch t-test treats patient samples as independent within clusters and tests one cell type at a time; multiple-testing correction must be considered separately.

## Evidence

- [abstract] PILOT provides a statistical approach to find changes in cell populations, gene expression, and tissue structures related to the trajectories or clusters: "PILOT provides a statistical approach to find changes in cell populations, gene expression, and tissue structures related to the trajectories or clusters"
- [discussion] Regarding gene expression markers, PILOT uses limma's empirical Bayes approach: [evidence span withheld — anchor status: not_found]
- [readme] Moreover, PILOT provides a statistical approach to find changes in cell populations, gene expression, and tissue structures related to the trajectories or clusters: "Moreover, PILOT provides a statistical approach to find changes in cell populations, gene expression, and tissue structures related to the trajectories or clusters"
