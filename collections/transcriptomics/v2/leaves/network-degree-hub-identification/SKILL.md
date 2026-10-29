---
name: network-degree-hub-identification
description: Use when scLink networks have been inferred for multiple conditions or
  time points and you need to identify genes whose connectivity (hub status) changes,
  e.g., 0 h vs 96 h of DE differentiation.
when_to_use_negative:
- Edges are directional or signed regulatory interactions — scLink edges are undirected
  statistical dependencies, so degree reflects co-expression connectivity only
- Only a single condition/network is available with no second time point or condition
  to compare degree against
- The input is raw single-cell counts that have not been normalized with sclink_norm
  and filtered to a gene set
edam_operation: http://edamontology.org/operation_3562
edam_topics:
- http://edamontology.org/topic_0605
- http://edamontology.org/topic_3170
tools:
- name: scLink
  role: Provides sclink_net to compute the robust correlation matrix and sparse adjacency
    matrix whose row sums yield per-gene degree; sclink_norm pre-processes raw counts
  repo: https://github.com/Vivianstats/scLink
- name: R
  role: Environment for running the scLink package, computing adjacency row sums,
    and writing degree_changes.csv
  repo: https://github.com/Vivianstats/scLink
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.gpb.2020.11.006
    title: 'scLink: Inferring Sparse Gene Co-Expression Networks from Single-Cell
      Expression Data'
  source_content_hash: sha256:4494382bb03699a606fa5b43688598a078476e48db069f9713dde7dd71120dd6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Vivianstats/scLink
  tools:
  - R
  - scLink
  edam_operation: http://edamontology.org/operation_3562
  edam_topics:
  - http://edamontology.org/topic_0605
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# network-degree-hub-identification

## Summary

Compute per-gene degree as row sums of scLink adjacency matrices (e.g., across definitive endoderm differentiation time points from GSE75748) and compare degree differences to identify hub genes such as LHX1, HAPLN1, and GNG11.

## When to use

Use when scLink networks have been inferred for multiple conditions or time points and you need to identify genes whose connectivity (hub status) changes, e.g., 0 h vs 96 h of DE differentiation.

## When NOT to use

- Edges are directional or signed regulatory interactions — scLink edges are undirected statistical dependencies, so degree reflects co-expression connectivity only
- Only a single condition/network is available with no second time point or condition to compare degree against
- The input is raw single-cell counts that have not been normalized with sclink_norm and filtered to a gene set

## Inputs

- scLink adjacency matrices (one per time point/condition), output of sclink_net
- scLink robust correlation matrices from sclink_net
- Normalized count matrix produced by sclink_norm (10^6 library size, log10 scale) restricted to the gene set of interest (e.g., 51 lineage marker genes from GSE75748)

## Outputs

- Per-gene degree values per time point (row sums of each adjacency matrix)
- Degree-change table (e.g., 0 h vs 96 h differences) written to degree_changes.csv, with hub genes such as LHX1, HAPLN1, GNG11 among top entries

## How to apply

After fitting scLink's penalized Gaussian graphical model with sclink_net (BIC-selected over lda = seq(0.5, 0.1, -0.05)) on sclink_norm-normalized counts (10^6 library size, log10 scale, filter.genes=FALSE for a fixed gene set such as the 51 lineage markers), extract each time point's binary adjacency matrix. Compute per-gene degree as row sums of the adjacency matrix at each time point, then tabulate degree differences between conditions (e.g., 0 h vs 96 h) to rank hub changes. Genes with the largest positive degree shifts are candidate hubs; in the GSE75748 DE differentiation analysis this flags LHX1, HAPLN1, and GNG11. Write the degree-change table to degree_changes.csv alongside the companion correlation_changes.csv from the pairwise-correlation analysis.

## Related tools

- **scLink** (Provides sclink_net to compute the robust correlation matrix and sparse adjacency matrix whose row sums yield per-gene degree; sclink_norm pre-processes raw counts) — https://github.com/Vivianstats/scLink
- **R** (Environment for running the scLink package, computing adjacency row sums, and writing degree_changes.csv) — https://github.com/Vivianstats/scLink

## Examples

```
norm <- sclink_norm(counts, filter.genes = FALSE); fit <- sclink_net(norm, lda = seq(0.5, 0.1, -0.05)); degree <- rowSums(fit$adj != 0)
```

## Evaluation signals

- degree_changes.csv exists and is parseable in R/Python
- Top 0 h-to-96 h degree changes include LHX1, HAPLN1, and GNG11
- Degree values are consistent with the number of edges in each time point's scLink adjacency matrix (degree = row sums)
- Analysis restricted to the 51 lineage marker genes (1275 unique pairs) from the GSE75748 DE differentiation data

## Limitations

- scLink edges are undirected statistical dependencies, so degree cannot distinguish regulatory from co-regulatory or functionally coordinated interactions
- Imputed data may still be much sparser than bulk data, and imputation can be time-consuming depending on cell number
- No software version is specified for the scLink R package, complicating exact reproducibility

## Evidence

- [methods] use the function `sclink_net` to calculate the robust correlation matrix and identifed sparse co-expression network of scLink: "use the function `sclink_net` to calculate the robust correlation matrix and identifed sparse co-expression network of scLink"
- [methods] normalize the read count matrix with a library size of 10^6: "normalize the read count matrix with a library size of $10^6$ and only keep the 500 genes in `genes` for downstream analysis"
- [readme] The scLink R package is available at https://github.com/Vivianstats/scLink.: "The package is available on CRAN. For installation please use the
following codes in `R`

 r
install.packages("scLink")
"
- [discussion] the edges identified by scLink are based on statistical dependencies and do not have directions: "the edges identified by scLink are based on statistical dependencies and do not have directions"
