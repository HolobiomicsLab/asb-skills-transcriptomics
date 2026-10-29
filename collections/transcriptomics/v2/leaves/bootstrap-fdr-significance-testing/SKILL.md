---
name: bootstrap-fdr-significance-testing
description: Use when comparing two scLink co-expression networks (e.g., normal vs
  tumor immune cells) and you must annotate differential edges' statistical significance,
  distinguishing reliable normal-only or tumor-only edges from chance changes in scLink
  correlation.
when_to_use_negative:
- You only need a single-condition co-expression network and are not comparing two
  biological states — no differential edge P values are involved.
- You need directed or regulatory interpretation of edges — scLink edges are undirected
  statistical dependencies and the paper notes regulatory vs co-regulatory edges cannot
  be distinguished from expression data alone.
- The data are bulk RNA-seq or otherwise dense — the bootstrap procedure is tied to
  scLink's handling of sparse single-cell count data with excess zeros.
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scLink
  role: Computes the robust correlation matrices (networks$cor) and sparse adjacency
    summaries (networks$summary$adj) whose between-condition differences the bootstrap
    and FDR procedure tests; also used for sclink_norm preprocessing.
  repo: https://github.com/Vivianstats/scLink
- name: R
  role: Implementation environment for sclink_net/sclink_norm calls, the bootstrap
    resampling loop, FDR adjustment, and writing differential_edges.csv.
  repo: https://github.com/Vivianstats/scLink
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.gpb.2020.11.006
    title: 'scLink: Inferring Sparse Gene Co-Expression Networks from Single-Cell
      Expression Data'
  source_content_hash: sha256:9d177beeb32e97af02c49c95fffe73bfd1f8c2eb42cd417df5f295618adb2825
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Vivianstats/scLink
  tools:
  - R
  - scLink
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# Bootstrap FDR significance testing for differential co-expression edges

## Summary

Statistical validation step for scLink differential co-expression: bootstrap resampling is used to attach P values to differential edges between condition-specific networks (e.g., normal vs tumor breast cancer immune cells from GSE114727), and FDR adjustment controls false discoveries among the thousands of tested edges.

## When to use

Use when comparing two scLink co-expression networks (e.g., normal vs tumor immune cells) and you must annotate differential edges' statistical significance, distinguishing reliable normal-only or tumor-only edges from chance changes in scLink correlation.

## When NOT to use

- You only need a single-condition co-expression network and are not comparing two biological states — no differential edge P values are involved.
- You need directed or regulatory interpretation of edges — scLink edges are undirected statistical dependencies and the paper notes regulatory vs co-regulatory edges cannot be distinguished from expression data alone.
- The data are bulk RNA-seq or otherwise dense — the bootstrap procedure is tied to scLink's handling of sparse single-cell count data with excess zeros.

## Inputs

- Normalized scLink expression matrices per condition (sclink_norm output, top 500 genes by average expression)
- Fitted scLink network objects per condition (networks$cor, networks$summary$adj, networks$summary$Sigma) with lda = seq(0.5, 0.1, -0.05) selected by BIC
- Bootstrap resamples of the single-cell count matrices per condition

## Outputs

- Differential-edge table with per-condition scLink correlations, |change in correlation|, edge direction (normal-only vs tumor-only), and FDR-adjusted P values
- differential_edges.csv (~453 normal-only and ~1384 tumor-only edges at |delta correlation| > 0.5)

## How to apply

After running sclink_net separately on each condition (e.g., 211 matched-normal and 656 tumor immune cells from GSE114727, normalized with sclink_norm at scale.factor 1e6 and the top 500 genes by average expression), bootstrap-resample the cells to assess stability of each differential edge and compute P values for the change in scLink correlation between conditions. Adjust P values for multiple testing with FDR correction, since ~453 normal-only and ~1384 tumor-only edges are tested at the |change in correlation| > 0.5 threshold. Note that the regularization parameter lda (e.g., seq(0.5, 0.1, -0.05)) is chosen by BIC so each network keeps at most 5% of possible edges, which constrains the hypothesis space before testing. Annotate each retained differential edge with the per-condition scLink correlations (from networks$cor), adjacency membership (networks$summary$adj), and its FDR-adjusted P; validate known rows such as FNBP1-MGP (~0.77 in normal vs ~-0.03 in tumor) and EGR1-NUDT3 (~-0.48 in normal, absent in tumor).

## Related tools

- **scLink** (Computes the robust correlation matrices (networks$cor) and sparse adjacency summaries (networks$summary$adj) whose between-condition differences the bootstrap and FDR procedure tests; also used for sclink_norm preprocessing.) — https://github.com/Vivianstats/scLink
- **R** (Implementation environment for sclink_net/sclink_norm calls, the bootstrap resampling loop, FDR adjustment, and writing differential_edges.csv.) — https://github.com/Vivianstats/scLink

## Examples

```
Rscript -e 'library(scLink); norm <- sclink_norm(counts, scale.factor=1e6, filter.genes=TRUE, n=500); net <- sclink_net(norm, lda=seq(0.5, 0.1, -0.05)); adj <- net$summary$adj; cor <- net$cor'
```

## Evaluation signals

- differential_edges.csv exists and contains the FNBP1-MGP row with correlation ~0.77 in normal vs ~-0.03 in tumor.
- differential_edges.csv contains the EGR1-NUDT3 row with correlation ~-0.48 in normal and the edge absent in tumor.
- Edge counts per direction are consistent with the reported ~453 normal-only and ~1384 tumor-only edges at |delta correlation| > 0.5.
- Most retained edges have FDR-adjusted P < 0.05 after the bootstrap-based multiple-testing correction.
- Each condition's scLink network retains at most 5% of possible edges, with lda selected by BIC.

## Limitations

- scLink edges are based on statistical dependencies and are undirected; regulatory vs co-regulatory vs coordinated edge types cannot be distinguished from expression data alone.
- Pearson/Spearman correlations are unreliable on sparse single-cell data with excessive zero counts, so the bootstrap significance testing must be applied to scLink's robust correlation, not raw correlations.
- The expected edge counts (~453 normal-only, ~1384 tumor-only) are specific to GSE114727 normal/tumor immune cells; other datasets will not reproduce them exactly.
- No software version for the scLink R package is specified in the article, so exact numerical reproducibility of P values may vary across package versions.

## Evidence

- [methods] use the function `sclink_net` to calculate the robust correlation matrix and identifed sparse co-expression network of scLink: "use the function `sclink_net` to calculate the robust correlation matrix and identifed sparse co-expression network of scLink"
- [discussion] Edges are undirected statistical dependencies: "the edges identified by scLink are based on statistical dependencies and do not have directions"
- [intro] scLink uses a data-adaptive penalized likelihood: "scLink uses a penalized likelihood approach to identify relatively sparse gene networks in a data-adaptive manner, adjusting the penalty strength on each edge based on the observed co-expression"
- [readme] scLink's three main functions: [evidence span withheld — anchor status: not_found]
