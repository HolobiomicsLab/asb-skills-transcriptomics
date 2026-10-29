---
name: single-cell-rna-seq-gene-detection-rate-filtering
description: Use when a raw single-cell read count matrix is highly zero-inflated
  (e.g. ~70% zeros) and must be filtered to well-detected genes prior to normalization,
  correlation estimation, or network inference with scLink.
when_to_use_negative:
- Input is already a normalized or imputed expression matrix rather than a raw count
  matrix (filtering should precede sclink_norm)
- Bulk RNA-seq or other non-single-cell data where zero inflation from non-detection
  is not the dominant feature
- A gene explicitly of biological interest falls below the detection-rate cutoff and
  should be force-included via a user-set threshold instead of dropped
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: scLink
  role: R package whose pipeline begins with gene filtering; sclink_norm (scale.factor=1e6)
    normalizes the filtered counts and sclink_cor/sclink_net consume them
  repo: https://github.com/Vivianstats/scLink
- name: R
  role: Environment in which the filtering and scLink functions are executed
  repo: https://github.com/Vivianstats/scLink
- name: scImpute
  role: Prior method whose treatment of excess zeros partially motivated scLink's
    first step of filtering low-detection genes
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.gpb.2020.11.006
    title: 'scLink: Inferring Sparse Gene Co-Expression Networks from Single-Cell
      Expression Data'
  source_content_hash: sha256:43d5cb4a5feedb45b962d6b8c2850adfb368de7b5665a6f75b41dd5da1851d3d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Vivianstats/scLink
  tools:
  - R
  - scImpute
  - scLink
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# single-cell-rna-seq-gene-detection-rate-filtering

## Summary

Pre-processes sparse single-cell RNA-seq count matrices by keeping only genes detected in a minimum fraction of cells (e.g. >=10%), reducing all-zero-heavy features before scLink's Gamma-Normal mixture modeling and robust co-expression estimation.

## When to use

Use when a raw single-cell read count matrix is highly zero-inflated (e.g. ~70% zeros) and must be filtered to well-detected genes prior to normalization, correlation estimation, or network inference with scLink.

## When NOT to use

- Input is already a normalized or imputed expression matrix rather than a raw count matrix (filtering should precede sclink_norm)
- Bulk RNA-seq or other non-single-cell data where zero inflation from non-detection is not the dominant feature
- A gene explicitly of biological interest falls below the detection-rate cutoff and should be force-included via a user-set threshold instead of dropped

## Inputs

- Raw single-cell RNA-seq read count matrix (genes x cells), e.g. GSE114727 immune-cell counts
- Optional detection-rate threshold (e.g. >=10% of cells) or mean-expression threshold
- Optional vector of genes of particular interest to force-include

## Outputs

- Filtered count matrix restricted to genes passing the detection-rate/mean-expression criterion (e.g. ~410 genes from ~thousands)
- Normalized expression matrix from sclink_norm (library size 10^6, log10 scale) ready for sclink_cor/sclink_net

## How to apply

Start from the raw single-cell count matrix (e.g. GSE114727 B cells, 109 cells) and compute each gene's detection rate across cells; retain genes detected in at least 10% of cells (yielding ~410 genes with ~71.2% zeros in the example). Alternatively, filter on mean expression levels, or set a threshold to include genes of particular interest. In the scLink R package this can be delegated via filter.genes=TRUE, which keeps the top n genes with largest average expression values. After filtering, normalize the retained genes with sclink_norm (scale.factor = 1e6, log10 scale) before downstream correlation or network inference. Rationale: scLink's first step is partially motivated by scImpute, and gene filtering removes genes too sparsely measured for reliable Gamma-Normal mixture fitting and robust Pearson correlation.

## Related tools

- **scLink** (R package whose pipeline begins with gene filtering; sclink_norm (scale.factor=1e6) normalizes the filtered counts and sclink_cor/sclink_net consume them) — https://github.com/Vivianstats/scLink
- **R** (Environment in which the filtering and scLink functions are executed) — https://github.com/Vivianstats/scLink
- **scImpute** (Prior method whose treatment of excess zeros partially motivated scLink's first step of filtering low-detection genes)

## Examples

```
counts.filtered <- counts[Matrix::rowSums(counts > 0) >= 0.10 * ncol(counts), ]; expr <- sclink_norm(counts.filtered, scale.factor = 1e6)
```

## Evaluation signals

- Filtered matrix retains only genes detected in >=10% of cells; verify per-gene detection counts meet the cutoff
- Number of retained genes is in the expected range for the dataset (e.g. ~410 genes from GSE114727 B cells with ~71.2% remaining zeros)
- sclink_norm output is on a library-size 10^6, log10 scale and restricted to the filtered gene set
- Downstream sclink_cor fits Gamma-Normal mixtures without errors on the retained genes, indicating sufficient non-zero support per gene
- If filter.genes=TRUE is used, confirm the retained set equals the top n genes by average expression

## Limitations

- Filtering thresholds (detection rate, mean expression, or top-n) are user choices; an arbitrary cutoff can exclude biologically relevant low-detection genes unless a gene-interest threshold is set
- Imputed/filtered data may still remain much sparser than bulk data, so bulk-data network methods may perform poorly downstream
- No software version for the scLink R package is specified, complicating exact reproduction of filtering behavior

## Evidence

- [discussion] the first step of scLink is partially motivated by our scImpute method: "the first step of scLink is partially motivated by our scImpute method [28]"
- [discussion] gene filtering step based on detection rates: "we also suggest a gene filtering step based on the gene detection rates or the mean expression levels [15]"
- [discussion] set threshold for genes of interest: "researchers can also set a threshold such that genes of particular interest will be included"
- [methods] filter.genes=TRUE keeps top-n average expression: "they can set `filter.genes=TRUE` to filter for the top $n$ genes with largest average expression values"
- [methods] normalize with library size 10^6: "normalize the read count matrix with a library size of $10^6$ and only keep the 500 genes in `genes` for downstream analysis"
- [readme] sclink_norm pre-processing function: [evidence span withheld — anchor status: external_doc]
