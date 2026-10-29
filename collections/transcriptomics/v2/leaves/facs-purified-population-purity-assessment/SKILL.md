---
name: facs-purified-population-purity-assessment
description: Use when you have expression matrices from FACS-sorted or otherwise purified
  cell populations (or a control such as ERCC spike-ins) and need a quantitative,
  thresholded measure of cluster/population purity before downstream analysis.
when_to_use_negative:
- The input is low-quality or continuous data, where the article states the recommended
  0.9 purity threshold is unsuitable and must instead be set from the global ROGUE
  values
- The goal is to assess purity of integrated cell populations from different protocols/platforms
  — the article notes this is not yet supported and is proposed as future work
- The input has no cell-level count matrix or cluster labels, e.g. it is already a
  summarized feature table or metadata-only summary
edam_operation: http://edamontology.org/operation_0576
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2229
tools:
- name: ROGUE
  role: Core R package implementing the Ratio of Global Unshifted Entropy statistic;
    its rogue() and CalculateRogue() functions are called on each FACS-purified expression
    matrix to produce purity scores
  repo: https://github.com/PaulingLiu/ROGUE
- name: tidyverse
  role: Dependency R package installed before ROGUE, used for data manipulation when
    loading and tabulating the expression matrices and results
- name: devtools
  role: R package used to install ROGUE from GitHub (devtools::install_github("PaulingLiu/ROGUE"))
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-020-16904-3
    title: An entropy-based metric for assessing the purity of single cell populations
  source_content_hash: sha256:c08edd4dd53f5998dee8e887b63907078c4dbb732249165c6eef541c7e699078
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PaulingLiu/ROGUE
  tools:
  - R
  - ROGUE
  - devtools
  - tidyverse
  edam_operation: http://edamontology.org/operation_0576
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# facs-purified-population-purity-assessment

## Summary

Quantify the purity of FACS-isolated single-cell populations (e.g. PBMC FACS-sorted immune subsets, ERCC spike-in controls, colorectal-cancer T-cell subtypes) using the entropy-based ROGUE statistic implemented in the ROGUE R package.

## When to use

Use when you have expression matrices from FACS-sorted or otherwise purified cell populations (or a control such as ERCC spike-ins) and need a quantitative, thresholded measure of cluster/population purity before downstream analysis.

## When NOT to use

- The input is low-quality or continuous data, where the article states the recommended 0.9 purity threshold is unsuitable and must instead be set from the global ROGUE values
- The goal is to assess purity of integrated cell populations from different protocols/platforms — the article notes this is not yet supported and is proposed as future work
- The input has no cell-level count matrix or cluster labels, e.g. it is already a summarized feature table or metadata-only summary

## Inputs

- Gene-by-cell UMI count matrix for FACS-purified PBMC populations (CD4 naive T cells, CD8 naive T cells, CD14 monocytes, CD34+ cells)
- ERCC spike-in droplet expression matrix from GSE118127
- Colorectal-cancer T-cell expression matrix with cluster labels for six well-defined T-cell subtypes plus the complete unsorted dataset
- Cluster/cell-type label assignments per cell (for CalculateRogue())

## Outputs

- Per-population ROGUE purity scores (ERCC ~1; CD4/CD8 naive T cells 0.94–1; CD14/CD34+ ~0.8; six CRC T-cell subtypes >0.9; complete CRC T-cell data ~0.78)
- A comparison table with dataset, population/subtype, and ROGUE columns (e.g. rogue_values.csv) benchmarked against manuscript-reported values

## How to apply

Install the tidyverse dependency and the ROGUE R package from GitHub via devtools::install_github("PaulingLiu/ROGUE"). Load each gene-by-cell UMI count matrix and compute ROGUE with platform-appropriate settings: the ERCC-only droplet control from GSE118127 should yield a ROGUE of ~1 when run with rogue(platform = 'droplet'), while FACS-purified PBMC populations are scored with UMI-appropriate settings (CD4/CD8 naive T cells expected at 0.94–1; CD14 monocytes and CD34+ cells near 0.8). For the colorectal-cancer T-cell matrix, call CalculateRogue() with the provided cluster labels to score each of the six well-defined subtypes (expected >0.9) and the complete unsplit dataset (expected ~0.78). Assemble all values into a dataset/population/ROGUE table and compare against the manuscript-reported values to confirm the populations are pure; a 0.9 threshold is the recommended purity benchmark for high-quality data.

## Related tools

- **ROGUE** (Core R package implementing the Ratio of Global Unshifted Entropy statistic; its rogue() and CalculateRogue() functions are called on each FACS-purified expression matrix to produce purity scores) — https://github.com/PaulingLiu/ROGUE
- **tidyverse** (Dependency R package installed before ROGUE, used for data manipulation when loading and tabulating the expression matrices and results)
- **devtools** (R package used to install ROGUE from GitHub (devtools::install_github("PaulingLiu/ROGUE")))

## Examples

```
install.packages("tidyverse"); devtools::install_github("PaulingLiu/ROGUE"); CalculateRogue(expr_crc_tcells, platform = 'UMI')
```

## Evaluation signals

- ERCC-only droplet control from GSE118127 returns ROGUE ≈ 1 (the theoretical maximum) with platform = 'droplet'
- FACS-purified CD4 and CD8 naive T cell populations fall in the 0.94–1 range
- CD14 monocytes and CD34+ cell populations score approximately 0.8, as reported for these less-homogeneous FACS populations
- All six CRC T-cell subtypes achieve ROGUE > 0.9 while the complete unsplit T-cell dataset scores ~0.78, demonstrating subtype-level purity gain
- Computed values written to rogue_values.csv match the manuscript-reported table within expected tolerance

## Limitations

- The recommended 0.9 purity threshold may not suit low-quality or continuous data, where it must be determined from global ROGUE values
- ROGUE does not yet assess purity of integrated cell populations from different protocols and platforms; this is proposed as future work
- Purified FACS populations with inherent biological heterogeneity (e.g. CD14 monocytes, CD34+ cells at ~0.8) may not exceed the 0.9 benchmark despite being correctly measured

## Evidence

- [abstract] entropy statistic to quantify purity: "Here we present an entropy-based statistic, ROGUE, to accurately quantify the purity of identified cell clusters."
- [methods] installation instructions: "Before installing ROGUE, the “tidyverse” package should be installed first:
install.packages("tidyverse")"
- [methods] devtools install command: "devtools::install_github("PaulingLiu/ROGUE")"
- [methods] reproduction scripts: "The scripts for producing all the quantitative results in our manuscript can be found in [scripts](./scripts)"
- [discussion] threshold caveat: "But for low-quality data or continuous data, the threshold could be determined by considering the global ROGUE values"
- [readme] README reproduction pointer: "The scripts for producing all the quantitative results in our manuscript can be found in [scripts](./scripts)."
- [intro] ROGUE definition: "we propose the Ratio of Global Unshifted Entropy (ROGUE) statistic to quantify the purity or homogeneity of a given single-cell population while accounting for other technical factors"
