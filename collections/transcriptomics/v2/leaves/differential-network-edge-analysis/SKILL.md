---
name: differential-network-edge-analysis
description: Use when you have single-cell count matrices from two matched conditions
  (e.g., 211 normal vs 656 tumor immune cells from GSE114727) and want to detect co-expression
  edges present in one condition but absent in the other.
when_to_use_negative:
- The analysis goal is directed regulatory relationships — scLink edges are based
  on statistical dependencies and do not have directions
- You only have bulk RNA-seq data — imputed single-cell-oriented networks may still
  be much sparser than bulk data, and bulk-data network methods follow a different
  workflow
- Gene set exceeds a few thousand genes without prior correlation analysis — the article
  advises first using sclink_cor and possibly dividing genes into modules before network
  inference
edam_operation: http://edamontology.org/operation_0533
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: scLink
  role: R package providing sclink_norm (normalization/filtering), sclink_net (robust
    correlation and sparse Gaussian graphical model network), used to build per-condition
    networks for edge comparison
  repo: https://github.com/Vivianstats/scLink
- name: R
  role: Execution environment for running the scLink pipeline and writing the differential_edges.csv
    table
  repo: https://github.com/Vivianstats/scLink
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.gpb.2020.11.006
    title: 'scLink: Inferring Sparse Gene Co-Expression Networks from Single-Cell
      Expression Data'
  source_content_hash: sha256:d946838001d47b4e8ec21b7fd7c87c90ac56e81b460bc391f949b483ef1f693e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Vivianstats/scLink
  tools:
  - R
  - scLink
  edam_operation: http://edamontology.org/operation_0533
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# differential-network-edge-analysis

## Summary

Identifies condition-specific gene co-expression edges by building separate scLink sparse networks (via sclink_norm and sclink_net with BIC-selected lda) for each condition, then tabulating normal-only vs tumor-only edges with |change in scLink correlation| > 0.5 and bootstrap FDR-adjusted significance.

## When to use

Use when you have single-cell count matrices from two matched conditions (e.g., 211 normal vs 656 tumor immune cells from GSE114727) and want to detect co-expression edges present in one condition but absent in the other.

## When NOT to use

- The analysis goal is directed regulatory relationships — scLink edges are based on statistical dependencies and do not have directions
- You only have bulk RNA-seq data — imputed single-cell-oriented networks may still be much sparser than bulk data, and bulk-data network methods follow a different workflow
- Gene set exceeds a few thousand genes without prior correlation analysis — the article advises first using sclink_cor and possibly dividing genes into modules before network inference

## Inputs

- inDrop single-cell count matrices for matched-normal (211 immune cells) and tumor (656 immune cells) from GSE114727
- Gene expression read count matrix compatible with sclink_norm
- Condition labels separating normal vs tumor cells

## Outputs

- Per-condition scLink network objects with robust correlation matrix (networks$cor) and adjacency/concentration summaries (networks$summary adj and Sigma)
- differential_edges.csv: full table of normal-only and tumor-only edges with delta correlation and FDR-adjusted bootstrap P values

## How to apply

Normalize each condition's count matrix with scLink's sclink_norm (scale.factor = 1e6, log10 scale), filtering to the top 500 genes by average expression of one condition (filter.genes = TRUE, n = 500). Run sclink_net separately on the normal and tumor normalized matrices with lda = seq(0.5, 0.1, -0.05), selecting the regularization parameter by BIC so each network retains at most 5% of possible edges. Extract each network's robust correlation matrix (networks$cor) and adjacency summaries (networks$summary elements adj and Sigma), then call normal-only and tumor-only edges from the two adjacency matrices. Attach the change in scLink correlation between conditions and retain edges with |change in correlation| > 0.5, annotating significance from the bootstrap procedure with FDR adjustment. Validate against known rows such as FNBP1-MGP (correlation ~0.77 in normal vs ~-0.03 in tumor) and EGR1-NUDT3 (~-0.48 in normal, absent in tumor), expecting ~453 normal-only and ~1384 tumor-only differential edges.

## Related tools

- **scLink** (R package providing sclink_norm (normalization/filtering), sclink_net (robust correlation and sparse Gaussian graphical model network), used to build per-condition networks for edge comparison) — https://github.com/Vivianstats/scLink
- **R** (Execution environment for running the scLink pipeline and writing the differential_edges.csv table) — https://github.com/Vivianstats/scLink

## Examples

```
normal <- sclink_net(counts_norm_normal, lda = seq(0.5, 0.1, -0.05), ncores = 1); tumor <- sclink_net(counts_norm_tumor, lda = seq(0.5, 0.1, -0.05), ncores = 1)
```

## Evaluation signals

- differential_edges.csv exists and contains the FNBP1-MGP row with correlation ~0.77 in normal vs ~-0.03 in tumor
- The EGR1-NUDT3 row shows correlation ~-0.48 in normal and is absent in tumor
- Edge counts are consistent with the reported 453 normal-only and 1384 tumor-only edges at |delta correlation| > 0.5
- Each condition's sclink_net run retains at most 5% of possible edges with lda selected by BIC over seq(0.5, 0.1, -0.05)
- Most retained differential edges show FDR-adjusted bootstrap P < 0.05

## Limitations

- Edges identified by scLink are based on statistical dependencies and do not have directions; regulatory, co-regulatory, and functionally coordinated edge types cannot be distinguished from expression data alone
- No software version is specified for the scLink R package in the article
- Imputed data may still be much sparser than bulk data, and imputation could be a time-consuming step depending on cell number

## Evidence

- [methods] normalize the read count matrix with a library size of 10^6: "normalize the read count matrix with a library size of $10^6$ and only keep the 500 genes in `genes` for downstream analysis"
- [methods] sclink_net function role: "use the function `sclink_net` to calculate the robust correlation matrix and identifed sparse co-expression network of scLink"
- [discussion] edges are undirected: "the edges identified by scLink are based on statistical dependencies and do not have directions"
- [discussion] gene filtering advice: "we also suggest a gene filtering step based on the gene detection rates or the mean expression levels"
- [readme] README main functions: [evidence span withheld — anchor status: not_found]
