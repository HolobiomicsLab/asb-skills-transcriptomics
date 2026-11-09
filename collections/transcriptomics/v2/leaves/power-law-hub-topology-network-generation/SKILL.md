---
name: power-law-hub-topology-network-generation
description: Use when benchmarking network inference methods (e.g. scLink vs glasso,
  PIDC, GENIE3, thresholded Pearson/Spearman) on simulated sparse single-cell data
  where the true adjacency matrix must be known, especially to compare power-law vs
  hub-based topologies and varying sparsity levels.
when_to_use_negative:
- The goal is to infer a network from real experimental data with no ground truth
  — this skill only generates synthetic benchmarks.
- A directed or signed regulatory network is required; the paper's simulated co-expression
  edges are undirected statistical dependencies.
- The analysis targets bulk RNA-seq sparsity levels; the dropout injection here models
  single-cell excess zeros.
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3524
tools:
- name: ldstatsHD
  role: R package used to generate block-diagonal connectivity matrices with power-law
    and hub-based topologies and sample multivariate Gaussian data
- name: scLink
  role: Target benchmark method whose simulation study these synthetic datasets feed;
    data parameters are estimated to match its workflow (sclink_norm/sclink_net)
  repo: https://github.com/Vivianstats/scLink
- name: R
  role: Language in which the simulation pipeline (ldstatsHD sampling, zero injection,
    sclink_net/baseline inference) is implemented
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.gpb.2020.11.006
    title: 'scLink: Inferring Sparse Gene Co-Expression Networks from Single-Cell
      Expression Data'
  source_content_hash: sha256:0d9993778917fd23fdbdf18265de7691e647717877b46c35c9af247d69d146cb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Vivianstats/scLink
  tools:
  - GENIE3
  - PIDC
  - R
  - glasso
  - ldstatsHD
  - scLink
  - sclink_net
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3524
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# power-law-hub-topology-network-generation

## Summary

Generate synthetic single-cell expression datasets with known ground-truth gene co-expression networks using block-diagonal connectivity matrices with power-law and hub-based topologies (via ldstatsHD), sampling multivariate Gaussian expression data whose per-gene means and standard deviations are estimated from real single-cell counts (GSE114727), and optionally injecting dropout zeros at controlled rates.

## When to use

Use when benchmarking network inference methods (e.g. scLink vs glasso, PIDC, GENIE3, thresholded Pearson/Spearman) on simulated sparse single-cell data where the true adjacency matrix must be known, especially to compare power-law vs hub-based topologies and varying sparsity levels.

## When NOT to use

- The goal is to infer a network from real experimental data with no ground truth — this skill only generates synthetic benchmarks.
- A directed or signed regulatory network is required; the paper's simulated co-expression edges are undirected statistical dependencies.
- The analysis targets bulk RNA-seq sparsity levels; the dropout injection here models single-cell excess zeros.

## Inputs

- Single-cell read count matrix used to estimate per-gene simulation parameters (e.g. GEO accession GSE114727)
- Target network size / number of genes and number of simulation repeats (paper: 50)
- Choice of topology: power-law or hub-based
- Dropout rates rho in {0.07, 0.10, 0.13, 0.16} plus a no-extra-sparsity control

## Outputs

- Block-diagonal true connectivity (adjacency) matrices with power-law or hub-based topology
- Simulated multivariate Gaussian expression count matrices with injected dropout zeros
- Per-repeat ground-truth/inferred pairs ready for AUPRC and AUROC computation

## How to apply

First estimate per-gene expression means and standard deviations from a real single-cell count matrix (the paper uses the GSE114727 immune-cell breast cancer dataset) to parameterize the simulation. Use ldstatsHD to build block-diagonal connectivity matrices under power-law and hub-based topologies, then sample multivariate Gaussian expression data for each repeat (the paper uses 50 repeats). Introduce zero counts at dropout rates rho in {0.07, 0.10, 0.13, 0.16}, and include a no-extra-sparsity control where the zero-introduction step is skipped. Retain the true connectivity matrix per setting as the ground truth against which inferred networks are scored via AUPRC and AUROC, so topological structure and sparsity effects on method accuracy can be separated.

## Related tools

- **ldstatsHD** (R package used to generate block-diagonal connectivity matrices with power-law and hub-based topologies and sample multivariate Gaussian data)
- **scLink** (Target benchmark method whose simulation study these synthetic datasets feed; data parameters are estimated to match its workflow (sclink_norm/sclink_net)) — https://github.com/Vivianstats/scLink
- **R** (Language in which the simulation pipeline (ldstatsHD sampling, zero injection, sclink_net/baseline inference) is implemented)

## Examples

```
library(ldstatsHD); Omega <- networkgeneration(p = n_genes, graph = 'power-law'); X <- mvrnorm(n_cells, mu = gene_means, Sigma = solve(Omega))
```

## Evaluation signals

- The true connectivity matrix is block-diagonal and exhibits the intended degree structure (power-law vs hub) for each topology setting.
- Empirical marginal mean and standard deviation of simulated genes match those estimated from the GSE114727 count matrix.
- Observed sparsity of zero-injected datasets approximates the target dropout rates rho in {0.07, 0.10, 0.13, 0.16}; the control setting has no extra zeros beyond baseline sparsity.
- Across 50 repeats, aggregated AUPRC/AUROC reproduce the paper's pattern: scLink best in power-law and hub conditions, with all methods improving on the no-extra-sparsity control.
- Each simulated dataset ships with its ground-truth adjacency so AUPRC and AUROC can be computed against it.

## Limitations

- Edges represent undirected statistical dependencies; the simulated ground truth cannot distinguish regulatory, co-regulatory, or functionally coordinated edge types.
- Simulated data are multivariate Gaussian and may not capture all real single-cell distributional quirks beyond injected dropout zeros.
- Simulation results depend on the parameter source dataset (GSE114727); conclusions may not transfer to tissues or platforms with different expression profiles.

## Evidence

- [results] Estimate per-gene expression means and standard deviations from the GSE114727 single-cell count matrix (GEO accession GSE114727) to use as simulation parameters.: [evidence span withheld — anchor status: not_found]
- [results] Use ldstatsHD to generate block-diagonal connectivity matrices with power-law and hub-based topologies, sampling multivariate Gaussian expression data: [evidence span withheld — anchor status: not_found]
- [results] Introduce zero counts at dropout rates rho in {0.07, 0.10, 0.13, 0.16}, plus a no-extra-sparsity control where the zero-introduction step is skipped.: [evidence span withheld — anchor status: not_found]
- [intro] The gene expression dataset of immune cells from breast cancer patients is available at Gene Expression Omnibus (GEO: GSE114727: "The gene expression dataset of immune cells from breast cancer patients is available at Gene Expression Omnibus (GEO: GSE114727"
- [discussion] the edges identified by scLink are based on statistical dependencies and do not have directions: "the edges identified by scLink are based on statistical dependencies and do not have directions"
