---
name: gaussian-graphical-model-penalized-estimation
description: Use when a single-cell read-count matrix must be turned into a sparse,
  undirected gene co-expression network that separates direct gene-gene dependencies
  from indirect ones, and standard Pearson/Spearman correlations plus plain glasso
  are unreliable due to excessive zero counts.
when_to_use_negative:
- The goal is directional regulation or distinguishing regulatory vs co-regulatory
  vs functionally coordinated edges — expression data alone cannot support this, and
  scLink edges are undirected
- 'Data are bulk expression or already-imputed single-cell data treated as bulk: imputed
  data may still be much sparser than bulk data, so bulk-data network methods perform
  poorly and this skill''s single-cell robust-correlation rationale does not apply'
- A gene-gene correlation structure survey is all that is needed — use sclink_cor
  first instead of the full penalized network fit
edam_operation: http://edamontology.org/operation_0279
edam_topics:
- http://edamontology.org/topic_0602
- http://edamontology.org/topic_3308
tools:
- name: scLink
  role: R package providing sclink_norm (library-size 10^6 normalization), sclink_cor
    (robust correlation matrix), and sclink_net (penalized Gaussian graphical model
    with data-adaptive edge penalty weights, BIC model selection)
  repo: https://github.com/Vivianstats/scLink
- name: QUIC
  role: 'Solves the QUIC-based penalized graphical model used for the glasso-r ablation:
    penalized estimation on scLink''s robust correlation matrix with constant unit
    penalty weights on every edge'
- name: R
  role: Runtime environment for the scLink package, the QUIC glasso-r ablation, glasso-f
    (glasso on cells filtered for >70% zero counts), and AUPRC/AUROC evaluation
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.gpb.2020.11.006
    title: 'scLink: Inferring Sparse Gene Co-Expression Networks from Single-Cell
      Expression Data'
  source_content_hash: sha256:be397801b7abf1cc0db19baa18ebd2a97fc22cea9514c7b6d3910507d59531b3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Vivianstats/scLink
  tools:
  - QUIC
  - R
  - parallel
  - scLink
  edam_operation: http://edamontology.org/operation_0279
  edam_topics:
  - http://edamontology.org/topic_0602
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# gaussian-graphical-model-penalized-estimation

## Summary

Infers a sparse gene co-expression network from single-cell expression data by adapting the Gaussian graphical model with a penalized likelihood that assigns data-adaptive penalty weights to each edge based on scLink's robust correlation, rather than relying on plain graphical lasso on standard correlations.

## When to use

Use when a single-cell read-count matrix must be turned into a sparse, undirected gene co-expression network that separates direct gene-gene dependencies from indirect ones, and standard Pearson/Spearman correlations plus plain glasso are unreliable due to excessive zero counts.

## When NOT to use

- The goal is directional regulation or distinguishing regulatory vs co-regulatory vs functionally coordinated edges — expression data alone cannot support this, and scLink edges are undirected
- Data are bulk expression or already-imputed single-cell data treated as bulk: imputed data may still be much sparser than bulk data, so bulk-data network methods perform poorly and this skill's single-cell robust-correlation rationale does not apply
- A gene-gene correlation structure survey is all that is needed — use sclink_cor first instead of the full penalized network fit

## Inputs

- Single-cell gene read count matrix (cells x genes)
- Vector of gene names/identifiers for the ~500 genes retained after normalization and filtering
- Candidate regularization grid, e.g. lda = seq(0.5, 0.1, -0.05)
- Optional true simulated adjacency matrix (power-law or hub-based topology) for accuracy evaluation

## Outputs

- Sparse undirected gene co-expression network (edge list / precision-matrix support) from sclink_net
- Robust correlation matrix of scLink
- Model selection diagnostics (BIC from the summary element over the lda grid)
- Method-by-network comparison table of AUPRC and AUROC (e.g. figure_s2_auprc_auroc_comparison.csv)

## How to apply

First normalize the read count matrix with sclink_norm using a library size of 10^6, keeping the ~500 genes selected for analysis (optionally with filter.genes=TRUE or a detection-rate/mean-expression gene filter). Compute scLink's robust correlation matrix, which is built from cells in which both genes are accurately measured with high confidence. Fit the penalized Gaussian graphical model with sclink_net, setting ncores=1 and sweeping candidate regularization parameters lda = seq(0.5, 0.1, -0.05); select the final model by BIC from the returned summary element. The rationale is that scLink adjusts the penalty strength on each edge based on the observed robust co-expression, so it recovers direct associations that plain glasso on standard correlations, glasso-r (QUIC on the robust correlation with unit penalty weights), or glasso-f (glasso after removing cells with >70% zero counts) miss; the adaptive penalty, not the robust correlation alone, drives the accuracy gain in AUPRC.

## Related tools

- **scLink** (R package providing sclink_norm (library-size 10^6 normalization), sclink_cor (robust correlation matrix), and sclink_net (penalized Gaussian graphical model with data-adaptive edge penalty weights, BIC model selection)) — https://github.com/Vivianstats/scLink
- **QUIC** (Solves the QUIC-based penalized graphical model used for the glasso-r ablation: penalized estimation on scLink's robust correlation matrix with constant unit penalty weights on every edge)
- **R** (Runtime environment for the scLink package, the QUIC glasso-r ablation, glasso-f (glasso on cells filtered for >70% zero counts), and AUPRC/AUROC evaluation)

## Examples

```
net <- sclink_net(expr, ncores = 1, lda = seq(0.5, 0.1, -0.05))
```

## Evaluation signals

- Model selected by BIC from the summary element over the lda = seq(0.5, 0.1, -0.05) grid
- Against a known simulated adjacency (power-law or hub-based), scLink attains higher AUPRC than glasso-r, with slightly lower AUROC
- glasso-f (filtering cells with >70% zero counts before glasso) shows no effective AUPRC/AUROC improvement over plain glasso — if it does, the comparison implementation is suspect
- Resulting network is sparse and its edges reflect scLink's robust correlation structure rather than plain Pearson/Spearman correlations

## Limitations

- Edges are undirected statistical dependencies; regulatory, co-regulatory, and functionally coordinated edge types cannot be distinguished from expression data alone
- No software version specified for the scLink R package in the article or README; no validation code/data beyond the GitHub repository for reproducing real-data analyses
- Pearson/Spearman correlations are unreliable under excessive zero counts, so the robust-correlation step is essential; plain glasso baselines suffer as a result
- Applied adaptively per edge, so results depend on the candidate lda grid; modular analyses may require dividing genes into modules based on scLink's correlation before fitting separately

## Evidence

- [abstract] abstract: "uses statistical network modeling to understand the co-expression relationships among genes and construct sparse gene co-expression networks from single-cell gene expression data"
- [intro] intro: "scLink uses a penalized likelihood approach to identify relatively sparse gene networks in a data-adaptive manner, adjusting the penalty strength on each edge based on the observed co-expression"
- [methods] methods: "use the function `sclink_net` to calculate the robust correlation matrix and identifed sparse co-expression network of scLink"
- [methods] methods_norm: "We use the function `sclink_norm` to process single cell read count for application of the sclink method"
- [readme] readme_functions: [evidence span withheld — anchor status: external_doc]
