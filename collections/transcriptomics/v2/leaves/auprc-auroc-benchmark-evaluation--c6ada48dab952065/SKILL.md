---
name: auprc-auroc-benchmark-evaluation
description: Use when comparing gene co-expression network inference methods on simulated
  single-cell data with known ground-truth edges (power-law, hub-based, or no-extra-sparsity
  controls) and quantifying accuracy with AUPRC and AUROC across repeats.
when_to_use_negative:
- Inferring a network from real single-cell data with no known ground-truth adjacency
  — AUPRC/AUROC cannot be computed and downstream biological validation is needed
  instead
- A single network inference run without method comparison — this skill is specifically
  for benchmark scoring against known true edges
- Data with no zero-count sparsity at all (bulk-like expression) — the paper's imputation/simulation
  discussion notes bulk-data methods behave differently, so the dropout-condition
  design does not apply
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: scLink
  role: Primary method under benchmark; sclink_net computes the robust correlation
    matrix and sparse network, with BIC selection over lda = seq(0.5, 0.1, -0.05)
  repo: https://github.com/Vivianstats/scLink
- name: QUIC
  role: Fits the glasso-r variant (penalized graphical model on scLink's robust correlation
    matrix with constant unit edge penalty weights)
- name: PIDC
  role: Baseline method using partial information decomposition to quantify statistical
    dependencies between genes
- name: GENIE3
  role: Baseline method decomposing network prediction into p tree-based ensemble
    regression problems
- name: glasso
  role: Plain graphical lasso baseline on standard correlations of unfiltered data;
    also used for glasso-f after filtering cells with >70% zero counts
- name: R
  role: Implementation environment for simulation, network fitting, and AUPRC/AUROC
    scoring
  repo: https://github.com/Vivianstats/scLink
provenance:
  source_task_ids:
  - task_003
  - task_004
  source_papers:
  - doi: 10.1016/j.gpb.2020.11.006
    title: 'scLink: Inferring Sparse Gene Co-Expression Networks from Single-Cell
      Expression Data'
  source_content_hash: sha256:3dd6ee4cb067d26026320f979d9eecede104b70ca9c33c59e44f756082a7b491
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Vivianstats/scLink
  tools:
  - GENIE3
  - PIDC
  - QUIC
  - R
  - glasso
  - parallel
  - scLink
  - sclink_net
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# auprc-auroc-benchmark-evaluation

## Summary

Benchmark evaluation of gene co-expression network inference methods (scLink, glasso, glasso-r, glasso-f, PIDC, GENIE3, thresholded Pearson/Spearman) on simulated single-cell expression data with known power-law or hub-based ground-truth topologies, scoring each inferred adjacency against the true connectivity matrix via AUPRC and AUROC.

## When to use

Use when comparing gene co-expression network inference methods on simulated single-cell data with known ground-truth edges (power-law, hub-based, or no-extra-sparsity controls) and quantifying accuracy with AUPRC and AUROC across repeats.

## When NOT to use

- Inferring a network from real single-cell data with no known ground-truth adjacency — AUPRC/AUROC cannot be computed and downstream biological validation is needed instead
- A single network inference run without method comparison — this skill is specifically for benchmark scoring against known true edges
- Data with no zero-count sparsity at all (bulk-like expression) — the paper's imputation/simulation discussion notes bulk-data methods behave differently, so the dropout-condition design does not apply

## Inputs

- Simulated single-cell count matrices over power-law and hub-based block-diagonal topologies (50 repeats per condition, dropout rates rho in {0.07, 0.10, 0.13, 0.16} plus no-extra-sparsity control)
- True block-diagonal connectivity matrices from ldstatsHD used as ground-truth adjacency
- Per-gene expression means and standard deviations estimated from GSE114727 single-cell counts as simulation parameters

## Outputs

- Per-repeat AUPRC and AUROC values for each method and network-type/dropout condition
- Aggregated method-by-network AUPRC/AUROC summary table written to simulation_auprc_auroc.csv or figure_s2_auprc_auroc_comparison.csv

## How to apply

