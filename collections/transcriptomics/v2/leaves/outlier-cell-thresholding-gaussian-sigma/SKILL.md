---
name: outlier-cell-thresholding-gaussian-sigma
description: Use when scoring single-cell transcriptomes with a senescence signature
  (e.g. SenePy hub scores) and you need to call individual senescent outlier cells
  and test whether their proportion shifts across groups such as donor age.
when_to_use_negative:
- 'When you need calibrated per-cell senescence probabilities rather than a population-relative
  tail call: mu + 3 sigma is distribution-relative, so a hub-score shift in the whole
  population changes the threshold'
- When the data are scaled (z-scored) — score_hub expects unscaled data; binarized
  default scoring makes normalization optional but scaling breaks it
- When comparing across hubs or datasets with very different cell numbers without
  care, since the 3-sigma cutoff and chi-square counts are sensitive to population
  size and score distribution shape
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: SenePy
  role: Provides load_hubs/get_genes signatures and score_hub scoring (binarized,
    connection-count-weighted) whose per-cell scores feed the mu + 3 sigma outlier
    rule
  repo: https://github.com/jaleesr/senepy
- name: scanpy
  role: Used with AnnData to load and manipulate the single-cell object and store
    hub scores in adata.obs
- name: AnnData
  role: Data container for the single-cell expression matrix and obs metadata (scores,
    age, outlier calls)
- name: Python
  role: Implementation environment for scoring, thresholding, chi-square testing,
    and CSV export
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-025-57047-7
    title: Unveiling the cell-type-specific landscape of cellular senescence through
      single-cell transcriptomics using SenePy
  source_content_hash: sha256:5edb9fe0c5a2a8a475f1909cc677915ad1fc1fc131cd1b4d6d98414b58f54d5f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jaleesr/senepy
  tools:
  - AnnData
  - Python
  - SenePy
  - scanpy
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# outlier-cell-thresholding-gaussian-sigma

## Summary

A statistical procedure for identifying senescent 'outlier' cells from single-cell SenePy hub-score distributions: cells exceeding the population mean plus 3 standard deviations (mu + 3 sigma) are flagged, and age-dependent shifts in outlier proportions are tested with a chi-square test on the outlier/non-outlier contingency table.

## When to use

Use when scoring single-cell transcriptomes with a senescence signature (e.g. SenePy hub scores) and you need to call individual senescent outlier cells and test whether their proportion shifts across groups such as donor age.

## When NOT to use

- When you need calibrated per-cell senescence probabilities rather than a population-relative tail call: mu + 3 sigma is distribution-relative, so a hub-score shift in the whole population changes the threshold
- When the data are scaled (z-scored) — score_hub expects unscaled data; binarized default scoring makes normalization optional but scaling breaks it
- When comparing across hubs or datasets with very different cell numbers without care, since the 3-sigma cutoff and chi-square counts are sensitive to population size and score distribution shape

## Inputs

- AnnData single-cell gene expression object (.h5ad), e.g. Tabula Muris Senis tongue keratinocytes, unscaled (raw or normalized, not scaled)
- SenePy hub object loaded via sp.load_hubs(species='Mouse') with derived tissue/cell-type signatures retrieved via hubs.get_genes((tissue, cell, hub_num))
- Per-cell metadata column with donor age group (adata.obs)

## Outputs

- Outlier/non-outlier binary calls per cell per hub (score > mu + 3 sigma), stored in adata.obs
- Per-age-group outlier proportions for each hub
- Chi-square p-values for age dependence per hub (e.g. 7.9e-86 hub 0, 7.9e-90 hub 1)
- CSV of outlier proportions and p-values, e.g. tongue_keratinocyte_senescence_outliers.csv with a p-value column for both hubs

## How to apply

Score every cell with sp.score_hub(adata, sig) using default settings, which binarizes the data and weights each gene's importance by its number of connections in the hub network, storing scores in adata.obs. For each hub score column, compute the population mean and standard deviation and define outlier cells as those exceeding mu + 3 sigma — the Gaussian tail threshold used to call senescent cells without requiring a normalized score cutoff. Stratify outlier calls by donor age (e.g. Tabula Muris Senis tongue keratinocytes), compute the proportion of outlier cells per age group, and build an outlier/non-outlier contingency table per hub for a chi-square test of age dependence. Rationale: the mu + 3 sigma rule captures the extreme tail of the hub-score distribution that marks senescent cells, and chi-square p-values (7.9e-86 for type-A hub 0, 7.9e-90 for type-B hub 1 in tongue keratinocytes) quantify whether senescent-cell burden increases with organismal age. Write per-age outlier proportions and chi-square p-values for both hubs to a CSV (e.g. tongue_keratinocyte_senescence_outliers.csv) with a p-value column for each hub.

## Related tools

- **SenePy** (Provides load_hubs/get_genes signatures and score_hub scoring (binarized, connection-count-weighted) whose per-cell scores feed the mu + 3 sigma outlier rule) — https://github.com/jaleesr/senepy
- **scanpy** (Used with AnnData to load and manipulate the single-cell object and store hub scores in adata.obs)
- **AnnData** (Data container for the single-cell expression matrix and obs metadata (scores, age, outlier calls))
- **Python** (Implementation environment for scoring, thresholding, chi-square testing, and CSV export)

## Examples

```
adata.obs['sp_score'] = sp.score_hub(adata, hubs.hubs[('Tongue', 'keratinocyte', 0)]); outliers = adata.obs['sp_score'] > adata.obs['sp_score'].mean() + 3*adata.obs['sp_score'].std()
```

## Evaluation signals

- Reproduce the reported chi-square p-values: 7.9e-86 for the type-A hub (hub 0) and 7.9e-90 for the type-B hub (hub 1) on tongue keratinocytes
- The output CSV contains per-age outlier proportions and a p-value column for both hubs (tongue_keratinocyte_senescence_outliers.csv)
- Outlier proportion per age group is monotonically consistent with increasing organismal age (more senescent keratinocytes in old tongues)
- Sanity check that roughly the extreme Gaussian tail (~0.1–0.3% under normality) is flagged per hub; large departures indicate score-distribution skew
- score_hub ran with defaults (binarize=True, importance weighting on) and data were not scaled

## Limitations

- The mu + 3 sigma threshold is population-relative: it assumes a dominant non-senescent distribution, so tissues/datasets with many senescent cells inflate sigma and mask outliers
- Default score_hub binarizes the data and weighs genes by hub-network connection count; mimicking scanpy.score_genes (binarize=False, importance=False) changes score scales and thus the 3-sigma outlier set
- Chi-square p-values can be driven by very large cell counts (e.g. 1e-86/1e-90 magnitudes), so effect size (per-age proportions) should be reported alongside
- No negative (downregulated) senescence markers are accounted for, and genes overlapping the organismal-aging transcriptional shift are discounted by the signature method

## Evidence

- [other] For each hub score, define outlier cells as those exceeding the population mean plus 3 standard deviations (mu + 3 sigma).: [evidence span withheld — anchor status: not_found]
- [other] Chi-square p-values reproduce the reported 7.9e-86 (type-A) and 7.9e-90 (type-B).: [evidence span withheld — anchor status: not_found]
- [readme] By default, it binarizes the data, which makes normalization and transformation optional.: "By default, it binarizes the data, which makes normalization and transformation optional."
- [readme] it weighs the importance of each gene based on the number of connections that gene has in the hub network: "it weighs the importance of each gene based on the number of connections that gene has in the hub network"
- [readme] Just make sure they are not scaled: "With default settings it does not matter if the data are normalized or raw. Just make sure they are not scaled"
