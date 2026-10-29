---
name: chi-square-contingency-testing
description: Use when you have per-cell binary calls (e.g. cells exceeding a hub-score
  outlier cutoff of mu + 3 sigma) stratified by a categorical variable such as donor
  age, and you need to test whether the outlier proportion shifts across categories.
when_to_use_negative:
- Your per-cell scores are continuous and you have not yet binarized them into outlier/non-outlier
  calls at a defined cutoff (e.g. mu + 3 sigma) — chi-square on unthresholded values
  is inappropriate
- Expected cell counts in any contingency-table cell are very small (e.g. rare age
  groups with few cells), where the chi-square approximation breaks down
- You want to compare mean score distributions rather than proportions of outlier
  cells — a rank-based or parametric test on continuous scores is the better choice
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_3308
tools:
- name: SenePy
  role: Provides hub signatures and score_hub scoring used to produce the per-cell
    scores that are binarized into outlier calls before chi-square testing
  repo: https://github.com/jaleesr/senepy
- name: scanpy
  role: Used to load and handle the single-cell AnnData object whose obs table is
    stratified by age for contingency-table construction
- name: AnnData
  role: Container format holding cell-by-gene expression and per-cell SenePy scores
    in obs for age-stratified outlier analysis
- name: Python
  role: Implementation language for scoring, outlier thresholding, contingency-table
    chi-square test, and CSV export
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-025-57047-7
    title: Unveiling the cell-type-specific landscape of cellular senescence through
      single-cell transcriptomics using SenePy
  source_content_hash: sha256:107b7dd8884e86276d9065a6ada356e840074e4911631b72b6bb91c407c04935
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jaleesr/senepy
  tools:
  - AnnData
  - Python
  - SenePy
  - scanpy
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# chi-square-contingency-testing

## Summary

Apply a chi-square test to an outlier/non-outlier contingency table to evaluate age dependence of senescent cell proportions, as done in SenePy to show tongue keratinocyte type-A/type-B hub outlier cells accumulate with organismal age (p = 7.9e-86 and 7.9e-90).

## When to use

Use when you have per-cell binary calls (e.g. cells exceeding a hub-score outlier cutoff of mu + 3 sigma) stratified by a categorical variable such as donor age, and you need to test whether the outlier proportion shifts across categories.

## When NOT to use

- Your per-cell scores are continuous and you have not yet binarized them into outlier/non-outlier calls at a defined cutoff (e.g. mu + 3 sigma) — chi-square on unthresholded values is inappropriate
- Expected cell counts in any contingency-table cell are very small (e.g. rare age groups with few cells), where the chi-square approximation breaks down
- You want to compare mean score distributions rather than proportions of outlier cells — a rank-based or parametric test on continuous scores is the better choice

## Inputs

- Single-cell gene expression AnnData object (e.g. Tabula Muris Senis tongue keratinocytes) with SenePy hub scores stored in adata.obs
- Per-hub outlier calls defined as scores > mu + 3 sigma
- Donor age group labels for each cell

## Outputs

- Per-age-group proportion of outlier (senescent) cells for each hub
- Chi-square p-value per hub from the outlier/non-outlier contingency table
- CSV file (e.g. tongue_keratinocyte_senescence_outliers.csv) containing per-age outlier proportions and chi-square p-values for both hubs

## How to apply

Score each cell in the single-cell AnnData object with SenePy hub signatures (sp.score_hub with default settings: data binarized, gene importance weighted by hub-network connection count), then define outlier cells per hub as those with scores exceeding the population mean plus 3 standard deviations (mu + 3 sigma). Build a contingency table of outlier vs non-outlier counts stratified by donor age group, and run a chi-square test on each hub's table to assess age dependence. Record per-age outlier proportions and chi-square p-values; in the SenePy tongue keratinocyte analysis this reproduced p = 7.9e-86 (type-A hub 0) and p = 7.9e-90 (type-B hub 1), supporting an age-dependent increase in senescent keratinocytes. Write per-age proportions and p-values for both hubs to a CSV (e.g. tongue_keratinocyte_senescence_outliers.csv) with a p-value column for each hub.

## Related tools

- **SenePy** (Provides hub signatures and score_hub scoring used to produce the per-cell scores that are binarized into outlier calls before chi-square testing) — https://github.com/jaleesr/senepy
- **scanpy** (Used to load and handle the single-cell AnnData object whose obs table is stratified by age for contingency-table construction)
- **AnnData** (Container format holding cell-by-gene expression and per-cell SenePy scores in obs for age-stratified outlier analysis)
- **Python** (Implementation language for scoring, outlier thresholding, contingency-table chi-square test, and CSV export)

## Examples

```
hubs = sp.load_hubs(species='Mouse'); sig = hubs.get_genes(('Tongue', 'keratinocyte', 0)); adata.obs['sp_score'] = sp.score_hub(adata, sig); outliers = adata.obs['sp_score'] > adata.obs['sp_score'].mean() + 3 * adata.obs['sp_score'].std()
```

## Evaluation signals

- Computed chi-square p-values reproduce the reported values 7.9e-86 (type-A hub 0) and 7.9e-90 (type-B hub 1) for tongue keratinocytes
- Output CSV contains a p-value column for both hubs and per-age outlier proportions
- Outlier calls are exactly the cells with hub scores > mu + 3 sigma of the score population for that hub
- Proportions of outlier cells increase monotonically (or near-monotonically) with organismal age, consistent with more senescent keratinocytes in old tongues
- Contingency table outlier/non-outlier counts sum to the total number of cells scored per hub

## Limitations

- Outlier definition is sensitive to the mu + 3 sigma cutoff choice on hub scores; results depend on score_hub defaults (binarized data, gene importance weighting by hub-network connection count)
- Chi-square tests proportions but not score magnitudes, and requires adequate counts per age stratum
- SenePy signatures do not account for downregulated genes in senescent cells (no negative markers) and discount CS genes overlapping the age-dependent transcriptional shift

## Evidence

- [other] Scoring tongue keratinocytes with the type-A and type-B hub signatures shows that the number of outlier cells (> μ + 3σ) increases significantly with organismal age: [evidence span withheld — anchor status: not_found]
- [other] hub 0,1 Chi-Square p-value = 7.9 × 10−86, 7.9 × 10−90: "hub 0,1 Chi-Square p-value = 7.9 × 10−86, 7.9 × 10−90"
- [readme] score_hub default settings: "By default, it weighs the importance of each gene based on the number of connections that gene has in the hub network. By default, it binarizes the data"
- [readme] get_genes usage: "extracted_sig = hubs.get_genes(('Lung', 'fibroblast', 1)) #tissue, cell, hub_num"
- [readme] score_hub stores in obs: "adata.obs['sp_score'] = sp.score_hub(adata, hubs.hubs[('Lung', 'fibroblast', 1)])"
