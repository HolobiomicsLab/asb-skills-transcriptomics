---
name: graphical-model-regularization-tuning
description: Use when inferring a sparse gene co-expression network from normalized
  single-cell expression data and the penalization strength (lda) of a Gaussian graphical
  model must be selected by BIC.
when_to_use_negative:
- When Pearson/Spearman correlations or bulk RNA-seq co-expression methods already
  suffice — scLink's penalized graphical model is motivated by excessive zero counts
  in sparse single-cell data
- When the gene list is very large; the article suggests first dividing genes into
  major modules based on scLink's correlation and applying scLink separately to each
  module instead of fitting one network
- When edge direction or edge type (regulatory vs co-regulatory) is required — scLink
  edges are undirected and their type cannot be distinguished from expression data
  alone
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: scLink
  role: Implements sclink_net, which computes the robust correlation matrix and fits
    the penalized Gaussian graphical model with BIC-based tuning of lda
  repo: https://github.com/Vivianstats/scLink
- name: scImpute
  role: Prior method whose handling of excess zeros partially motivated scLink's first
    step (not required in this workflow)
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.gpb.2020.11.006
    title: 'scLink: Inferring Sparse Gene Co-Expression Networks from Single-Cell
      Expression Data'
  source_content_hash: sha256:f66ceb92db7ed82a66b40077b70f28d5aad24a0e5b60531a38d7cda5a6557e17
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Vivianstats/scLink
  tools:
  - R
  - scImpute
  - scLink
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# graphical-model-regularization-tuning

## Summary

Tunes the penalized Gaussian graphical model of scLink by sweeping the regularization parameter grid (lda) and selecting the value by BIC so each inferred gene co-expression network stays sparse (at most ~5% of possible edges).

## When to use

Use when inferring a sparse gene co-expression network from normalized single-cell expression data and the penalization strength (lda) of a Gaussian graphical model must be selected by BIC.

## When NOT to use

- When Pearson/Spearman correlations or bulk RNA-seq co-expression methods already suffice — scLink's penalized graphical model is motivated by excessive zero counts in sparse single-cell data
- When the gene list is very large; the article suggests first dividing genes into major modules based on scLink's correlation and applying scLink separately to each module instead of fitting one network
- When edge direction or edge type (regulatory vs co-regulatory) is required — scLink edges are undirected and their type cannot be distinguished from expression data alone

## Inputs

- Normalized single-cell expression matrix produced by sclink_norm (scale.factor = 1e6, log10 scale), e.g. top 500 genes by average expression from GSE114727 immune cells (211 matched-normal and 656 tumor cells)
- Regularization parameter grid, lda = seq(0.5, 0.1, -0.05)

## Outputs

- Sparse scLink co-expression network per condition with BIC-selected regularization
- Robust correlation matrix (networks$cor) per condition
- Adjacency matrix and concentration matrix summaries (networks$summary$adj, networks$summary$Sigma)
- Differential-edge table (e.g. differential_edges.csv) with per-edge change in scLink correlation and FDR-adjusted bootstrap significance

## How to apply

Normalize the single-cell count matrix first with sclink_norm (library size 1e6, log scale) so downstream penalties act on scLink's robust correlation scale. Run sclink_net separately on each condition's normalized matrix with lda = seq(0.5, 0.1, -0.05), letting scLink evaluate each candidate regularization level. Select the regularization parameter by BIC so that each network retains at most 5% of possible edges, ensuring a sparse, data-adaptive penalty strength on each edge. Extract the resulting robust correlation matrix (networks$cor) and the adjacency/concentration summaries (networks$summary$adj and networks$summary$Sigma) for downstream differential-edge analysis. In the GSE114727 breast-cancer immune-cell application, this tuning produced networks whose comparison yielded ~453 normal-only and ~1384 tumor-only differential edges with |change in scLink correlation| > 0.5.

## Related tools

- **scLink** (Implements sclink_net, which computes the robust correlation matrix and fits the penalized Gaussian graphical model with BIC-based tuning of lda) — https://github.com/Vivianstats/scLink
- **scImpute** (Prior method whose handling of excess zeros partially motivated scLink's first step (not required in this workflow))

## Examples

```
networks <- sclink_net(expr = counts.norm, lda = seq(0.5, 0.1, -0.05), model.type = "Gaussian")
```

## Evaluation signals

- Each BIC-selected network retains at most ~5% of possible edges
- Network matrices (networks$cor, networks$summary$adj, networks$summary$Sigma) exist and are symmetric for every condition
- Differential-edge counts at |change in scLink correlation| > 0.5 are consistent with reported values (~453 normal-only, ~1384 tumor-only)
- Spot-checkable rows reproduce reported values, e.g. FNBP1-MGP correlation ~0.77 in normal vs ~-0.03 in tumor, and EGR1-NUDT3 ~-0.48 in normal but absent in tumor
- Most retained differential edges show FDR-adjusted bootstrap P < 0.05

## Limitations

- scLink edges are undirected; regulatory vs co-regulatory edge types cannot be distinguished using only gene expression data
- No software version for the scLink R package is specified in the article or README
- Imputation alternatives can be time-consuming depending on cell number, and imputed data may still be much sparser than bulk data

## Evidence

- [intro] scLink uses a penalized likelihood approach to identify sparse gene networks in a data-adaptive manner: "scLink uses a penalized likelihood approach to identify relatively sparse gene networks in a data-adaptive manner, adjusting the penalty strength on each edge"
- [other] BIC-tuned lda grid over normal and tumor matrices from the reproduced workflow: [evidence span withheld — anchor status: not_found]
- [methods] sclink_net computes the robust correlation matrix and sparse network: "use the function `sclink_net` to calculate the robust correlation matrix and identifed sparse co-expression network of scLink"
- [readme] README lists sclink_net as the network construction function: "sclink_net for constructing the gene co-expression network by
    scLink"
