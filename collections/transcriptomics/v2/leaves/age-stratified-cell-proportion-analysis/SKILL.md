---
name: age-stratified-cell-proportion-analysis
description: Use when scoring cells (e.g., Tabula Muris Senis tongue keratinocytes)
  with senescence hub signatures and you need to test whether the proportion of senescence-outlier
  cells increases with organismal age.
when_to_use_negative:
- The goal is identifying which genes are senescence markers rather than testing age-dependent
  senescent-cell accumulation — use hubs.search_hubs_by_genes or merge_hubs instead
- The input is bulk transcriptomics or a pre-computed feature table with no per-cell
  scores and donor-age labels; mu + 3 sigma outlier calling and per-age contingency
  tables require single-cell scores
- Data are already scaled — score_hub expects raw or normalized (not scaled) expression
  since it binarizes by default
edam_operation: http://edamontology.org/operation_3656
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0625
tools:
- name: SenePy
  role: Provides load_hubs, get_genes, and score_hub to retrieve mouse hub signatures
    and score every cell's senescence state with default binarization and importance
    weighting
  repo: https://github.com/jaleesr/senepy
- name: AnnData
  role: Holds the single-cell expression matrix and donor-age observations; hub scores
    are stored in adata.obs
- name: scanpy
  role: Loads and manipulates the AnnData single-cell object prior to hub scoring
- name: Python
  role: Language in which the scoring, outlier calling (mu + 3 sigma), and chi-square
    tests are implemented
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-025-57047-7
    title: Unveiling the cell-type-specific landscape of cellular senescence through
      single-cell transcriptomics using SenePy
  source_content_hash: sha256:c33aa893b420557da399739d9adb26bb7c6ef0b1a1929e71f69044115e0c786e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jaleesr/senepy
  tools:
  - AnnData
  - Python
  - SenePy
  - scanpy
  edam_operation: http://edamontology.org/operation_3656
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# age-stratified-cell-proportion-analysis

## Summary

Quantifies age-dependent accumulation of senescent cells by scoring single-cell transcriptomes with SenePy cell-type-specific hub signatures, calling outlier cells at mu + 3 sigma, and testing whether the outlier proportion changes across donor ages with a chi-square test.

## When to use

Use when scoring cells (e.g., Tabula Muris Senis tongue keratinocytes) with senescence hub signatures and you need to test whether the proportion of senescence-outlier cells increases with organismal age.

## When NOT to use

- The goal is identifying which genes are senescence markers rather than testing age-dependent senescent-cell accumulation — use hubs.search_hubs_by_genes or merge_hubs instead
- The input is bulk transcriptomics or a pre-computed feature table with no per-cell scores and donor-age labels; mu + 3 sigma outlier calling and per-age contingency tables require single-cell scores
- Data are already scaled — score_hub expects raw or normalized (not scaled) expression since it binarizes by default

## Inputs

- Single-cell gene expression AnnData object (.h5ad) with donor age metadata, e.g., Tabula Muris Senis tongue keratinocytes
- SenePy hub signatures for the species (Mouse), retrieved as (tissue, cell, hub_num) gene lists with hub-network connection-count weights

## Outputs

- Per-age proportion of senescence outlier cells (score > mu + 3 sigma) for each hub
- Chi-square p-values for the outlier/non-outlier contingency table per hub (e.g., 7.9e-86 and 7.9e-90)
- CSV file, e.g., tongue_keratinocyte_senescence_outliers.csv, with p-value columns for both hubs
- Per-cell SenePy scores stored in adata.obs columns

## How to apply

Load the single-cell AnnData object (e.g., tongue keratinocytes from Tabula Muris Senis) with scanpy, then load mouse hub signatures with sp.load_hubs(species='Mouse') and extract the derived cell-type hubs via hubs.get_genes((tissue, cell, hub_num)), such as the type-A and type-B tongue keratinocyte hubs. Score every cell with sp.score_hub(adata, sig) under default settings — data binarized and each gene weighted by its number of connections in the hub network — storing scores in adata.obs. For each hub score, define outlier (senescent) cells as those exceeding the population mean plus 3 standard deviations (mu + 3 sigma). Stratify outlier calls by donor age, compute the per-age proportion of outlier cells, and evaluate age dependence with a chi-square test on the outlier/non-outlier contingency table for each hub. Correct application reproduces reported kinetics, e.g., chi-square p-values of 7.9e-86 (type-A hub) and 7.9e-90 (type-B hub) for tongue keratinocytes, and writes per-age outlier proportions plus both hub p-values to a CSV such as tongue_keratinocyte_senescence_outliers.csv.

## Related tools

- **SenePy** (Provides load_hubs, get_genes, and score_hub to retrieve mouse hub signatures and score every cell's senescence state with default binarization and importance weighting) — https://github.com/jaleesr/senepy
- **AnnData** (Holds the single-cell expression matrix and donor-age observations; hub scores are stored in adata.obs)
- **scanpy** (Loads and manipulates the AnnData single-cell object prior to hub scoring)
- **Python** (Language in which the scoring, outlier calling (mu + 3 sigma), and chi-square tests are implemented)

## Examples

```
import senepy as sp; hubs = sp.load_hubs(species='Mouse'); sig = hubs.get_genes(('Tongue', 'keratinocyte', 0)); adata.obs['sp_score'] = sp.score_hub(adata, sig)
```

## Evaluation signals

- Chi-square p-value for the type-A hub reproduces the reported 7.9e-86
- Chi-square p-value for the type-B hub reproduces the reported 7.9e-90
- Outlier counts (> mu + 3 sigma) increase monotonically/consistently with organismal age for both hubs
- Output CSV contains a p-value column for both hubs plus per-age outlier proportions
- Per-cell hub scores are present in adata.obs after score_hub calls

## Limitations

- The outlier definition (mu + 3 sigma) is population-relative; the threshold reflects each hub-score distribution rather than an absolute senescence level
- SenePy does not account for downregulated genes in senescent cells (no negative markers), which may miss parts of the CS program
- The methodology discounts CS genes that overlap with the transcriptional shift of organismal aging
- Cell-specific heterogeneity of cellular senescence means signatures are cell-type-specific; applying a hub from one tissue/cell type to another may not transfer

## Evidence

- [other] outlier cells as those exceeding the population mean plus 3 standard deviations (mu + 3 sigma): [evidence span withheld — anchor status: not_found]
- [other] evaluated with chi-square test on outlier contingency table per hub: [evidence span withheld — anchor status: not_found]
- [methods] score_hub weighs gene importance by hub-network connections and binarizes by default: "By default, it weighs the importance of each gene based on the number of connections that gene has in the hub network. By default, it binarizes the data"
- [readme] score_hub stores scores in anndata.obs: "adata.obs['sp_score'] = sp.score_hub(adata, hubs.hubs[('Lung', 'fibroblast', 1)])"
- [readme] score_hub expects unscaled data: "With default settings it does not matter if the data are normalized or raw. Just make sure they are not scaled"