Simulate single-cell expression data using per-gene means and standard deviations estimated from a real count matrix (e.g., GEO GSE114727) with ldstatsHD generating block-diagonal power-law and hub-based connectivity matrices; run 50 repeats per condition. Introduce zero counts at dropout rates rho in {0.07, 0.10, 0.13, 0.16} plus a no-extra-sparsity control where this step is skipped. Fit each method on every simulated dataset — sclink_net with ncores=1 over candidate lda = seq(0.5, 0.1, -0.05) selected by BIC, glasso-r (QUIC on scLink's robust correlation matrix with unit penalty weights), glasso-f (glasso after removing cells with >70% zero counts), plain glasso, PIDC, GENIE3, and thresholded Pearson/Spearman correlations. Compare each inferred adjacency against the true connectivity matrix, compute AUPRC and AUROC per repeat, aggregate the 50 repeats per condition, and write the summary table (e.g., simulation_auprc_auroc.csv or figure_s2_auprc_auroc_comparison.csv).

## Related tools

- **scLink** (Primary method under benchmark; sclink_net computes the robust correlation matrix and sparse network, with BIC selection over lda = seq(0.5, 0.1, -0.05)) — https://github.com/Vivianstats/scLink
- **QUIC** (Fits the glasso-r variant (penalized graphical model on scLink's robust correlation matrix with constant unit edge penalty weights))
- **PIDC** (Baseline method using partial information decomposition to quantify statistical dependencies between genes)
- **GENIE3** (Baseline method decomposing network prediction into p tree-based ensemble regression problems)
- **glasso** (Plain graphical lasso baseline on standard correlations of unfiltered data; also used for glasso-f after filtering cells with >70% zero counts)
- **R** (Implementation environment for simulation, network fitting, and AUPRC/AUROC scoring) — https://github.com/Vivianstats/scLink

## Examples

```
net <- sclink_net(counts, ncores = 1, lda = seq(0.5, 0.1, -0.05))
```

## Evaluation signals

- scLink attains the best AUPRC in every power-law and hub-based dropout condition, outperforming glasso, PIDC, GENIE3, and thresholded Pearson/Spearman
- Plain glasso does not beat thresholded Pearson correlation in the benchmark table
- All methods (especially thresholded Pearson/Spearman) improve on the no-extra-sparsity control, reproducing the Figure 2C-D and Figure S3 pattern
- glasso-f does not effectively improve accuracy over plain glasso
- scLink achieves higher AUPRC (with slightly lower AUROC) than glasso-r, indicating the data-adaptive penalty drives the gain

## Limitations

- scLink edges are undirected statistical dependencies; it is not feasible to distinguish regulatory, co-regulatory, and functionally coordinated edge types using only gene expression data
- AUROC is slightly lower for scLink than for glasso-r, so AUPRC (not AUROC) is the primary discriminator in sparse network benchmarks
- Aggregation relies on 50 simulated repeats per condition; results are specific to the simulated power-law/hub topologies and dropout rates tested
- No software version for the scLink R package is specified in the article, complicating exact reproducibility

## Evidence

- [other] Compare each inferred adjacency against the true connectivity matrix and compute AUPRC and AUROC per repeat and setting.: [evidence span withheld — anchor status: not_found]
- [other] scLink achieved the best accuracy for both power-law and hub-based network topologies: [evidence span withheld — anchor status: not_found]
- [abstract] We use both simulation and real data studies to demonstrate the advantages of scLink: "We use both simulation and real data studies to demonstrate the advantages of scLink and its ability to improve single-cell gene network analysis."
- [intro] scLink adapts the Gaussian graphical model to distinguish direct associations: "scLink adapts the Gaussian graphical model [29] to distinguish direct associations between genes from indirect ones and leads to sparse networks"
- [discussion] the edges identified by scLink are based on statistical dependencies and do not have directions: "the edges identified by scLink are based on statistical dependencies and do not have directions"
- [readme] sclink_net for constructing the gene co-expression network by scLink: "`sclink_net` for constructing the gene co-expression network by
    scLink"
