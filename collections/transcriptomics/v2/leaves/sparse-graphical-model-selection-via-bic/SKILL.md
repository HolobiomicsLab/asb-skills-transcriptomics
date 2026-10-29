---
name: sparse-graphical-model-selection-via-bic
description: Use when fitting scLink to a filtered single-cell expression matrix and
  you must pick the regularization strength (lda) for the sparse co-expression network
  objectively, e.g. via BIC rather than a hand-tuned edge cutoff.
when_to_use_negative:
- When the analysis goal is a plain similarity network from reliable bulk expression
  data — Pearson or Spearman correlation does not require model selection at all
- When gene expression data are heavily zero-inflated and unfiltered without prior
  normalization/gene filtering, since BIC selection on unprocessed scRNA-seq counts
  is unreliable
- When edge directionality is required — scLink edges are undirected statistical dependencies
  and cannot distinguish regulatory from co-regulatory edges
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: scLink
  role: R package providing sclink_net for computing the robust correlation matrix,
    fitting the penalized-likelihood sparse network across the lda grid, and selecting
    the model by minimum BIC
  repo: https://github.com/Vivianstats/scLink
- name: R
  role: Runtime environment for installing scLink from CRAN (install.packages('scLink'))
    and scripting the grid search and BIC-based selection
  repo: https://github.com/Vivianstats/scLink
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.gpb.2020.11.006
    title: 'scLink: Inferring Sparse Gene Co-Expression Networks from Single-Cell
      Expression Data'
  source_content_hash: sha256:ccc1377e0c4367bb776b138b621fd38528028f1f6f4787c5e5f763a87235ea1a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Vivianstats/scLink
  tools:
  - PIDC
  - R
  - scLink
  edam_operation: http://edamontology.org/operation_0564
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# sparse-graphical-model-selection-via-bic

## Summary

Selects scLink's sparse Gaussian graphical-model network over a candidate penalty (lambda) grid by choosing the model with minimum BIC, optionally capping edges at a fraction of all possible edges. This makes network sparsity data-adaptive rather than fixed by an arbitrary threshold.

## When to use

Use when fitting scLink to a filtered single-cell expression matrix and you must pick the regularization strength (lda) for the sparse co-expression network objectively, e.g. via BIC rather than a hand-tuned edge cutoff.

## When NOT to use

- When the analysis goal is a plain similarity network from reliable bulk expression data — Pearson or Spearman correlation does not require model selection at all
- When gene expression data are heavily zero-inflated and unfiltered without prior normalization/gene filtering, since BIC selection on unprocessed scRNA-seq counts is unreliable
- When edge directionality is required — scLink edges are undirected statistical dependencies and cannot distinguish regulatory from co-regulatory edges

## Inputs

- Normalized single-cell count matrix from sclink_norm (top 500 highly expressed genes, library size 1e6)
- Candidate regularization grid, e.g. lda = seq(0.5, 0.1, -0.05)
- Optional edge-density cap (e.g. at most 5% of all possible edges)

## Outputs

- Selected sparse gene co-expression network (TF-gene / gene-gene edges) chosen at minimum-BIC lambda
- BIC values across the lambda grid (implicit via sclink_net model selection)

## How to apply

After normalizing counts with sclink_norm (library size 1e6, filter.genes=TRUE with n=500 top expressed genes), run sclink_net on each cell-type count matrix using a candidate grid such as lda = seq(0.5, 0.1, -0.05). For each lambda value scLink computes the penalized-likelihood sparse network; select the lambda whose model attains the minimum BIC as the final network. In workflows like the Tabula Muris Figure 3A reproduction, further restrict the selected network to at most 5% of all possible edges. The rationale is that scLink's penalized likelihood uses data-adaptive penalty weights per edge, so grid search plus BIC yields a sparse network without user-chosen density thresholds.

## Related tools

- **scLink** (R package providing sclink_net for computing the robust correlation matrix, fitting the penalized-likelihood sparse network across the lda grid, and selecting the model by minimum BIC) — https://github.com/Vivianstats/scLink
- **R** (Runtime environment for installing scLink from CRAN (install.packages('scLink')) and scripting the grid search and BIC-based selection) — https://github.com/Vivianstats/scLink

## Examples

```
install.packages("scLink"); net <- sclink_net(expr = dat_norm, lda = seq(0.5, 0.1, -0.05), model.select = "BIC")
```

## Evaluation signals

- The selected model corresponds to the lambda in the candidate grid (e.g. lda = seq(0.5, 0.1, -0.05)) with the minimum BIC value.
- Final network edge density is capped at or below the specified fraction (e.g. <=5% of all possible edges among the 500 filtered genes).
- Output network contains only genes surviving sclink_norm filtering (top 500 by mean expression), not the full input gene set.
- Running sclink_net over the full grid deterministically reproduces the same selected network for the same normalized input matrix.
- On Tabula Muris Smart-seq2 cell types, the resulting TF-gene overlap proportions (e.g. scLink median 59.3%) remain in the expected range when edges are re-selected via BIC.

## Limitations

- scLink edges are undirected and based on statistical dependencies, so BIC-selected networks cannot distinguish regulatory, co-regulatory, or functionally coordinated edges.
- Penalty grid and BIC are applied after gene filtering (top 500 by mean expression); results depend on this pre-filtering choice.
- No software version for the scLink R package is documented, complicating exact reproduction of BIC-selected networks.

## Evidence

- [intro] scLink uses a penalized likelihood approach to identify relatively sparse gene networks in a data-adaptive manner, adjusting the penalty strength on each edge based on the observed co-expression: "scLink uses a penalized likelihood approach to identify relatively sparse gene networks in a data-adaptive manner, adjusting the penalty strength on each edge based on the observed co-expression"
- [methods] use the function `sclink_net` to calculate the robust correlation matrix and identifed sparse co-expression network of scLink: "use the function `sclink_net` to calculate the robust correlation matrix and identifed sparse co-expression network of scLink"
- [methods] normalize the read count matrix with a library size of $10^6$ and only keep the 500 genes in `genes` for downstream analysis: "normalize the read count matrix with a library size of $10^6$ and only keep the 500 genes in `genes` for downstream analysis"
- [readme] The package is available on CRAN. For installation please use the following codes in `R` install.packages("scLink"): [evidence span withheld — anchor status: not_found]
