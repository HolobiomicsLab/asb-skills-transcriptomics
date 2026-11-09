---
name: latent-factor-removal-from-expression-matrices
description: Use when a normalized log-transformed scRNA-Seq expression matrix shows
  within-cell-type heterogeneity suspected to arise from cell-cycle effects, and a
  known cell-cycle gene annotation set is available to guide removal before clustering.
when_to_use_negative:
- When the goal is to remove non-cell-cycle latent confounders (e.g. batch or mitochondrial
  effects) — ccRemover is designed for removing the cell-cycle effect exclusively
- When cell-cycle activity is very low and few control genes are influenced (below
  ~8% affected), the effect may not be captured by the principal components and may
  be missed
- When no reliable cell-cycle gene annotation set is available for the species
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: ccRemover
  role: Primary R function implementing the iterative PCA-bootstrap detection-removal
    loop (cutoff=3, max_it=4, nboot=200, ntop=10)
  repo: cran/ccRemover
- name: gene_indexer
  role: Maps gene names to cell-cycle gene indices for the chosen species and name
    type
  repo: cran/ccRemover
- name: bootstrap_diff
  role: Generates bootstrap replicates of the loading-strength difference between
    cell-cycle and control genes
- name: get_diff
  role: Computes the difference in loading strength between cell-cycle and control
    genes used in the bootstrap t-statistic
- name: scLVM
  role: Alternative latent-factor removal method compared against ccRemover (applied
    via Python script); lacks a formal test that a leading factor is cell-cycle related
- name: org.Mm.eg.db
  role: Annotation source for mouse cell-cycle gene lists
- name: org.Hs.eg.db
  role: Annotation source for human cell-cycle gene lists
- name: Biomart
  role: Used to download lists of genes annotated to the cell cycle
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/srep33892
    title: Identifying and removing the cell-cycle effect from single-cell RNA-Sequencing
      data
  source_content_hash: sha256:973a592732992d1431402a755efa125edd99b9a70660a37f1a97106f24c95907
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cran/ccRemover
  tools:
  - Biomart
  - bootstrap_diff
  - ccRemover
  - gene_indexer
  - get_diff
  - org.Hs.eg.db
  - org.Mm.eg.db
  - scLVM
  - stats
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7619047619047619
n_claims: 21
---

# latent-factor-removal-from-expression-matrices

## Summary

Identifies and removes unwanted latent factors (specifically the cell-cycle effect) from single-cell RNA-Seq expression matrices using ccRemover's PCA-based iterative loop, preserving other biological signals of interest.

## When to use

Use when a normalized log-transformed scRNA-Seq expression matrix shows within-cell-type heterogeneity suspected to arise from cell-cycle effects, and a known cell-cycle gene annotation set is available to guide removal before clustering.

## When NOT to use

- When the goal is to remove non-cell-cycle latent confounders (e.g. batch or mitochondrial effects) — ccRemover is designed for removing the cell-cycle effect exclusively
- When cell-cycle activity is very low and few control genes are influenced (below ~8% affected), the effect may not be captured by the principal components and may be missed
- When no reliable cell-cycle gene annotation set is available for the species

## Inputs

- Normalized, log-transformed, gene-by-gene centered gene-by-cell expression matrix
- if_cc indicator vector marking annotated cell-cycle genes (from Biomart / org.Mm.eg.db / org.Hs.eg.db / Cyclebase)
- Gene name list with species and name type for gene_indexer ('mouse'/'human', 'symbols')

## Outputs

- Cell-cycle-free corrected data matrix written to xhat.csv (xhat object in R)
- Per-iteration diagnostics of which principal components were declared significant cell-cycle effects

## How to apply

