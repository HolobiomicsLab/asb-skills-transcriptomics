---
name: single-cell-rna-seq-entropy-modeling
description: Use when you need to determine whether an scRNA-seq cluster is a pure
  cell population, e.g. assessing purity loss from batch effects such as aggregating
  control and IFN-beta stimulated PBMCs or cells from multiple unrelated individuals
  (Kang et al., GSE96583).
when_to_use_negative:
- Assessing purity of cell populations integrated across different protocols and platforms
  — the paper states ROGUE does not yet support this and proposes it as future work
- Low-quality or continuous datasets where the recommended 0.9 purity threshold is
  unsuitable — the paper notes the threshold should instead be derived from global
  ROGUE values
- Clustering tasks aimed at detecting rare cell types, which are the purpose of GiniClust/GiniClust2/RaceID
  rather than purity quantification
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: ROGUE
  role: Core R package implementing entropy_fit, DetermineK, matr.filter, and CalculateRogue
    to compute cluster purity scores
  repo: https://github.com/PaulingLiu/ROGUE
- name: tidyverse
  role: Dependency R package that must be installed before installing ROGUE
- name: devtools
  role: Used to install ROGUE from GitHub via devtools::install_github("PaulingLiu/ROGUE")
- name: R
  role: Runtime environment for the ROGUE workflow
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-020-16904-3
    title: An entropy-based metric for assessing the purity of single cell populations
  source_content_hash: sha256:b0d65fad17c158372acdd531024832401f94d95713e68fd7ff6e4fc8035ad12d
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
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# single-cell-rna-seq-entropy-modeling

## Summary

An entropy-based modeling workflow using the ROGUE statistic (Ratio of Global Unshifted Entropy) to quantify the purity/homogeneity of single-cell RNA-seq clusters, implemented in the open-source R package ROGUE.

## When to use

Use when you need to determine whether an scRNA-seq cluster is a pure cell population, e.g. assessing purity loss from batch effects such as aggregating control and IFN-beta stimulated PBMCs or cells from multiple unrelated individuals (Kang et al., GSE96583).

## When NOT to use

- Assessing purity of cell populations integrated across different protocols and platforms — the paper states ROGUE does not yet support this and proposes it as future work
- Low-quality or continuous datasets where the recommended 0.9 purity threshold is unsuitable — the paper notes the threshold should instead be derived from global ROGUE values
- Clustering tasks aimed at detecting rare cell types, which are the purpose of GiniClust/GiniClust2/RaceID rather than purity quantification

## Inputs

- Gene-by-cell scRNA-seq count matrix (raw UMI counts, e.g. 10x data; GSE96583 Kang et al. PBMC count matrix)
- Cell metadata with cell-type labels, patient/batch, and condition (control vs IFN-beta stimulated)
- min.cells and min.genes thresholds for expression-matrix filtering

## Outputs

- Per-cluster ROGUE purity scores (per cell type, per batch, and per aggregated population)
- Results table (e.g. rogue_pbmc_results.csv) with ROGUE values per cell type under both conditions
- Fitted entropy-mean relationships and selected k per cell type (from entropy_fit / DetermineK)

## How to apply

Download a gene-by-cell UMI count matrix with per-cell metadata (cell type, patient, condition). Filter the expression matrix with matr.filter(expr, min.cells, min.genes), then for each cell type fit the mean-expression/entropy relationship with entropy_fit(.x, span, mt.method) and select variable genes and optimal k via DetermineK(expr, span, r, mt.method, if.adj). Compute per-cluster purity with CalculateRogue(.x, platform, cutoff, k, features), setting the platform argument to match the UMI-based 10x protocol. To quantify batch-effect impact (as in the Kang et al. PBMC study), compare ROGUE per cell type for patient-specific batches against the aggregated population, optionally equalizing cell counts by down-sampling to remove cell-count confounding; aggregated populations should show systematically lower ROGUE, indicating purity reduction.

## Related tools

- **ROGUE** (Core R package implementing entropy_fit, DetermineK, matr.filter, and CalculateRogue to compute cluster purity scores) — https://github.com/PaulingLiu/ROGUE
- **tidyverse** (Dependency R package that must be installed before installing ROGUE)
- **devtools** (Used to install ROGUE from GitHub via devtools::install_github("PaulingLiu/ROGUE"))
- **R** (Runtime environment for the ROGUE workflow)

## Examples

```
devtools::install_github("PaulingLiu/ROGUE"); rogue <- ROGUE::CalculateRogue(expr_filtered, platform = "UMI")
```

## Evaluation signals

- The results table contains ROGUE values for every cell type under both control and IFN-beta stimulated conditions
- Aggregated-population ROGUE values are systematically lower than per-patient batch ROGUE values for each cell type, reproducing the manuscript's purity-reduction pattern
- Down-sampled matrices equalized to the same cell number per cell type still show the purity-reduction pattern, confirming it is not a cell-count artifact
- DetermineK returns an optimal k and a variable-gene set for every cell-type/batch combination before ROGUE is computed

## Limitations

- ROGUE does not yet assess purity of integrated cell populations from different protocols and platforms; this is proposed as future work
- The recommended 0.9 purity threshold may not suit low-quality or continuous data, where it must be determined from global ROGUE values
- Cell-count differences can confound ROGUE comparisons, requiring down-sampling to equalize cell numbers per cell type

## Evidence

- [abstract] ROGUE quantifies purity: "Here we present an entropy-based statistic, ROGUE, to accurately quantify the purity of identified cell clusters."
- [intro] ROGUE statistic definition: "we propose the Ratio of Global Unshifted Entropy (ROGUE) statistic to quantify the purity or homogeneity of a given single-cell population while accounting for other technical factors"
- [readme] Installation of ROGUE: "To install ROGUE, run:
if (!requireNamespace("devtools", quietly = TRUE)) install.packages("devtools")
devtools::install_github("PaulingLiu/ROGUE")"
- [methods] CalculateRogue function signature: "CalculateRogue(.x, platform, cutoff, k, features)"
- [discussion] Purity threshold caveat: "But for low-quality data or continuous data, the threshold could be determined by considering the global ROGUE values"
