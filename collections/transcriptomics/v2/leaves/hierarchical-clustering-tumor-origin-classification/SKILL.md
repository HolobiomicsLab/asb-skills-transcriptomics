---
name: hierarchical-clustering-tumor-origin-classification
description: Use when scRNA-Seq cells from multiple tumor samples should be grouped
  by tumor of origin, and you need to test whether removing the cell-cycle effect
  (scLVM/ccRemover) raises clustering accuracy over uncorrected data.
when_to_use_negative:
- The goal is to identify cell types rather than recover tumor-of-origin structure
  — ccRemover preserves biological signals of interest, so cell-type distinctions
  are meant to survive correction; this benchmark is about tumor identity only.
- The dataset has high cell-cycle activity — glioblastoma specifically warrants the
  reduced cutoff CΔ = 2 'because cell-cycle activity there is known to be very low';
  other datasets use the default cutoff CΔ = 3.
- scLVM's leading factor does not represent the cell cycle — the article notes scLVM
  'has a high risk of removing other biological components of interest' without a
  statistical test, so its corrected output here is a comparator, not the primary
  correction.
edam_operation: http://edamontology.org/operation_0315
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: ccRemover
  role: Produces the ccRemover-corrected expression matrix with cutoff CΔ=2 for the
    low-cell-cycle-activity glioblastoma data before hierarchical clustering
  repo: https://github.com/cran/ccRemover
- name: scLVM
  role: Produces the comparator corrected matrix via the Python script from the scLVM
    GitHub page, with K=1 and scree-plot K tried
- name: Biomart
  role: Downloads the lists of human genes annotated to the cell cycle used to build
    the if_cc indicator
- name: org.Hs.eg.db
  role: Human annotation database supplying the cell-cycle gene set for ccRemover
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/srep33892
    title: Identifying and removing the cell-cycle effect from single-cell RNA-Sequencing
      data
  source_content_hash: sha256:a0f69b3a1312253501782a853cc974861a7346d900dc17896ca35562760bb866
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cran/ccRemover
  tools:
  - Biomart
  - ccRemover
  - org.Hs.eg.db
  - scLVM
  edam_operation: http://edamontology.org/operation_0315
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# hierarchical-clustering-tumor-origin-classification

## Summary

Benchmarks whether cell-cycle correction (scLVM or ccRemover) improves hierarchical clustering of Patel et al. glioblastoma cells (430 cells, five tumors MGH26-MGH31, GSE57872) by tumor of origin, comparing accuracies against the uncorrected original data.

## When to use

Use when scRNA-Seq cells from multiple tumor samples should be grouped by tumor of origin, and you need to test whether removing the cell-cycle effect (scLVM/ccRemover) raises clustering accuracy over uncorrected data.

## When NOT to use

- The goal is to identify cell types rather than recover tumor-of-origin structure — ccRemover preserves biological signals of interest, so cell-type distinctions are meant to survive correction; this benchmark is about tumor identity only.
- The dataset has high cell-cycle activity — glioblastoma specifically warrants the reduced cutoff CΔ = 2 'because cell-cycle activity there is known to be very low'; other datasets use the default cutoff CΔ = 3.
- scLVM's leading factor does not represent the cell cycle — the article notes scLVM 'has a high risk of removing other biological components of interest' without a statistical test, so its corrected output here is a comparator, not the primary correction.

## Inputs

- Glioblastoma scRNA-Seq expression matrix (GSE57872; 430 cells from five tumors MGH26-MGH31), log-transformed and gene-by-gene centered
- Human cell-cycle gene annotation set (if_cc indicator vector) from Biomart/org.Hs.eg.db
- Corrected expression matrices: scLVM-corrected (K=1 and scree-plot K) and ccRemover-corrected (cutoff CΔ=2)

## Outputs

- Five-cluster hierarchical clustering assignment of cells per data version, each cluster labeled with its majority tumor-of-origin class
- Tumor-of-origin clustering accuracy per version (target: 87.44% original, 90.00% scLVM, 92.32% ccRemover)
- clustering_accuracies.csv with the three accuracies for comparison against the paper

