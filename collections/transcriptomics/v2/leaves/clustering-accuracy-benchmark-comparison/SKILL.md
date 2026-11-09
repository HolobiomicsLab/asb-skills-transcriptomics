---
name: clustering-accuracy-benchmark-comparison
description: Use when a scRNA-Seq dataset has known per-cell labels (e.g. tumor of
  origin MGH26-MGH31, cell type, replicate) and you must quantify whether a correction
  method such as ccRemover or scLVM improves cluster-vs-label agreement over uncorrected
  data.
when_to_use_negative:
- The number of true underlying groups and per-cell labels are unknown, so majority-class
  accuracy cannot be computed (use internal cluster-quality metrics instead).
- The cell-cycle state is itself the biological signal of interest — removing it (via
  scLVM or ccRemover) would erase the structure being benchmarked.
- Data has already been corrected or the clustering algorithm differs between the
  original and corrected matrices, making the accuracy comparison confounded.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: ccRemover
  role: Produces the corrected matrix expected to give the largest clustering-accuracy
    gain; run with cutoff = 2 for low cell-cycle-activity glioblastoma data
  repo: cran/ccRemover
- name: scLVM
  role: Baseline latent-factor correction (K = 1 and scree-plot K) whose clustering
    accuracy is compared against ccRemover and original data
- name: org.Hs.eg.db
  role: Human cell-cycle gene annotations used to build the if_cc indicator vector
    for ccRemover
- name: Biomart
  role: Source of cell-cycle-annotated gene lists used to mark cell-cycle genes
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/srep33892
    title: Identifying and removing the cell-cycle effect from single-cell RNA-Sequencing
      data
  source_content_hash: sha256:3d9afd37ce8f052aa804ef744520e4975384bfd42e0c55806fafb1a42e3927ff
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cran/ccRemover
  tools:
  - Biomart
  - ccRemover
  - org.Hs.eg.db
  - scLVM
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# clustering-accuracy-benchmark-comparison

## Summary

Benchmarks cell-cycle correction methods (scLVM vs ccRemover) by running the same clustering algorithm on original and corrected scRNA-Seq matrices and measuring clustering accuracy against known labels, e.g. tumor-of-origin for the Patel et al. glioblastoma data (87.44% original, 90.00% scLVM, 92.32% ccRemover).

## When to use

Use when a scRNA-Seq dataset has known per-cell labels (e.g. tumor of origin MGH26-MGH31, cell type, replicate) and you must quantify whether a correction method such as ccRemover or scLVM improves cluster-vs-label agreement over uncorrected data.

## When NOT to use

- The number of true underlying groups and per-cell labels are unknown, so majority-class accuracy cannot be computed (use internal cluster-quality metrics instead).
- The cell-cycle state is itself the biological signal of interest — removing it (via scLVM or ccRemover) would erase the structure being benchmarked.
- Data has already been corrected or the clustering algorithm differs between the original and corrected matrices, making the accuracy comparison confounded.

## Inputs

- Gene-by-cell scRNA-Seq expression matrix with per-cell ground-truth labels (e.g. GSE57872, 430 cells from five glioblastomas MGH26-MGH31)
- Original (uncorrected) log-transformed, gene-by-gene centered matrix
- scLVM-corrected matrix (Python scLVM script, K = 1 and scree-plot K)
- ccRemover-corrected matrix (if_cc cell-cycle gene indicator, cutoff = 2 for glioblastoma)
- clustering_accuracies.csv with paper-reported accuracies (87.44%, 90.00%, 92.32%)

## Outputs

- Clustering accuracy (%) for each of the original, scLVM-corrected, and ccRemover-corrected matrices
- Five-cluster assignment of cells per data version with majority tumor-of-origin class per cluster
- Comparison table showing accuracy gain of each correction over the original data

## How to apply

