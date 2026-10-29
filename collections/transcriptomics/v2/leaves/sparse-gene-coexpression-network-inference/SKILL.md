---
name: sparse-gene-coexpression-network-inference
description: Use when analyzing sparse single-cell RNA-seq count matrices where Pearson/Spearman
  correlations are unreliable due to excessive zeros, and you need a sparse gene co-expression
  network or comparison of networks across conditions/time points.
when_to_use_negative:
- When the goal is to determine edge direction or distinguish regulatory, co-regulatory,
  and functionally coordinated edge types — scLink edges are undirected statistical
  dependencies and expression data alone cannot resolve these.
- 'When imputation-first strategies are mandatory for the analysis design: imputed
  data may still be much sparser than bulk data, and imputation can be time-consuming
  depending on cell number.'
- When analyzing very large gene sets without pre-filtering — gene filtering by detection
  rate or mean expression, or module-based subdivision, is recommended before applying
  scLink.
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: scLink
  role: Core R package providing sclink_norm for normalization/filtering, sclink_cor
    for correlation structure exploration, and sclink_net for penalized Gaussian graphical
    model network inference
  repo: https://github.com/Vivianstats/scLink
- name: R
  role: Runtime environment for installing and executing the scLink package and downstream
    differential-edge and degree computations
- name: scImpute
  role: Prior work by the same authors addressing excess zeros; the first step of
    scLink is partially motivated by scImpute
provenance:
  source_task_ids:
  - task_006
  - task_007
  source_papers:
  - doi: 10.1016/j.gpb.2020.11.006
    title: 'scLink: Inferring Sparse Gene Co-Expression Networks from Single-Cell
      Expression Data'
  source_content_hash: sha256:e4291d16bcac4cb51add3f3e13a4cad33450ad74619fb5c9b6f987717f6f7c62
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Vivianstats/scLink
  tools:
  - R
  - scImpute
  - scLink
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# sparse-gene-coexpression-network-inference

## Summary

Infer sparse, undirected gene co-expression networks from sparse single-cell RNA-seq count data using scLink, which combines a robust correlation estimator with a penalized Gaussian graphical model and BIC-based regularization selection. It enables differential network comparison across conditions and time-course co-expression change detection.

## When to use

Use when analyzing sparse single-cell RNA-seq count matrices where Pearson/Spearman correlations are unreliable due to excessive zeros, and you need a sparse gene co-expression network or comparison of networks across conditions/time points.

## When NOT to use

- When the goal is to determine edge direction or distinguish regulatory, co-regulatory, and functionally coordinated edge types — scLink edges are undirected statistical dependencies and expression data alone cannot resolve these.
- When imputation-first strategies are mandatory for the analysis design: imputed data may still be much sparser than bulk data, and imputation can be time-consuming depending on cell number.
- When analyzing very large gene sets without pre-filtering — gene filtering by detection rate or mean expression, or module-based subdivision, is recommended before applying scLink.

## Inputs

- Raw single-cell RNA-seq read/gene count matrix (cells x genes), e.g. inDrop counts from GEO accessions GSE114727 or GSE75748
- Optional: list of genes of interest (e.g. 51 lineage marker genes) for filter.genes=FALSE subsetting

## Outputs

- Per-condition/per-time-point scLink network object containing the robust correlation matrix (networks$cor) and adjacency/concentration summaries (networks$summary adj, Sigma)
- Differential-edge table (e.g. differential_edges.csv) with normal-only and tumor-only edges, delta correlation, and FDR-adjusted bootstrap P values
- Marker-pair correlation-change table (e.g. correlation_changes.csv) and per-gene degree-change table (e.g. degree_changes.csv) across time points

## How to apply

