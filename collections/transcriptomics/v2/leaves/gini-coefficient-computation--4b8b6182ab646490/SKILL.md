---
name: gini-coefficient-computation
description: Use when you have factor loadings from a Tensor-cell2cell decomposition
  (e.g. rank=10 BALF COVID-19 tensor) and need a scalar inequality statistic per factor
  to characterize loading concentration across contexts or LR pairs.
when_to_use_negative:
- The tensor contains missing values or cell types not shared across all samples —
  the article notes the algorithm is not optimized for missing values and the COVID-19/ASD
  tensors only include cell types shared across all samples
- You need per-pair communication inference in a single context rather than a concentration
  statistic on loadings of a multi-context decomposition
- Loadings are not comparable across contexts (e.g. different normalization or non-converged
  factorization), which would distort the inequality measure
tools:
- name: cell2cell
  role: Provides the cell2cell.stats module used to compute the Gini coefficient of
    factor loadings
  repo: https://github.com/earmingol/cell2cell
- name: Tensor-cell2cell
  role: Produces the factor loadings (rank=10 non-negative tensor factorization) that
    the Gini coefficient is computed on
  repo: https://github.com/earmingol/cell2cell
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-022-31369-2
    title: Context-aware deconvolution of cell–cell communication with Tensor-cell2cell
  source_content_hash: sha256:fce820e1562b65b8c65c8087868fe66660191ed6159005f9e1ce08d68adec2b4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/earmingol/cell2cell
  tools:
  - CellChat
  - Tensor-cell2cell
  - cell2cell
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# gini-coefficient-computation

## Summary

Computes the Gini coefficient (a measure of loading concentration/inequality) on Tensor-cell2cell factor loadings, e.g. quantifying how concentrated factor-9 loadings are across contexts in the 12x189x6x6 BALF COVID-19 4D-communication tensor, using cell2cell.stats.

## When to use

Use when you have factor loadings from a Tensor-cell2cell decomposition (e.g. rank=10 BALF COVID-19 tensor) and need a scalar inequality statistic per factor to characterize loading concentration across contexts or LR pairs.

## When NOT to use

- The tensor contains missing values or cell types not shared across all samples — the article notes the algorithm is not optimized for missing values and the COVID-19/ASD tensors only include cell types shared across all samples
- You need per-pair communication inference in a single context rather than a concentration statistic on loadings of a multi-context decomposition
- Loadings are not comparable across contexts (e.g. different normalization or non-converged factorization), which would distort the inequality measure

## Inputs

- Factor loadings vector from a Tensor-cell2cell decomposition (e.g. factor-9 context loadings over 12 BALF samples)
- Prebuilt 12 x 189 x 6 x 6 BALF COVID-19 4D-communication tensor (GSE145926; three control, three moderate, six severe patients; CellChat Human-2020-Jin LR pairs)

## Outputs

- Gini coefficient per factor (factor-9 reference value approximately 0.09)
- Rows appended to tensor_factorization_results.csv including factor loadings, Spearman correlations, Gini coefficient, and top LR-pair rankings

## How to apply

After running non-negative tensor factorization with Tensor-cell2cell (rank=10, init='svd', falling back to init='random' on memory errors) on the 12x189x6x6 BALF COVID-19 tensor built from CellChat Human-2020-Jin ligand-receptor scores, extract the context loadings for the factor of interest (e.g. factor 9). Compute the Gini coefficient with the cell2cell.stats module, expecting approximately 0.09 for factor-9 loadings in the reference BALF decomposition. Accept the run if the computed Gini falls within +/-0.02 of 0.09, alongside the companion Spearman checks of factors 1, 2, and 8 (0.72, 0.61, 0.92 +/-0.05). A low Gini indicates loadings spread relatively evenly across the 12 patient contexts, supporting interpretation of the factor as a shared communication module rather than a context-specific one.

## Related tools

- **cell2cell** (Provides the cell2cell.stats module used to compute the Gini coefficient of factor loadings) — https://github.com/earmingol/cell2cell
- **Tensor-cell2cell** (Produces the factor loadings (rank=10 non-negative tensor factorization) that the Gini coefficient is computed on) — https://github.com/earmingol/cell2cell

## Examples

```
from cell2cell.stats import gini; gini_factor9 = gini(factor9_context_loadings)
```

## Evaluation signals

- Factor-9 Gini coefficient falls within +/-0.02 of 0.09
- Companion Spearman correlations of factor 1, 2, and 8 loadings against the reference decomposition fall within +/-0.05 of 0.72, 0.61, and 0.92 respectively
- Top-ranked ligand-receptor pairs by loading include APP/MDK/MIF/CD99, CCL2/CCL3/CCL8 with CCR1/CCR2/CCR5, and MIF-CD74/CD44
- tensor_factorization_results.csv contains loadings, Spearman correlations, the Gini coefficient, and LR-pair rankings with consistent schema

## Limitations

- The Gini is only meaningful on loadings from a converged, non-negative decomposition; 'svd' initialization can raise memory errors, requiring a switch to init='random'
- The implemented tensor decomposition algorithm is not optimized for missing values, so factors from incomplete tensors may bias loading-concentration statistics
- Only cell types shared across all samples were included in the COVID-19 tensors, so rare cell types are absent from the loadings being summarized

## Evidence

- [methods] Compute the Gini coefficient of the factor-9 loadings with cell2cell.stats, expecting approximately 0.09.: [evidence span withheld — anchor status: not_found]
- [methods] Run non-negative tensor factorization with Tensor-cell2cell (cell2cell.tensor module) at rank=10 using init='svd', switching to init='random' if memory errors occur: [evidence span withheld — anchor status: not_found]
- [discussion] Tensor-cell2cell results can be coupled with downstream analysis methods to facilitate interpretation and provide further insights of underlying mechanisms.: "Tensor-cell2cell results can be coupled with downstream analysis methods to facilitate interpretation and provide further insights of underlying mechanisms."
- [discussion] we built a 4D-communication tensor that contains only the cell types that are shared across all samples in our COVID-19 and ASD study cases: "we built a 4D-communication tensor that contains only the cell types that are shared across all samples in our COVID-19 and ASD study cases"
- [readme] For tutorials and documentation, visit [**cell2cell ReadTheDocs**](https://cell2cell.readthedocs.org/en/latest/?badge=latest).: "For tutorials and documentation, visit [**cell2cell ReadTheDocs**](https://cell2cell.readthedocs.org/en/latest/?badge=latest)."