Prepare each data version identically: log-transform and center the expression matrix on a gene-by-gene basis, then produce the original matrix, a scLVM-corrected matrix (try default K = 1 and the value indicated by the scree plot), and a ccRemover-corrected matrix built with the if_cc cell-cycle indicator from Biomart/org.Hs.eg.db annotations. For low cell-cycle-activity datasets like the glioblastoma data, run ccRemover with the reduced cutoff CΔ = 2 (ccRemover(dat, cutoff = 2, max_it = 4, nboot = 200, ntop = 10)). Apply the SAME clustering algorithm to all matrices — for the five glioblastomas, hierarchical clustering split into five clusters, with each cluster assigned the majority tumor-of-origin class — and compute accuracy as the fraction of cells whose cluster majority label matches their true label. The rationale is that identical clustering on parallel matrices isolates the correction method's effect; ccRemover wins when it removes the cell-cycle effect while preserving the biological signal of interest.

## Related tools

- **ccRemover** (Produces the corrected matrix expected to give the largest clustering-accuracy gain; run with cutoff = 2 for low cell-cycle-activity glioblastoma data) — cran/ccRemover
- **scLVM** (Baseline latent-factor correction (K = 1 and scree-plot K) whose clustering accuracy is compared against ccRemover and original data)
- **org.Hs.eg.db** (Human cell-cycle gene annotations used to build the if_cc indicator vector for ccRemover)
- **Biomart** (Source of cell-cycle-annotated gene lists used to mark cell-cycle genes)

## Examples

```
xhat <- ccRemover(dat, cutoff = 2, max_it = 4, nboot = 200, ntop = 10)
```

## Evaluation signals

- Each reported accuracy (original, scLVM, ccRemover) falls within ±2 percentage points of the paper's 87.44%, 90.00%, and 92.32%.
- The ccRemover-corrected accuracy exceeds the original-data accuracy (92.32% > 87.44%).
- Identical clustering parameters (e.g. hierarchical clustering split into five clusters) are applied to all three matrices.
- Each cluster's assigned majority tumor-of-origin class yields a per-cell accuracy computed against the true MGH26-MGH31 labels.
- Comparison table reproduces the ordering original < scLVM < ccRemover for glioblastoma clustering accuracy.

## Limitations

- scLVM does not inspect whether a leading factor represents the cell cycle and can mistakenly remove the cell-type effect (K = 1 on simulated data) or both cell-cycle and cell-type effects (K = 3).
- ccRemover may miss the cell-cycle effect if it influences too few control genes (below roughly 8% affected).
- Relies on annotation databases (Biomart, org.Hs.eg.db) that are always incomplete and inaccurate; no software versions are reported for ccRemover or scLVM.

## Evidence

- [results] Hierarchical clustering was applied to the (original, scLVM corrected, and ccRemover corrected) data, splitting the cells into five clusters, with each cluster being assigned the class of the: "Hierarchical clustering was applied to the (original, scLVM corrected, and ccRemover corrected) data, splitting the cells into five clusters, with each cluster being assigned the class of the"
- [results] the same clustering algorithms and statistical tests are applied to each of them allowing us to compare the performance of the methods: "the same clustering algorithms and statistical tests are applied to each of them allowing us to compare the performance of the methods"
- [methods] except for the glioblastoma data, where it is known that the cell-cycle activity is at a very low level47. We use a smaller cutoff value CΔ = 2: "except for the glioblastoma data, where it is known that the cell-cycle activity is at a very low level47. We use a smaller cutoff value CΔ = 2"
- [abstract] The effectiveness of ccRemover is demonstrated using simulation data and three real scRNA-Seq datasets, where it boosts the performance of existing clustering algorithms in distinguishing between: "The effectiveness of ccRemover is demonstrated using simulation data and three real scRNA-Seq datasets, where it boosts the performance of existing clustering algorithms in distinguishing between"
- [results] For the choice of K (the number of leading factors to be removed) in scLVM, we try both the default value K = 1 and the value given by the scree plot: "For the choice of K (the number of leading factors to be removed) in scLVM, we try both the default value K = 1 and the value given by the scree plot"
