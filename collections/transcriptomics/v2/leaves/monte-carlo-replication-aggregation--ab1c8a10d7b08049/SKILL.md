---
name: monte-carlo-replication-aggregation
description: Use when a paper reports averaged simulation results (e.g., Table 2 true-feature-gene
  counts over 100 replications) that must be reproduced, or when a stochastic pipeline's
  performance needs Monte Carlo error bounds.
when_to_use_negative:
- The evaluation target is a single deterministic run rather than an expected value
  over stochastic replications
- Real experimental datasets without simulation ground truth are the input, so 'true
  feature genes' cannot be enumerated per replication
edam_operation: http://edamontology.org/operation_3562
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: EDGE
  role: Feature gene ranking applied identically in every replication via endr(dat_log,
    custom_defs)
  repo: https://github.com/shawnstat/EDGE
- name: splatter
  role: Generates the independent two-group simulated count matrices for each Monte
    Carlo replication
- name: RSpectra
  role: Dependency for EDGE's large-scale eigenvalue decomposition during spectral
    embedding
- name: Rcpp
  role: Prerequisite (>= 1.0.1) for building the EDGE package
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-020-19465-7
    title: Ensemble dimensionality reduction and feature gene extraction for single-cell
      RNA-seq data
  source_content_hash: sha256:b0b4d5287edaf93ed10d293957e39913aebebd0dc362eb3be35e8fb6d5f6e204
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/shawnstat/EDGE
  tools:
  - EDGE
  - RSpectra
  - Rcpp
  - splatter
  edam_operation: http://edamontology.org/operation_3562
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# monte-carlo-replication-aggregation

## Summary

Quantify a method's average performance by running many independent simulated-dataset replications (e.g., 100 Splatter simulations of two-cell-type scRNA-seq data under low/high dropout) and aggregating per-replication metrics, such as true feature genes recovered among EDGE's top-15/top-30 rankings.

## When to use

Use when a paper reports averaged simulation results (e.g., Table 2 true-feature-gene counts over 100 replications) that must be reproduced, or when a stochastic pipeline's performance needs Monte Carlo error bounds.

## When NOT to use

- The evaluation target is a single deterministic run rather than an expected value over stochastic replications
- Real experimental datasets without simulation ground truth are the input, so 'true feature genes' cannot be enumerated per replication

## Inputs

- Simulated scRNA-seq count matrices from splatter (500 genes x 1000 cells, two groups with rare and common cell types)
- Known ground-truth differentially expressed (feature) gene labels per simulation
- An endr_defs configuration object for the EDGE package

## Outputs

- Per-replication counts of true feature genes among the top 15 and top 30 ranked genes
- Averaged true-gene counts per dropout scenario (low vs high), e.g., written to table2_feature_genes.csv

## How to apply

Fix a random seed and, for each of 100 replications, generate an independent simulated dataset — here a two-group scRNA-seq count matrix via splatSimulate(method='groups') with newSplatParams(nGenes=500, batchCells=1000, group.prob yielding a rare and a common cell type) under both low- and high-dropout settings, while recording the simulated differentially expressed genes as ground truth. Apply the identical deterministic pipeline to each replication: normalize counts by sweeping library sizes (rs/rs_med), log2-transform after adding a pseudocount to get dat_log, then rank genes with endr(dat_log, custom_defs) configured with n_wl=1000, n_neigs=15, opt=TRUE, seed=2020. In each replication, count true feature genes among the top 15 and top 30 ranked genes, then average these counts separately per scenario. Validate by comparing the averages to the published targets (15.00/27.06 low dropout; 14.79/25.74 high dropout) within Monte Carlo sampling variability.

## Related tools

- **EDGE** (Feature gene ranking applied identically in every replication via endr(dat_log, custom_defs)) — https://github.com/shawnstat/EDGE
- **splatter** (Generates the independent two-group simulated count matrices for each Monte Carlo replication)
- **RSpectra** (Dependency for EDGE's large-scale eigenvalue decomposition during spectral embedding)
- **Rcpp** (Prerequisite (>= 1.0.1) for building the EDGE package)

## Examples

```
custom_defs <- endr_defs; custom_defs$n_wl <- 1000; custom_defs$n_neigs <- 15; custom_defs$seed <- 2020; simu.endr <- endr(dat_log, custom_defs)
```

## Evaluation signals

- Averaged top-15 true-gene count ≈ 15.00 (low dropout) and ≈ 14.79 (high dropout) across 100 replications
- Averaged top-30 true-gene count ≈ 27.06 (low dropout) and ≈ 25.74 (high dropout) across 100 replications
- Every replication uses the same pipeline parameters (n_wl=1000, n_neigs=15, seed=2020) so between-replication variation reflects simulation randomness only
- Group probabilities in newSplatParams produce a rare and a common cell type in each simulated dataset
- Results agree with published Table 2 within Monte Carlo sampling variability

## Limitations

- Large memory is required for many weak learners and large hash tables; the README notes the optimal hash table size is 1,017,881 but a reduced 101,107 'works well and uses less memory'
- Reproduced averages are subject to Monte Carlo sampling variability, not exact matches to published values
- Validation is against Splatter-simulated two-cell-type data; generalization to other simulation regimes is not established by these numbers

## Evidence

- [results] In simulated two-cell-type datasets, EDGE detected on average 15.00 and 27.06 true feature genes among the top 15 and top 30 ranked genes in the low dropout scenario, and 14.79 and 25.74 in the high dropout scenario over 100 replications.: [evidence span withheld — anchor status: not_found]
- [readme] Normalization by library size scaling to the median followed by log2 transform is the standard pre-processing before EDGE.: "dat_norm <- sweep(counts_dt,2,rs/rs_med,"/")
    dat_log <- t(log2(dat_norm+1))"
- [methods] Splatter simulation parameters fixed per replication with a seed and two-group structure.: "params <- newSplatParams(nGenes=GC, seed=myseed, batchCells=BC, group.prob=gp)
    simu_dt <- splatSimulate(params=params,method="groups")"
- [methods] EDGE endr call with configurable weak learners, neighbors, and seed.: "simu.endr <- endr(dat_log,custom_defs)"
- [readme] The more weak learners used, the better the result, at the cost of memory.: "Theoretically, the more weak learners and larger hash table sizes you use, the better result you will obtain. However, large memory will be required."
