---
name: platform-aware-variable-gene-selection
description: Use when quantifying cluster purity or selecting highly variable genes
  for scRNA-seq clusters, especially UMI-based (10x) data, and when purity metrics
  must be comparable across datasets, batches, or platforms.
when_to_use_negative:
- The goal is integrated-data purity across cells from different protocols and platforms
  — the article notes current ROGUE does not yet assess purity of integrated cell
  populations from different protocols/platforms (proposed future work)
- The data are low-quality or continuous, where the recommended 0.9 purity threshold
  is not suitable and must instead be determined from global ROGUE values
- The aim is rare cell-type detection/clustering itself (e.g., GiniClust/RaceID) rather
  than purity assessment of already-identified clusters
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2269
tools:
- name: ROGUE
  role: R package providing matr.filter, entropy_fit, DetermineK, Entropy, and CalculateRogue
    to select variable genes and quantify cluster purity via the ROGUE statistic
  repo: https://github.com/PaulingLiu/ROGUE
- name: tidyverse
  role: Required dependency installed before ROGUE (install.packages("tidyverse"))
- name: devtools
  role: Used to install ROGUE from GitHub via devtools::install_github("PaulingLiu/ROGUE")
- name: R
  role: Runtime environment in which the ROGUE pipeline is executed
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-020-16904-3
    title: An entropy-based metric for assessing the purity of single cell populations
  source_content_hash: sha256:8d306e6457cf95ba94933b006b590d85f039620d594f967235193d26cd1331cf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PaulingLiu/ROGUE
  tools:
  - R
  - ROGUE
  - devtools
  - tidyverse
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# Platform-aware variable gene selection

## Summary

Select variable genes for cluster-purity assessment using ROGUE's entropy-based model (entropy_fit and DetermineK), which measures the randomness of gene expression while accounting for technical factors and the scRNA-seq platform (e.g., UMI-based 10x data).

## When to use

Use when quantifying cluster purity or selecting highly variable genes for scRNA-seq clusters, especially UMI-based (10x) data, and when purity metrics must be comparable across datasets, batches, or platforms.

## When NOT to use

- The goal is integrated-data purity across cells from different protocols and platforms — the article notes current ROGUE does not yet assess purity of integrated cell populations from different protocols/platforms (proposed future work)
- The data are low-quality or continuous, where the recommended 0.9 purity threshold is not suitable and must instead be determined from global ROGUE values
- The aim is rare cell-type detection/clustering itself (e.g., GiniClust/RaceID) rather than purity assessment of already-identified clusters

## Inputs

- Gene-by-cell raw UMI count matrix (e.g., 10x scRNA-seq data such as the Kang et al. PBMC matrix, GSE96583)
- Cell metadata assigning each cell to a cell type/cluster, batch, patient, or condition (e.g., control vs. IFN-beta stimulated, eight unrelated individuals)
- Function parameters: min.cells and min.genes for matr.filter; span, mt.method for entropy_fit; r, mt.method, if.adj for DetermineK; platform, cutoff, features for CalculateRogue

## Outputs

- Per-cluster ROGUE purity values (Ratio of Global Unshifted Entropy) with the selected variable-gene set and optimal k from DetermineK
- Fitted mean-expression/entropy relationship per cell type from entropy_fit
- Comparison table (e.g., rogue_pbmc_results.csv) of ROGUE values per cell type across batches, patients, or aggregated populations

## How to apply

First filter the gene-by-cell expression matrix with matr.filter(expr, min.cells, min.genes) to retain adequately expressed genes, removing genes expressed in too few cells that could otherwise inflate entropy estimates. For each cell-type cluster, fit the mean-expression/entropy relationship with entropy_fit(.x, span, mt.method); this entropy-based model measures the randomness of gene expression and identifies variable genes with high sensitivity and precision, avoiding the biases of scran, Gini, dropout-based, or dimensionality-reduction-weight alternatives. Then call DetermineK(expr, span, r, mt.method, if.adj) to select the variable genes and determine the optimal k, adjusting for technical noise; when running on UMI-based 10x data, pass the matching platform argument downstream to CalculateRogue(.x, platform, cutoff, k, features). Finally, pass the selected features and k to CalculateRogue (or the rogue() wrapper) to obtain the ROGUE (Ratio of Global Unshifted Entropy) purity value for the cluster. When comparing populations with unequal cell counts (e.g., batch-aggregated vs. patient-specific), down-sample matrices to equal cell numbers per cell type first to remove cell-count confounding.

## Related tools

- **ROGUE** (R package providing matr.filter, entropy_fit, DetermineK, Entropy, and CalculateRogue to select variable genes and quantify cluster purity via the ROGUE statistic) — https://github.com/PaulingLiu/ROGUE
- **tidyverse** (Required dependency installed before ROGUE (install.packages("tidyverse")))
- **devtools** (Used to install ROGUE from GitHub via devtools::install_github("PaulingLiu/ROGUE"))
- **R** (Runtime environment in which the ROGUE pipeline is executed)

## Examples

```
devtools::install_github("PaulingLiu/ROGUE"); expr <- matr.filter(expr, min.cells = 3, min.genes = 200); k <- DetermineK(expr, span = 0.55, r = 4, mt.method = "permutation_p", if.adj = TRUE); rogue_val <- CalculateRogue(expr, platform = "UMI", k = k, features = k$features)
```

## Evaluation signals

- The results table contains ROGUE values for every cell type under both conditions (e.g., control and IFN-beta stimulated PBMCs)
- Aggregated-population ROGUE values are systematically lower than per-patient/per-batch ROGUE values for each cell type, reproducing the purity-reduction pattern of the manuscript figures
- Down-sampling to equal cell numbers per cell type does not change the direction of the purity comparison (confirming cell-count is not the confounder)
- DetermineK returns a plausible optimal k and variable-gene set per cell type, and the fitted entropy curves from entropy_fit follow the expected mean-expression/entropy relationship

## Limitations

- Current ROGUE does not yet assess the purity of integrated cell populations from different protocols and platforms (proposed as future work)
- The recommended 0.9 purity threshold may not suit low-quality or continuous data; the threshold must be determined from global ROGUE values in those cases
- Unequal cell counts across compared populations confound ROGUE values, so matrices must be down-sampled to the same cell number per cell type before comparison
- Entropy-based variable-gene selection alternatives have known biases: scran may select many lowly expressed genes, Gini is designed for rare cell-type identification, and dropout-based approaches hinder capturing the global expression distribution

## Evidence

- [intro] Entropy-based variable gene selection via entropy_fit/DetermineK: "we present an entropy-based model to measure the randomness of gene expression in single cells"
- [intro] Accounting for technical factors in purity quantification: "we propose the Ratio of Global Unshifted Entropy (ROGUE) statistic to quantify the purity or homogeneity of a given single-cell population while accounting for other technical factors"
- [methods] Core function calls for the workflow: "matr.filter(expr, min.cells, min.genes)"
- [methods] Installation path grounding the invocation: "devtools::install_github("PaulingLiu/ROGUE")"
- [readme] Installation prerequisite: "Before installing ROGUE, the “tidyverse” package should be installed first"
