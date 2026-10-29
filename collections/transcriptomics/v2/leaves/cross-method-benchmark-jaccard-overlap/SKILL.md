---
name: cross-method-benchmark-jaccard-overlap
description: Use when multiple GSE or DE methods (e.g., iDEA, fGSEA, GSEA, CAMERA,
  PAGE) are run on identical DE summary statistics and you need quantitative concordance
  metrics such as Jaccard overlap to compare their outputs.
when_to_use_negative:
- When only one method is applied and there is no second method's gene list to compute
  a Jaccard overlap against
- When methods are run on different DE summary statistics or gene universe — the comparison
  requires all methods to consume identical inputs (same beta/z-score statistics and
  same gene sets)
- When the goal is absolute FDR calibration rather than cross-method power/concordance
  comparison (use permutation-based null calibration via iDEA.fit.null and iDEA.FDR
  instead)
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: iDEA
  role: Fits the joint DE+GSE model (iDEA.fit, iDEA.louis, iDEA.fit.null, iDEA.FDR,
    iDEA.BMA) whose gene lists are Jaccard-compared against baselines
  repo: https://github.com/xzhoulab/iDEA
- name: GSEA
  role: Baseline competitive GSE method included in the benchmark; second-best with
    246 gene sets at FDR 5%
- name: CAMERA
  role: Baseline competitive GSE method included in the benchmark (134 gene sets at
    FDR 5%)
- name: PAGE
  role: Baseline competitive GSE method included in the benchmark (205 gene sets at
    FDR 5%)
- name: DAVID
  role: Baseline over-representation method run on the same summary statistics for
    the comparison
- name: zingeR
  role: Source of per-gene DE p-values converted to iDEA summary statistics via zingeR-DESeq2
  repo: https://github.com/statOmics/zinbwaveZinger
- name: MAST
  role: DE method whose results supply one set of summary statistics for the multi-method
    benchmark
- name: SCDE
  role: DE method whose results supply one set of summary statistics for the multi-method
    benchmark
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-020-15298-6
    title: Integrative differential expression and gene set enrichment analysis using
      summary statistics for scRNA-seq studies
  source_content_hash: sha256:8e00bda08c6ddb5e31e233fc3fd22348d613259518c9f17c5ef85c04f6de3284
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/xzhoulab/iDEA
  tools:
  - CAMERA
  - DAVID
  - GSEA
  - MAST
  - PAGE
  - R
  - SCDE
  - iDEA
  - zingeR
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# Cross-method benchmark with Jaccard overlap

## Summary

A benchmarking procedure for comparing gene set enrichment and differential expression methods on the same scRNA-seq summary statistics, quantifying agreement via the Jaccard index between method-specific gene lists and recording per-method counts (e.g., enriched gene sets at FDR 5%, genomic-control lambda_gc) in a single results table.

## When to use

Use when multiple GSE or DE methods (e.g., iDEA, fGSEA, GSEA, CAMERA, PAGE) are run on identical DE summary statistics and you need quantitative concordance metrics such as Jaccard overlap to compare their outputs.

## When NOT to use

- When only one method is applied and there is no second method's gene list to compute a Jaccard overlap against
- When methods are run on different DE summary statistics or gene universe — the comparison requires all methods to consume identical inputs (same beta/z-score statistics and same gene sets)
- When the goal is absolute FDR calibration rather than cross-method power/concordance comparison (use permutation-based null calibration via iDEA.fit.null and iDEA.FDR instead)

## Inputs

- Per-gene DE summary statistics (beta, beta_var / z-scores) from zingeR-DESeq2, MAST, and SCDE on NP1 vs other cell types
- Gene set annotations (mouseGeneSets compiled from MGI GO annotations, GAF 2.0 format)
- Known NP1 DE gene list (top-100) for overlap comparison
- Per-method enrichment p-values/FDR and DE gene lists from iDEA, fGSEA, GSEA, CAMERA, PAGE, DAVID, Fisher's exact test

## Outputs

- usoskin_np1_results.csv with per-method lambda_gc, counts of enriched gene sets at FDR 5%, DE gene counts at FDR 1% anchored on GO:0097458, top-100 overlap, and Jaccard indices
- Jaccard index between iDEA and each baseline DE gene list
- Number of enriched gene sets per method at FDR 5%

## How to apply

