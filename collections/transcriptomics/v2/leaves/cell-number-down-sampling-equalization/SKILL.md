---
name: cell-number-down-sampling-equalization
description: Use when comparing cluster-purity (ROGUE) values across populations whose
  cell-type cell counts differ substantially — e.g., aggregated batches vs. individual
  patient samples in Kang et al. GSE96583 PBMC data — to rule out cell-count confounding.
when_to_use_negative:
- When cell-type cell counts are already balanced across the populations being compared,
  down-sampling adds no information and discards data unnecessarily
- When the analysis goal is a single population's purity rather than a comparison
  across populations or batches — there is no confound to equalize
- When rare cell types would be reduced to too few cells for stable entropy estimation
  by equalizing to the smallest group
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: ROGUE
  role: R package whose CalculateRogue function is re-run on the down-sampled, equalized
    matrices to recompute cluster purity under balanced cell numbers
  repo: https://github.com/PaulingLiu/ROGUE
- name: tidyverse
  role: R ecosystem used for subsetting and down-sampling cell metadata and expression
    matrices to equal cell numbers per type
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-020-16904-3
    title: An entropy-based metric for assessing the purity of single cell populations
  source_content_hash: sha256:a79dea1a4062d1e49c16232bc2ce744115811249e4805768f37c809ab7a56ded
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PaulingLiu/ROGUE
  tools:
  - R
  - ROGUE
  - devtools
  - tidyverse
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# cell-number-down-sampling-equalization

## Summary

A confound-removal procedure from the ROGUE cluster-purity workflow: before comparing ROGUE values across populations (e.g., per-patient batches vs. aggregated IFN-stimulated PBMCs), matrices are down-sampled so each cell type contributes an equal cell number, eliminating cell-count differences as a driver of measured purity.

## When to use

Use when comparing cluster-purity (ROGUE) values across populations whose cell-type cell counts differ substantially — e.g., aggregated batches vs. individual patient samples in Kang et al. GSE96583 PBMC data — to rule out cell-count confounding.

## When NOT to use

- When cell-type cell counts are already balanced across the populations being compared, down-sampling adds no information and discards data unnecessarily
- When the analysis goal is a single population's purity rather than a comparison across populations or batches — there is no confound to equalize
- When rare cell types would be reduced to too few cells for stable entropy estimation by equalizing to the smallest group

## Inputs

- Gene-by-cell UMI count matrix from Kang et al. PBMC data (GSE96583), pre-filtered with matr.filter(expr, min.cells, min.genes)
- Cell metadata assigning cells to cell type, individual (eight donors), and condition (control vs. IFN-beta stimulated)
- Per-cell-type cell counts used to determine the equalized down-sampling size

## Outputs

- Down-sampled, cell-number-equalized expression matrices per cell type and per aggregation level
- ROGUE values computed on equalized matrices for per-patient batches versus aggregated populations
- Results table (e.g., rogue_pbmc_results.csv) demonstrating that purity reduction upon aggregation persists after cell-count equalization

## How to apply

After computing ROGUE with matr.filter, entropy_fit, DetermineK, and CalculateRogue on the original count matrices, construct down-sampled expression matrices in which each cell type is represented by the same number of cells (equalized to the smallest group). Recompute per-cluster ROGUE on these equalized matrices for both the per-patient batches and the aggregated population, setting the platform argument to match the UMI-based 10x protocol. The rationale is that ROGUE estimates entropy over cells, so unequal cell numbers per type could alone produce apparent purity differences; if the purity-reduction pattern upon aggregation persists on equalized data, it is attributable to batch/individual effects rather than cell-count imbalance. Tabulate the down-sampled ROGUE values alongside the original ones to demonstrate the confound was removed.

## Related tools

- **ROGUE** (R package whose CalculateRogue function is re-run on the down-sampled, equalized matrices to recompute cluster purity under balanced cell numbers) — https://github.com/PaulingLiu/ROGUE
- **tidyverse** (R ecosystem used for subsetting and down-sampling cell metadata and expression matrices to equal cell numbers per type)

## Examples

```
set.seed(1); ds <- cell_meta %>% group_by(cell_type, patient) %>% slice_sample(n = min_n); rogue_ds <- CalculateRogue(expr[, ds$cell], platform = 'UMI')
```

## Evaluation signals

- Down-sampled matrices contain identical cell counts per cell type across all compared populations (per-patient vs. aggregated)
- Aggregated-population ROGUE remains systematically lower than per-patient batch ROGUE for each cell type even after equalization, reproducing the manuscript's purity-reduction pattern
- Results table contains ROGUE values for every cell type under both control and IFN-beta stimulated conditions
- Down-sampling does not reverse the sign or rank-order of purity differences observed on the full data

## Limitations

- Down-sampling discards cells, reducing the data available for entropy estimation and potentially increasing ROGUE variance for rare cell types
- Equalization controls cell-count confounding but not other technical confounders; the article notes current ROGUE does not yet assess purity of integrated populations from different protocols and platforms

## Evidence

- [other] Repeat the ROGUE computation on down-sampled matrices equalized to the same cell number per cell type to remove cell-count confounding.: [evidence span withheld — anchor status: not_found]
- [methods] Compute per-cluster purity with CalculateRogue: "CalculateRogue(.x, platform, cutoff, k, features)"
- [abstract] ROGUE is robust across dataset sizes: "We demonstrate that our ROGUE metric is broadly applicable, and enables accurate, sensitive and robust assessment of cluster purity"
- [readme] README installation and reproduction instructions: "devtools::install_github("PaulingLiu/ROGUE")"
