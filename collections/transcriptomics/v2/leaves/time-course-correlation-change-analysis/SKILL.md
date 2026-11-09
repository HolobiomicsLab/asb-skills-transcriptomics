---
name: time-course-correlation-change-analysis
description: Use when a single scRNA-seq dataset spans multiple time points (e.g.
  GSE75748 definitive endoderm 0/12/24/36/72/96 h) and the goal is to quantify co-expression
  correlation changes or hub-gene degree shifts between time points.
when_to_use_negative:
- Data come from a single condition or time point with no temporal contrast — per-time-point
  correlation-change comparison is undefined
- The goal is directed regulatory or causal edges between genes — scLink edges are
  undirected statistical dependencies
- Gene list is very large — the article suggests instead dividing genes into modules
  based on scLink's correlation and applying scLink separately to each module
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: scLink
  role: 'Core method: sclink_norm normalizes counts to 10^6 library size, sclink_net
    fits the penalized Gaussian graphical model and returns robust correlation and
    adjacency matrices per time point'
  repo: https://github.com/Vivianstats/scLink
- name: R
  role: Implementation language for scLink package and downstream correlation-change
    and degree-change tables
  repo: https://github.com/Vivianstats/scLink
- name: scImpute
  role: Related prior method; scLink's first step is partially motivated by scImpute
    for handling excess zeros, but it is not required in this workflow
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.gpb.2020.11.006
    title: 'scLink: Inferring Sparse Gene Co-Expression Networks from Single-Cell
      Expression Data'
  source_content_hash: sha256:72f7090332c23424a0b70435a750c490580d66001712dcb9be817e0ea68d4e8e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Vivianstats/scLink
  tools:
  - R
  - scImpute
  - scLink
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# time-course-correlation-change-analysis

## Summary

Detect changes in gene co-expression and network structure across a differentiation or perturbation time course by fitting scLink penalized Gaussian graphical models per time point and comparing robust correlation matrices and adjacency-derived degrees across marker-gene pairs.

## When to use

Use when a single scRNA-seq dataset spans multiple time points (e.g. GSE75748 definitive endoderm 0/12/24/36/72/96 h) and the goal is to quantify co-expression correlation changes or hub-gene degree shifts between time points.

## When NOT to use

- Data come from a single condition or time point with no temporal contrast — per-time-point correlation-change comparison is undefined
- The goal is directed regulatory or causal edges between genes — scLink edges are undirected statistical dependencies
- Gene list is very large — the article suggests instead dividing genes into modules based on scLink's correlation and applying scLink separately to each module

## Inputs

- scRNA-seq raw count matrix spanning multiple time points (e.g. GSE75748 DE differentiation, 758 cells x 0/12/24/36/72/96 h)
- List of target genes (e.g. 51 lineage marker genes from GEO)
- Regularization sequence lda = seq(0.5, 0.1, -0.05)
- Correlation-change threshold (absolute change > 0.5)

## Outputs

- Per-time-point scLink robust correlation matrix
- Per-time-point scLink adjacency (network) matrix selected by BIC
- Marker-pair correlation-change table (correlation_changes.csv) flagging pairs with |Δ correlation| > 0.5 out of 1275 pairs
- Gene degree-change table (degree_changes.csv) ranking hub genes by 0 h vs 96 h degree difference

## How to apply

Download the time-course scRNA-seq count matrix (e.g. GSE75748: 758 cells across 0/12/24/36/72/96 h) and the gene set of interest (e.g. 51 lineage marker genes) from GEO. Normalize raw counts per cell to a 10^6 library size on the log10 scale and subset to the gene set with scLink's sclink_norm (filter.genes=FALSE to retain all marker genes). For each time point, fit scLink's penalized Gaussian graphical model with sclink_net over the candidate regularization sequence lda = seq(0.5, 0.1, -0.05), selecting the network per time point by BIC, and extract the robust correlation matrix and adjacency matrix. Compute pairwise correlation changes across time points for the marker-gene pairs (51 x 50 / 2 = 1275 pairs), flagging pairs with absolute change > 0.5, and compute per-gene degree as row sums of each time point's adjacency matrix to identify hub genes whose degree changes most (e.g. 0 h vs 96 h).

## Related tools

- **scLink** (Core method: sclink_norm normalizes counts to 10^6 library size, sclink_net fits the penalized Gaussian graphical model and returns robust correlation and adjacency matrices per time point) — https://github.com/Vivianstats/scLink
- **R** (Implementation language for scLink package and downstream correlation-change and degree-change tables) — https://github.com/Vivianstats/scLink
- **scImpute** (Related prior method; scLink's first step is partially motivated by scImpute for handling excess zeros, but it is not required in this workflow)

## Examples

```
norm <- sclink_norm(counts, filter.genes = FALSE); net <- sclink_net(norm, lda.seq = seq(0.5, 0.1, -0.05)); corr <- net$corr; adj <- net$adj
```

## Evaluation signals

- correlation_changes.csv exists and is parseable, and reports 240 of 1275 marker-gene pairs with absolute correlation change above the 0.5 threshold
- correlation_changes.csv contains the NANOG-PECAM1 and MT1X-SOX17 rows among flagged pairs
- degree_changes.csv exists and is parseable, and lists LHX1, HAPLN1, and GNG11 among the top 0 h-to-96 h hub changes
- Per-time-point networks were each selected by BIC over the lda sequence 0.5 to 0.1 in steps of -0.05
- All 51 marker genes retained after normalization (filter.genes=FALSE), preserving the 1275 = 51*50/2 pair space

## Limitations

- scLink edges are based on statistical dependencies and have no directions; regulatory, co-regulatory, and functionally coordinated edge types cannot be distinguished from expression data alone
- No software version for the scLink R package is specified, and no validation code/data availability beyond the GitHub repository is documented for reproducing real-data analyses
- Pearson/Spearman correlations are unreliable on sparse single-cell data with excessive zeros; imputed data may remain much sparser than bulk data, so bulk network methods perform poorly and imputation itself can be time-consuming
- For very large gene sets, fitting scLink directly may be infeasible; the article recommends module subdivision

## Evidence

- [abstract] uses statistical network modeling to understand the co-expression relationships among genes and construct sparse gene co-expression networks from single-cell gene expression data: "uses statistical network modeling to understand the co-expression relationships among genes and construct sparse gene co-expression networks from single-cell gene expression data"
- [intro] The gene expression dataset of definitive endoderm (DE) differentiation is available at GEO (GEO: GSE75748: "The gene expression dataset of definitive endoderm (DE) differentiation is available at GEO (GEO: GSE75748"
- [methods] normalize the read count matrix with a library size of $10^6$ and only keep the 500 genes in `genes` for downstream analysis: "normalize the read count matrix with a library size of $10^6$ and only keep the 500 genes in `genes` for downstream analysis"
- [methods] use the function `sclink_net` to calculate the robust correlation matrix and identifed sparse co-expression network: "use the function `sclink_net` to calculate the robust correlation matrix and identifed sparse co-expression network"
- [discussion] the edges identified by scLink are based on statistical dependencies and do not have directions: "the edges identified by scLink are based on statistical dependencies and do not have directions"
- [readme] `scLink` has three main functions: `sclink_norm` for pre-processing gene expression data, `sclink_cor` for calculating the co-expression matrix: "`scLink` has three main functions:

-   `sclink_norm` for pre-processing gene expression data
-   `sclink_cor` for calculating the co-expression matrix by scLink"