Split genes into a cell-cycle set XA (using the if_cc indicator vector, retrieved via annotation databases such as org.Mm.eg.db/org.Hs.eg.db or Biomart) and a control set XB, then PCA-decompose XB to obtain the top ntop=10 loadings. For each loading, compute the difference in loading strength between cell-cycle and control genes, and use a two-class bootstrap (nboot=200 resamples) with a bootstrap-based t-statistic to declare components significant at cutoff C=3 (roughly p=0.01). Project the full matrix X onto the significant directions and subtract those projections; repeat the detection-removal loop until a full pass finds no significant component or max_it=4 iterations is reached. Finally add stored gene means back to yield the corrected matrix xhat, which serves as a pre-processing step before downstream clustering.

## Related tools

- **ccRemover** (Primary R function implementing the iterative PCA-bootstrap detection-removal loop (cutoff=3, max_it=4, nboot=200, ntop=10)) — cran/ccRemover
- **gene_indexer** (Maps gene names to cell-cycle gene indices for the chosen species and name type) — cran/ccRemover
- **bootstrap_diff** (Generates bootstrap replicates of the loading-strength difference between cell-cycle and control genes)
- **get_diff** (Computes the difference in loading strength between cell-cycle and control genes used in the bootstrap t-statistic)
- **scLVM** (Alternative latent-factor removal method compared against ccRemover (applied via Python script); lacks a formal test that a leading factor is cell-cycle related)
- **org.Mm.eg.db** (Annotation source for mouse cell-cycle gene lists)
- **org.Hs.eg.db** (Annotation source for human cell-cycle gene lists)
- **Biomart** (Used to download lists of genes annotated to the cell cycle)

## Examples

```
gene_cycle_gene_indices <- gene_indexer(gene_names, species = "mouse", name_type = "symbols"); xhat <- ccRemover(dat, cutoff = 3, max_it = 4, nboot = 200, ntop = 10, bar = FALSE)
```

## Evaluation signals

- After correction, cells should separate by cell type with cell-cycle stages mixed — as in the simulation where ccRemover correctly identified the 2nd and 3rd principal components as cell-cycle effects
- The loop must terminate: either no significant component (bootstrap t > C=3) is found in a pass, or max_it=4 iterations is reached
- Downstream clustering odds ratios should improve (e.g. TH2 signature odds ratio > 1 after correction, versus < 1 on original data)
- Other biological signals must be preserved — corrected data should not collapse to noise as occurred with scLVM at K=3
- On glioblastoma data (very low cell-cycle activity), a smaller cutoff CΔ=2 is expected to be needed

## Limitations

- ccRemover relies on known sets of cell-cycle genes from annotation databases, which are always incomplete and inaccurate
- If the cell-cycle effect only influences a few of the control genes (below ~8% affected), it may not be captured by the principal components and hence missed for detection
- ccRemover does not remove other confounding effects (e.g. batch) — it is designed for removing the cell-cycle effect exclusively
- For datasets with very low cell-cycle activity (e.g. glioblastoma data), the default cutoff C=3 may be too stringent; a smaller CΔ=2 was used

## Evidence

- [methods] iterative control loop definition: "The ccRemover algorithm follows these steps: 1. Perform a PCA on the data matrix of control genes XB."
- [methods] default parameters: "xhat <- ccRemover(dat, cutoff = 3, max_it = 4, nboot = 200, ntop = 10, bar=FALSE)"
- [methods] log-transform recommendation: "We recommend transforming the data to a log scale and centering it on a gene-by-gene basis."
- [abstract] preserves other signals: "ccRemover preserves other biological signals of interest in the data and thus can serve as an important pre-processing step for many scRNA-Seq data analyses."
- [discussion] control-gene miss risk: "if the cell-cycle effect only influences a few of the control genes, it may not be captured by the principal components and hence missed for detection"
- [discussion] exclusive scope: "ccRemover does not remove these effects as it is designed for removing the cell-cycle effect exclusively."
- [results] simulation result: "ccRemover is able to correctly identify the second and third principal components as cell-cycle effects and removes them"
- [discussion] annotation incompleteness: "In reality, the annotation databases are always incomplete and inaccurate."