## How to apply

Download the Patel et al. glioblastoma matrix (GSE57872), log-transform, and center it on a gene-by-gene basis. Build the if_cc indicator using human cell-cycle gene annotations retrieved from Biomart/org.Hs.eg.db, treating all non-annotated genes as control genes. Produce corrected versions with scLVM (trying default K=1 and the scree-plot value) and ccRemover with the reduced cutoff CΔ = 2 prescribed for glioblastoma because cell-cycle activity there is very low (ccRemover(dat, cutoff=2, max_it=4, nboot=200, ntop=10)). Apply hierarchical clustering to each of the three matrices (original, scLVM-corrected, ccRemover-corrected), splitting the cells into five clusters and assigning each cluster the majority tumor-of-origin class. Compare the resulting accuracies against the reported 87.44% (original), 90.00% (scLVM), and 92.32% (ccRemover).

## Related tools

- **ccRemover** (Produces the ccRemover-corrected expression matrix with cutoff CΔ=2 for the low-cell-cycle-activity glioblastoma data before hierarchical clustering) — https://github.com/cran/ccRemover
- **scLVM** (Produces the comparator corrected matrix via the Python script from the scLVM GitHub page, with K=1 and scree-plot K tried)
- **Biomart** (Downloads the lists of human genes annotated to the cell cycle used to build the if_cc indicator)
- **org.Hs.eg.db** (Human annotation database supplying the cell-cycle gene set for ccRemover)

## Examples

```
gene_cycle_gene_indices <- gene_indexer(gene_names, species = "human", name_type = "symbols"); xhat <- ccRemover(dat, cutoff = 2, max_it = 4, nboot = 200, ntop = 10, bar = FALSE)
```

## Evaluation signals

- Original-data accuracy reproduces at ~87.44% and corrected accuracies fall within ±2 percentage points of 90.00% (scLVM) and 92.32% (ccRemover) in clustering_accuracies.csv
- ccRemover-corrected accuracy strictly exceeds the original-data accuracy (92.32% > 87.44%), confirming the correction helps rather than harms tumor-of-origin clustering
- scLVM-corrected accuracy (90.00%) exceeds original but falls below ccRemover-corrected, matching the ordering reported in the paper
- Exactly five clusters are produced per data version (matching the five tumors MGH26-MGH31), and each cluster is assigned its majority tumor-of-origin class
- The ccRemover call uses the reduced cutoff CΔ = 2 (not the default 3), consistent with the paper's prescription for glioblastoma's very low cell-cycle activity

## Limitations

- ccRemover's bootstrap-based test may miss the cell-cycle effect if it influences too few control genes (below ~8% affected), so the correction — and hence the accuracy gain — can fail silently
- Annotation databases are always incomplete and inaccurate; the quality of the if_cc indicator constrains which components ccRemover can flag as cell-cycle effects
- ccRemover removes only the cell-cycle effect; other confounding effects may dominate the corrected data afterward and are not addressed
- scLVM does not inspect whether a leading factor represents the cell cycle, risking removal of tumor-of-origin structure instead of cell-cycle structure

## Evidence

- [results] Hierarchical clustering was applied to the (original, scLVM corrected, and ccRemover corrected) data, splitting the cells into five clusters, with each cluster being assigned the class of the: "Hierarchical clustering was applied to the (original, scLVM corrected, and ccRemover corrected) data, splitting the cells into five clusters, with each cluster being assigned the class of the"
- [methods] Low cell-cycle activity cutoff: "except for the glioblastoma data, where it is known that the cell-cycle activity is at a very low level47. We use a smaller cutoff value CΔ = 2"
- [results] scLVM K choice: "For the choice of K (the number of leading factors to be removed) in scLVM, we try both the default value K = 1 and the value given by the scree plot"
- [methods] Preprocessing recommendation: "We recommend transforming the data to a log scale and centering it on a gene-by-gene basis."
- [results] Cell-cycle gene annotation source: "Firstly Biomart was used to download lists of genes that were annotated to the cell cycle"
