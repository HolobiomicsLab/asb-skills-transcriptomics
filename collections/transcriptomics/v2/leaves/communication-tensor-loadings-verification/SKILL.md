---
name: communication-tensor-loadings-verification
description: Use when a completed Tensor-cell2cell factorization (rank=10) must be
  validated against a reference decomposition before its factors are interpreted or
  downstream analyses are trusted.
when_to_use_negative:
- The input is a raw communication-score matrix or single-context CellChat output
  rather than factor loadings from a completed tensor factorization
- The tensor was built with cell types not shared across all samples, so loadings
  are not comparable to the reference COVID-19/ASD decomposition (the paper's tensors
  contain only cell types shared across all samples)
- The factorization was run at a rank other than 10, making the reference factor indices
  and expected correlations (0.72/0.61/0.92) inapplicable
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3308
tools:
- name: Tensor-cell2cell
  role: Produces the rank-10 non-negative tensor factorization whose LR-pair and context
    loadings are verified (cell2cell.tensor module; init='svd' with fallback to init='random'
    on memory errors)
  repo: https://github.com/earmingol/cell2cell
- name: cell2cell
  role: Provides cell2cell.stats for computing the Gini coefficient of factor-9 loadings
    and cell2cell.analysis for downstream analyses of factors
  repo: https://github.com/earmingol/cell2cell
- name: CellChat
  role: Supplies the Human-2020-Jin ligand-receptor pair list used to compute the
    per-context communication scores that build the tensor being verified
  repo: https://github.com/sqjin/CellChat
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-022-31369-2
    title: Context-aware deconvolution of cell–cell communication with Tensor-cell2cell
  source_content_hash: sha256:6dc1f7923ecea1ed8a8327b610db3371ea5409a26efa0c675c23ff11fc462eb0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/earmingol/cell2cell
  tools:
  - CellChat
  - Tensor-cell2cell
  - cell2cell
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# communication-tensor-loadings-verification

## Summary

Verifies that a Tensor-cell2cell decomposition (e.g., the rank-10, 12 x 189 x 6 x 6 BALF COVID-19 4D-communication tensor) reproduces the reference factor loadings by comparing LR-pair and context loadings via Spearman correlations and Gini coefficients, and confirming top-ranked ligand-receptor pairs.

## When to use

Use when a completed Tensor-cell2cell factorization (rank=10) must be validated against a reference decomposition before its factors are interpreted or downstream analyses are trusted.

## When NOT to use

- The input is a raw communication-score matrix or single-context CellChat output rather than factor loadings from a completed tensor factorization
- The tensor was built with cell types not shared across all samples, so loadings are not comparable to the reference COVID-19/ASD decomposition (the paper's tensors contain only cell types shared across all samples)
- The factorization was run at a rank other than 10, making the reference factor indices and expected correlations (0.72/0.61/0.92) inapplicable

## Inputs

- 4D-communication tensor loadings (rank=10) from Tensor-cell2cell, e.g., 12 x 189 x 6 x 6 BALF COVID-19 tensor (GSE145926; 3 control, 3 moderate, 6 severe patients)
- Reference decomposition loadings (LR-pair and context dimensions) for factors 1, 2, 8, and 9
- Communication scores computed per context with the CellChat Human-2020-Jin ligand-receptor pair list

## Outputs

- Spearman rank correlations of factor 1, 2, and 8 loadings vs. reference (expected 0.72, 0.61, 0.92)
- Gini coefficient of factor-9 loadings (expected ~0.09)
- Ranked list of top ligand-receptor pairs per factor (e.g., APP/MDK/MIF/CD99, CCL2/CCL3/CCL8-CCR1/CCR2/CCR5, MIF-CD74/CD44)
- tensor_factorization_results.csv with loadings, correlations, Gini coefficient, and top LR-pair rankings

## How to apply

Load the factorized tensor's LR-pair and context loadings and compute Spearman rank correlations of factor 1, 2, and 8 loadings against the reference decomposition, accepting 0.72, 0.61, and 0.92 within +/-0.05. Compute the Gini coefficient of the factor-9 loadings with cell2cell.stats, expecting approximately 0.09 (tolerance +/-0.02). Rank ligand-receptor pairs by loadings in the relevant factors and confirm that APP/MDK/MIF/CD99, CCL2/CCL3/CCL8 with CCR1/CCR2/CCR5, and MIF-CD74/CD44 appear among the highest-loading entries. If Spearman coefficients are out of tolerance, re-run the non-negative tensor factorization (cell2cell.tensor module) at rank=10 with init='random' instead of init='svd' to exclude initialization-driven divergence.

## Related tools

- **Tensor-cell2cell** (Produces the rank-10 non-negative tensor factorization whose LR-pair and context loadings are verified (cell2cell.tensor module; init='svd' with fallback to init='random' on memory errors)) — https://github.com/earmingol/cell2cell
- **cell2cell** (Provides cell2cell.stats for computing the Gini coefficient of factor-9 loadings and cell2cell.analysis for downstream analyses of factors) — https://github.com/earmingol/cell2cell
- **CellChat** (Supplies the Human-2020-Jin ligand-receptor pair list used to compute the per-context communication scores that build the tensor being verified) — https://github.com/sqjin/CellChat

## Examples

```
factors = tensor.compute_tensor_factorization(rank=10, init='svd')  # fallback: init='random' on memory errors; then spearmanr(factors['lr_loadings'][f], ref_loadings[f])
```

## Evaluation signals

- Spearman correlation of factor-1 loadings vs. reference is within +/-0.05 of 0.72
- Spearman correlation of factor-2 loadings vs. reference is within +/-0.05 of 0.61
- Spearman correlation of factor-8 loadings vs. reference is within +/-0.05 of 0.92 (factor 8 is the most severity-correlated factor)
- Gini coefficient of factor-9 loadings is within +/-0.02 of 0.09
- Top-ranked LR pairs include APP/MDK/MIF/CD99, CCL2/CCL3/CCL8 with CCR1/CCR2/CCR5, and MIF-CD74/CD44, and results are written to tensor_factorization_results.csv

## Limitations

- The implemented tensor decomposition algorithm is not optimized for missing values, so cell types or LR pairs absent in some contexts must be dropped before loadings are comparable
- init='svd' can raise memory errors at this tensor size; the documented fallback is init='random', which may alter exact loading values
- Only cell types shared across all samples were included in the COVID-19 tensor, potentially missing rare cell types and limiting generalization of loading verification
- Reproducibility depends on exact code and software versions deposited in the Code Ocean capsule (https://doi.org/10.24433/CO.0051950.v2); differing dependency versions may shift loadings

## Evidence

- [other] Factor 8 was the most correlated with severity of the disease, with a Spearman coefficient of 0.92: [evidence span withheld — anchor status: not_found]
- [methods] the latter is for deconvolving complex patterns: "the latter is for deconvolving complex patterns of cell-cell communication across multiple samples or contexts simultaneously into interpretable factors"
- [discussion] we built a 4D-communication tensor that contains only the cell types that are shared across all samples: "we built a 4D-communication tensor that contains only the cell types that are shared across all samples in our COVID-19 and ASD study cases"
- [results] we use a list of LR pairs published with CellChat: "we use a list of LR pairs published with CellChat (Jin et al. 2021, Nature Communications)"
- [readme] Install cell2cell via pip: [evidence span withheld — anchor status: not_found]
