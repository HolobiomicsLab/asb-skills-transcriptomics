---
name: parameter-sensitivity-comparison
description: Use when scoring gene signatures on low-depth or downsampled single-cell
  data with UCell and you need to justify or reproduce a non-default maxRank setting,
  or to verify how missing signature genes affect scores under 'impute' vs 'skip'.
when_to_use_negative:
- Data are high-depth so the default maxRank=1500 cap is adequate and no parameter
  adjustment needs justification
- The task is generic gene signature scoring rather than reproducing or validating
  parameter choices of the UCell Seurat vignette
- Input is a raw expression matrix without a Seurat object and the goal is not to
  compare AddModuleScore_UCell runs
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: UCell
  role: Provides AddModuleScore_UCell/ScoreSignatures_UCell to compute rank-based
    (Mann-Whitney U) signature scores under varying maxRank and missing_genes settings
  repo: https://github.com/carmonalab/UCell
- name: Seurat
  role: Holds the downsampled Zilionis dataset as a Seurat object that AddModuleScore_UCell
    operates on directly
  repo: https://github.com/satijalab/seurat
- name: data.table
  role: Used internally by UCell via the frank function for efficient gene rank computation
- name: future
  role: Enables parallel batching of UCell rank computations when scoring the Seurat
    object
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.csbj.2021.06.043
    title: 'UCell: Robust and scalable single-cell gene signature scoring'
  source_content_hash: sha256:e78b47196d508801e0c8004af3bb69492ce3ce645e85465e3427699de584d0d5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/carmonalab/UCell
  tools:
  - R
  - Seurat
  - UCell
  - data.table
  - future
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.46153846153846156
n_claims: 13
---

# parameter-sensitivity-comparison

## Summary

A reproducibility exercise that quantifies how UCell signature scores on the low-depth downsampled Zilionis lung immune dataset change when the maxRank cap (default 1500 vs adjusted 800-1000) or the missing_genes policy ('impute' vs 'skip') is varied, producing a per-cell comparison table of differing score columns.

## When to use

Use when scoring gene signatures on low-depth or downsampled single-cell data with UCell and you need to justify or reproduce a non-default maxRank setting, or to verify how missing signature genes affect scores under 'impute' vs 'skip'.

## When NOT to use

- Data are high-depth so the default maxRank=1500 cap is adequate and no parameter adjustment needs justification
- The task is generic gene signature scoring rather than reproducing or validating parameter choices of the UCell Seurat vignette
- Input is a raw expression matrix without a Seurat object and the goal is not to compare AddModuleScore_UCell runs

## Inputs

- Downsampled Zilionis lung immune dataset as a Seurat object
- Gene signature list with positive/negative CD8T, CD4, NK signatures and a Myeloid signature containing 'not_a_gene'
- UCell parameter settings: maxRank (1000 vs default 1500), chunk.size=500, w_neg=1.0, missing_genes ('impute' vs 'skip')

## Outputs

- ucell_parameter_comparison_scores.csv containing per-cell score columns for each parameter setting
- Identification of which score columns differ between maxRank=1000 and rmax=1500, and between missing_genes='impute' and 'skip'

## How to apply

Load the downsampled Zilionis dataset as a Seurat object and define the gene signature list with positive and negative CD8T, CD4, and NK signatures plus a Myeloid signature containing the deliberately absent gene 'not_a_gene'. Run AddModuleScore_UCell twice with chunk.size=500 and w_neg=1.0, once with maxRank=1000 and once with the default rmax=1500, which caps ranks to mitigate the uninformative tail of zero-expression genes. Then score the Myeloid signature twice with missing_genes='impute' (sets absent genes to rmax+1) and missing_genes='skip' (drops absent genes from the rank sum). Align the resulting score columns by cell, identify which columns differ between parameter settings, and assemble them into ucell_parameter_comparison_scores.csv. UCell scores are rank-based (normalized Mann-Whitney U statistic, related to AUC), so identical parameter settings must reproduce reference values exactly.

## Related tools

- **UCell** (Provides AddModuleScore_UCell/ScoreSignatures_UCell to compute rank-based (Mann-Whitney U) signature scores under varying maxRank and missing_genes settings) — https://github.com/carmonalab/UCell
- **Seurat** (Holds the downsampled Zilionis dataset as a Seurat object that AddModuleScore_UCell operates on directly) — https://github.com/satijalab/seurat
- **data.table** (Used internally by UCell via the frank function for efficient gene rank computation)
- **future** (Enables parallel batching of UCell rank computations when scoring the Seurat object)

## Examples

```
obj <- AddModuleScore_UCell(obj, features = gene.signatures, maxRank = 1000, chunk.size = 500, w_neg = 1.0, missing_genes = 'impute')
```

## Evaluation signals

- ucell_parameter_comparison_scores.csv exists and contains score columns for each parameter setting keyed by cell
- Rank-based UCell scores for identical parameter settings reproduce the vignette reference values exactly (scores depend only on per-cell gene ranks)
- Differences between runs are confined to columns affected by the changed maxRank or missing_genes settings; other signature columns are identical
- The Myeloid 'not_a_gene' signature yields different scores under missing_genes='impute' (absent genes set to rmax+1) versus 'skip' (absent genes dropped from the rank sum)
- maxRank=1000 run completes with the same UCell normalization formula (1 - U/Umax, Umax = n*maxRank - n(n+1)/2) as the rmax=1500 default run

## Limitations

- maxRank adjustment is data-dependent: the vignette recommends ~800-1000 only because the downsampled Zilionis dataset has relatively low depth; the default is rmax=1500
- Signatures containing genes absent from the dataset require an explicit missing_genes policy; 'impute' assigns them rank rmax+1 while 'skip' drops them, changing the score scale
- Earlier UCell implementations used Umax = n*maxRank (without subtracting n(n+1)/2), so scores may differ across package versions (>=2.7.6 changed the normalization)

## Evidence

- [methods] cap ranks above rmax to rmax + 1 to mitigate the uninformative tail of zero-expression genes: "we set rg,c = rmax + 1 for all rg,c > rmax, with rmax = 1500 by default"
- [methods] UCell scores are rank-based and robust: "This score depends only on the gene expression ranks of individual cell, and therefore is robust across datasets."
- [abstract] UCell wraps Seurat interaction: "includes functions to directly interact with Seurat objects"
- [readme] UCell supports positive and negative gene sets and maxRank parameter: [evidence span withheld — anchor status: not_found]
- [methods] UCell uses data.table frank for ranks and future for parallelism: "Internally, UCell uses the frank function from the data.table package [9] for efficient ranks computations."
