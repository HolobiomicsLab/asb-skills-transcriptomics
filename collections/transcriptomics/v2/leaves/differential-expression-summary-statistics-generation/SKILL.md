---
name: differential-expression-summary-statistics-generation
description: Use when you have a two-group scRNA-seq comparison (e.g. DEC vs EC, CD4+
  vs CD8+ T cells) and need per-gene beta/se summary statistics as input to iDEA or
  other GSE methods that consume DE summary statistics.
when_to_use_negative:
- Input is already a per-gene summary-statistics table with beta and beta_var columns
- The goal is a full individual-level DE analysis without downstream gene set enrichment,
  where tools like MAST or SCDE can be run directly
- Comparing more than two groups without a defined two-group contrast
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: zingeR
  role: zingeR-DESeq2 computes the per-gene beta coefficients and coefficient variances
    from scRNA-seq counts that iDEA consumes
  repo: https://github.com/statOmics/zinbwaveZinger
- name: MAST
  role: Alternative DE method for generating gene-level summary statistics on scRNA-seq
    data
- name: SCDE
  role: Alternative DE method for generating per-gene summary statistics
- name: iDEA
  role: Downstream consumer of the summary statistics; CreateiDEAObject pairs beta/se
    with gene set annotations (max_var_beta = 100, min_precent_annot = 0.0025)
  repo: xzhoulab/iDEA
provenance:
  source_task_ids:
  - task_005
  - task_007
  source_papers:
  - doi: 10.1038/s41467-020-15298-6
    title: Integrative differential expression and gene set enrichment analysis using
      summary statistics for scRNA-seq studies
  source_content_hash: sha256:92a3f9d0f59b54e4a47d80d3fe6524d9bd6471c9bab9ffea114486f18c09847c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/statOmics/zinbwaveZinger
  tools:
  - CAMERA
  - GSEA
  - MAST
  - PAGE
  - SCDE
  - fGSEA
  - iDEA
  - zingeR
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# differential-expression-summary-statistics-generation

## Summary

Generate per-gene differential expression summary statistics (beta coefficients and coefficient variances, or z-scores converted from p-values) from scRNA-seq count data using methods such as zingeR-DESeq2 or MAST, formatted as the input required by iDEA's CreateiDEAObject.

## When to use

Use when you have a two-group scRNA-seq comparison (e.g. DEC vs EC, CD4+ vs CD8+ T cells) and need per-gene beta/se summary statistics as input to iDEA or other GSE methods that consume DE summary statistics.

## When NOT to use

- Input is already a per-gene summary-statistics table with beta and beta_var columns
- The goal is a full individual-level DE analysis without downstream gene set enrichment, where tools like MAST or SCDE can be run directly
- Comparing more than two groups without a defined two-group contrast

## Inputs

- Raw scRNA-seq count matrix (e.g. pbmc3k: 2,638 cells x 13,713 genes; or Chu et al. human ESC DEC vs EC data)
- Two-group cell-type or condition labels (e.g. CD4+ vs CD8+ T cells)
- Optional: pre-computed DE p-values and LogFC from an existing DE tool (to convert to z-scores)

## Outputs

- Per-gene beta coefficient vector with gene names as row names
- Per-gene beta variance / standard error vector (se = |beta/zscore| if converted from p-values)
- Summary-statistics data.frame ready for CreateiDEAObject

## How to apply

Run an existing scRNA-seq DE method such as zingeR-DESeq2 or MAST on the count matrix to obtain a data.frame of per-gene beta coefficients and coefficient variances with gene names as row names. If only p-values and LogFC are available, convert the p-value to a z-score via qnorm and set se = |beta/zscore| and beta_var = se^2. The beta variance is used downstream as a filtering quantity: genes with variance smaller than max_var_beta (default 100) are maintained, and min_degene sets the threshold for the number of detected DE genes in the summary statistics. Using summary statistics instead of individual-level data makes downstream GSE analysis less susceptible to gene-gene correlations and dropout events, and keeps computation scalable.

## Related tools

- **zingeR** (zingeR-DESeq2 computes the per-gene beta coefficients and coefficient variances from scRNA-seq counts that iDEA consumes) — https://github.com/statOmics/zinbwaveZinger
- **MAST** (Alternative DE method for generating gene-level summary statistics on scRNA-seq data)
- **SCDE** (Alternative DE method for generating per-gene summary statistics)
- **iDEA** (Downstream consumer of the summary statistics; CreateiDEAObject pairs beta/se with gene set annotations (max_var_beta = 100, min_precent_annot = 0.0025)) — xzhoulab/iDEA

## Examples

```
devtools::install_github('xzhoulab/iDEA'); library(iDEA); idea <- CreateiDEAObject(beta, se, geneid, chr, geneid_annot, annotation)
```

## Evaluation signals

- Summary-statistics table is a data.frame with gene names as row names and finite beta and beta_var for tested genes
- Genes with coefficient variance >= max_var_beta (default 100) are filtered before building the iDEA object
- The number of detected DE genes at FDR 1% falls within reasonable tolerance of published values (e.g. 255 vs 221 on pbmc3k CD4+/CD8+; 2,753 vs 1,673 on DEC vs EC)
- Genomic control factor lambda_gc matches reported values (e.g. 1.13, 1.02, 0.96, 1.01, 0.29) within numerical tolerance
- min_precent_annot filter (default 0.0025) yields gene sets whose coverage rate exceeds the threshold

## Limitations

- z-score modeling effectively assumes the prior distribution of true effect sizes is dependent on the standard errors, and subsequently the sample size
- DE analysis on inferred cell types may lead to inflated DE test statistics with artificially smaller standard errors
- iDEA makes no explicit modeling assumptions on the individual-level scRNA-seq data, so quality of the summary statistics depends entirely on the upstream DE method chosen

## Evidence

- [intro] iDEA makes use of summary statistics output from existing DE tools and does not make explicit modeling assumptions on the individual-level scRNA-seq data: "iDEA makes use of summary statistics output from existing DE tools and does not make explicit modeling assumptions on the individual-level scRNA-seq data"
- [methods] max_var_beta filter: "max_var_beta: the cutoff of the variance of the coefficient of genes. Genes with variance smaller than 'max_var_beta' are maintained. Default is 100"
- [intro] existing DE methods: "Commonly applied DE methods in scRNA-seq include MAST3, SCDE4, and zingeR5, to name a few."
- [readme] zingeR-DESeq2 preparation: "prepared by [zingeR-DESeq2](https://github.com/statOmics/zinbwaveZinger) method."
