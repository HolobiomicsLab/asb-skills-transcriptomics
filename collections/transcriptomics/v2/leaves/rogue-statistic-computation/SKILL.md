---
name: rogue-statistic-computation
description: Use when a scRNA-seq analysis needs to quantify whether an identified
  cell cluster is a pure population, or to compare cluster purity across batches,
  individuals, or re-clustering strategies.
when_to_use_negative:
- Assessing purity of integrated cell populations from different protocols or platforms
  — the article notes this is not yet supported and proposed as future work
- Low-quality or continuous data where the recommended 0.9 purity threshold is unsuitable
  and must be derived from global ROGUE values
- The input is not a single-cell expression matrix (e.g., bulk RNA-seq or already-summarized
  feature tables)
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: ROGUE
  role: Core R package providing matr.filter, entropy_fit, DetermineK, and CalculateRogue
    for computing the entropy-based purity statistic
  repo: https://github.com/PaulingLiu/ROGUE
- name: tidyverse
  role: Required dependency installed before ROGUE
- name: devtools
  role: Used to install ROGUE from GitHub via install_github
- name: R
  role: Runtime environment for the ROGUE package and reproduction scripts
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-020-16904-3
    title: An entropy-based metric for assessing the purity of single cell populations
  source_content_hash: sha256:7cb925bb28900c04f051635ffbc879217fc7a0f8c687cc82662fe9be9cc6c1f2
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
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# rogue-statistic-computation

## Summary

Compute the ROGUE (Ratio of Global Unshifted Entropy) entropy-based statistic to quantify the purity or homogeneity of single-cell clusters, implemented in the open-source R package ROGUE.

## When to use

Use when a scRNA-seq analysis needs to quantify whether an identified cell cluster is a pure population, or to compare cluster purity across batches, individuals, or re-clustering strategies.

## When NOT to use

- Assessing purity of integrated cell populations from different protocols or platforms — the article notes this is not yet supported and proposed as future work
- Low-quality or continuous data where the recommended 0.9 purity threshold is unsuitable and must be derived from global ROGUE values
- The input is not a single-cell expression matrix (e.g., bulk RNA-seq or already-summarized feature tables)

## Inputs

- Gene-by-cell raw UMI count matrix (e.g., 10x scRNA-seq data such as the Kang et al. PBMC dataset, GEO GSE96583)
- Cell metadata assigning cells to clusters/cell types, batches, and conditions (control vs IFN-beta stimulated)

## Outputs

- Per-cluster ROGUE purity scores (values near 1 indicate pure populations; the recommended purity threshold is 0.9)
- Results table (e.g., rogue_pbmc_results.csv) of ROGUE values per cell type per batch/condition
- Variable gene sets and optimal k per cluster from DetermineK

## How to apply

Install the tidyverse package first, then install ROGUE from GitHub with devtools::install_github("PaulingLiu/ROGUE"). Filter the gene-by-cell expression matrix with matr.filter(expr, min.cells, min.genes) to retain adequately expressed genes. For each cluster, fit the mean-expression/entropy relationship with entropy_fit(.x, span, mt.method), select variable genes and the optimal k with DetermineK(expr, span, r, mt.method, if.adj), then compute cluster purity with CalculateRogue(.x, platform, cutoff, k, features), setting the platform argument to match the sequencing protocol (e.g., UMI-based 10x data). To compare batch effects, compute ROGUE separately per batch and on the aggregated population; lower ROGUE values on aggregation indicate purity loss. Optionally equalize cell counts per cluster by down-sampling to remove cell-number confounding before comparison.

## Related tools

- **ROGUE** (Core R package providing matr.filter, entropy_fit, DetermineK, and CalculateRogue for computing the entropy-based purity statistic) — https://github.com/PaulingLiu/ROGUE
- **tidyverse** (Required dependency installed before ROGUE)
- **devtools** (Used to install ROGUE from GitHub via install_github)
- **R** (Runtime environment for the ROGUE package and reproduction scripts)

## Examples

```
devtools::install_github("PaulingLiu/ROGUE"); expr <- matr.filter(expr, min.cells = 3, min.genes = 200); rogue_val <- CalculateRogue(expr, platform = "UMI")
```

## Evaluation signals

- The ROGUE value approaches 1 for pure clusters; clusters below the 0.9 threshold are flagged as low purity
- Aggregated-population ROGUE must be systematically lower than per-batch/per-patient ROGUE values for the same cell type (purity-reduction pattern)
- Results table contains ROGUE values for every cell type under both control and stimulated conditions
- Down-sampled matrices equalized by cell number per cell type reproduce the same purity-reduction trend, confirming it is not a cell-count artifact
- Installation succeeds with devtools::install_github("PaulingLiu/ROGUE") after tidyverse is installed

## Limitations

- The recommended 0.9 purity threshold may not be suitable for low-quality or continuous data, where it must be determined from global ROGUE values
- Current ROGUE does not yet assess purity of integrated cell populations from different protocols and platforms
- Cell-count differences can confound ROGUE comparisons; down-sampling to equal cell numbers per cell type is needed

## Evidence

- [intro] ROGUE statistic to quantify purity: "we propose the Ratio of Global Unshifted Entropy (ROGUE) statistic to quantify the purity or homogeneity of a given single-cell population"
- [methods] Calculation functions: "matr.filter(expr, min.cells, min.genes)"
- [readme] Install ROGUE: "devtools::install_github("PaulingLiu/ROGUE")"
- [discussion] Threshold caveat: "But for low-quality data or continuous data, the threshold could be determined by considering the global ROGUE values"