First normalize raw counts with scLink's sclink_norm (library size 10^6, log10 scale), either filtering to the top n genes by average expression (filter.genes=TRUE, e.g. n=500 as in GSE114727) or subsetting to a curated gene list with filter.genes=FALSE (e.g. 51 lineage markers in GSE75748). Run sclink_net on each normalized matrix over the candidate regularization sequence lda = seq(0.5, 0.1, -0.05), selecting the lambda by BIC so each network keeps at most ~5% of possible edges; extract the robust correlation matrix (networks$cor) and adjacency/concentration summaries (networks$summary elements adj and Sigma). For large gene lists, first use sclink_cor to investigate correlation structures or divide genes into major modules and apply scLink per module. Compare networks across conditions or time points by computing pairwise correlation changes among gene pairs and retaining edges with |change in correlation| > 0.5, and annotate significance via bootstrap with FDR adjustment. Edges are undirected statistical dependencies, so they cannot be labeled as regulatory versus co-regulatory from expression alone.

## Related tools

- **scLink** (Core R package providing sclink_norm for normalization/filtering, sclink_cor for correlation structure exploration, and sclink_net for penalized Gaussian graphical model network inference) — https://github.com/Vivianstats/scLink
- **R** (Runtime environment for installing and executing the scLink package and downstream differential-edge and degree computations)
- **scImpute** (Prior work by the same authors addressing excess zeros; the first step of scLink is partially motivated by scImpute)

## Examples

```
install.packages("scLink"); expr_norm <- sclink_norm(counts, filter.genes = TRUE, n = 500); networks <- sclink_net(expr_norm, lda = seq(0.5, 0.1, -0.05))
```

## Evaluation signals

- The normalization step used library size 10^6 on the log10 scale via sclink_norm, and only the target gene set (e.g. top 500 genes or 51 markers) is retained downstream.
- Each sclink_net fit selects lambda by BIC from lda = seq(0.5, 0.1, -0.05) and keeps at most ~5% of possible edges (sparse network).
- Differential-edge table reproduces reported counts at |delta correlation| > 0.5: ~453 normal-only and ~1384 tumor-only edges in GSE114727, and 240 of 1275 marker pairs changed > 0.5 across the GSE75748 DE differentiation time course.
- Checkable rows validate with reported values, e.g. FNBP1-MGP (cor ~0.77 in normal vs ~-0.03 in tumor), EGR1-NUDT3 (~-0.48 in normal, absent in tumor), NANOG-PECAM1, MT1X-SOX17, and hub-gene degree changes for LHX1, HAPLN1, GNG11.
- Output CSV files (e.g. differential_edges.csv, correlation_changes.csv, degree_changes.csv) exist, are parseable, and contain the expected validation rows.

## Limitations

- Edges identified by scLink are based on statistical dependencies and do not have directions; it is not feasible to directly distinguish regulatory, co-regulatory, and functionally coordinated edge types using only gene expression data.
- Imputed data may still be much sparser than bulk data, so bulk-data network methods may perform poorly on imputed single-cell data; imputation itself can be time-consuming depending on cell number.
- For large gene lists, scLink should be applied after gene filtering (by detection rate or mean expression) or after dividing genes into major modules based on scLink's correlation, rather than run on all genes at once.
- No software version is specified for the scLink R package in the article, which may complicate exact reproducibility.

## Evidence

- [intro] scLink adapts the Gaussian graphical model to distinguish direct associations between genes from indirect ones and leads to sparse networks: "scLink adapts the Gaussian graphical model [29] to distinguish direct associations between genes from indirect ones and leads to sparse networks"
- [methods] normalize the read count matrix with a library size of 10^6 and only keep the 500 genes: "normalize the read count matrix with a library size of $10^6$ and only keep the 500 genes in `genes` for downstream analysis"
- [methods] use the function sclink_net to calculate the robust correlation matrix and identified sparse co-expression network: "use the function `sclink_net` to calculate the robust correlation matrix and identifed sparse co-expression network of scLink"
- [methods] they can set filter.genes=TRUE to filter for the top n genes with largest average expression values: "they can set `filter.genes=TRUE` to filter for the top $n$ genes with largest average expression values"
- [discussion] the edges identified by scLink are based on statistical dependencies and do not have directions: "the edges identified by scLink are based on statistical dependencies and do not have directions"
- [readme] The package is available on CRAN: "The package is available on CRAN. For installation please use the
following codes in `R`

 r
install.packages("scLink")
"