First run every method on the same inputs — for the iDEA paper this means converting zingeR-DESeq2, MAST, and SCDE DE results to iDEA summary statistics (z-scores via qnorm(pvalue/2, lower.tail=FALSE), se_beta = abs(beta/zscore)) and calling iDEA.fit, iDEA.louis, iDEA.fit.null, and iDEA.FDR alongside baseline methods DAVID, Fisher's exact test, PAGE, GSEA, and CAMERA. Then compute, per method: the genomic-control lambda_gc, the number of enriched gene sets at an FDR of 5% (iDEA identified 1,268 vs GSEA's 246, fGSEA's 236, CAMERA's 134, and PAGE's 205 on Usoskin NP1 vs other neurons), the number of DE genes at FDR 1% anchored on a specific gene set (e.g., GO:0097458), and the overlap with top-100 known DE genes. Quantify method-to-method concordance with the Jaccard index between the iDEA and baseline DE gene lists, and write all quantities to a single CSV file (usoskin_np1_results.csv) so results are directly comparable. Judge correctness by whether all methods consume identical summary statistics and whether reported counts match the expected relative power ordering.

## Related tools

- **iDEA** (Fits the joint DE+GSE model (iDEA.fit, iDEA.louis, iDEA.fit.null, iDEA.FDR, iDEA.BMA) whose gene lists are Jaccard-compared against baselines) — https://github.com/xzhoulab/iDEA
- **GSEA** (Baseline competitive GSE method included in the benchmark; second-best with 246 gene sets at FDR 5%)
- **CAMERA** (Baseline competitive GSE method included in the benchmark (134 gene sets at FDR 5%))
- **PAGE** (Baseline competitive GSE method included in the benchmark (205 gene sets at FDR 5%))
- **DAVID** (Baseline over-representation method run on the same summary statistics for the comparison)
- **zingeR** (Source of per-gene DE p-values converted to iDEA summary statistics via zingeR-DESeq2) — https://github.com/statOmics/zinbwaveZinger
- **MAST** (DE method whose results supply one set of summary statistics for the multi-method benchmark)
- **SCDE** (DE method whose results supply one set of summary statistics for the multi-method benchmark)

## Examples

```
jaccard <- length(intersect(idea_de_genes, gsea_de_genes)) / length(union(idea_de_genes, gsea_de_genes)); write.csv(res, "usoskin_np1_results.csv")
```

## Evaluation signals

- All methods receive identical inputs: the same beta/beta_var or z-score summary statistics and the same mouseGeneSets object built with min_precent_annot = 0.0025 and max_var_beta = 100
- Reproduced ordering of enriched gene set counts at FDR 5%: iDEA 1,268 > GSEA 246 > fGSEA 236 > PAGE 205 > CAMERA 134
- Jaccard index is computed on identical gene universes and is symmetric and bounded in [0, 1]; it should reflect overlap between iDEA and baseline DE gene lists
- Genomic-control lambda_gc is reported per method; methods without correlation-aware calibration should show lambda deviations from 1
- All per-method quantities are written as rows/columns of usoskin_np1_results.csv with no missing method entries

## Limitations

- iDEA does not explicitly account for gene set overlap that may cause non-independence among gene sets; FDR is calibrated via permuted gene labels (iDEA.fit.null) rather than explicit modeling
- DE analysis on inferred cell types may lead to inflated DE test statistics with artificially smaller standard errors, biasing cross-method comparisons
- The paper notes no software/code availability URL or version numbers for the compared tools in the discussion, so exact versions of benchmarked methods must be pinned separately

## Evidence

- [other] Compute the reported quantities: genomic-control lambda_gc for each method, number of enriched gene sets at FDR 5% ... and the Jaccard index between iDEA and baseline DE gene lists: [evidence span withheld — anchor status: not_found]
- [other] iDEA identified 1,268 enriched gene sets at an FDR of 5%, five times more than the second-best method GSEA (246): [evidence span withheld — anchor status: not_found]
- [discussion] GSE methods that do not explicitly account for gene-gene correlation (e.g. iDEA, PAGE, fGSEA and GSEA) appear to provide more calibrated p-values: "GSE methods that do not explicitly account for gene-gene correlation (e.g. iDEA, PAGE, fGSEA and GSEA) appear to provide more calibrated p-values"
- [readme] devtools::install_github('xzhoulab/iDEA'): "devtools::install_github('xzhoulab/iDEA')"
- [discussion] By estimating FDR based on such permuted null, we can account for test non-independence due to gene set overlaps: "By estimating FDR based on such permuted null, we can account for test non-independence due to gene set overlaps"
