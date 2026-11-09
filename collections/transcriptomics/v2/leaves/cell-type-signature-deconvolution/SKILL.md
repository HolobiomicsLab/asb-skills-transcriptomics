---
name: cell-type-signature-deconvolution
description: Use when you need to infer relative cell type composition from bulk RNA-seq
  by scoring curated cell-type gene signatures (GMT module lists) with sample-level
  PC1 module activity rather than a regression-based deconvolution tool.
when_to_use_negative:
- The expression matrix is raw counts — rRoma requires normalized data, so normalization
  must happen first
- The goal is testing broad pathway activity databases (e.g. MSigDB HALLMARK across
  a disease contrast) rather than estimating cell type abundance from curated cell-type
  signatures
- You need absolute cell fractions from a mixture model — rROMA provides relative
  module activity scores, not a regression-based deconvolution
edam_operation: http://edamontology.org/operation_3565
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: rROMA
  role: 'Core algorithm: computes PC1-based module activity scores for cell-type signature
    gene sets on the normalized bulk matrix'
  repo: https://github.com/sysbio-curie/rROMA
- name: edgeR
  role: cpm normalization of the RNA-seq count matrix prior to rRoma
- name: DESeq2
  role: Alternative vst normalization of the count matrix prior to rRoma
- name: irlba
  role: Efficient truncated SVD used internally by rROMA to compute only the first
    principal component of each signature module
- name: mice
  role: Iterative PCA imputation of missing values in the global expression matrix
    before analysis
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41540-024-00331-x
    title: Representation and quantification of module activity from omics data with
      rROMA
  source_content_hash: sha256:74dee35523634e1f8772a443035bfc5b96d9c51e6a0114166042d4fc95b41ea4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sysbio-curie/rROMA
  tools:
  - DESeq2
  - R
  - edgeR
  - irlba
  - mice
  - rROMA
  edam_operation: http://edamontology.org/operation_3565
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# cell-type-signature-deconvolution

## Summary

Estimate cell type abundances from bulk transcriptomic data by running rROMA on normalized gene-by-sample matrices with cell-type gene signature GMT files (e.g. Plasschaert, Okuda airway epithelial signatures), using PC1 sample scores per signature module as abundance proxies.

## When to use

Use when you need to infer relative cell type composition from bulk RNA-seq by scoring curated cell-type gene signatures (GMT module lists) with sample-level PC1 module activity rather than a regression-based deconvolution tool.

## When NOT to use

- The expression matrix is raw counts — rRoma requires normalized data, so normalization must happen first
- The goal is testing broad pathway activity databases (e.g. MSigDB HALLMARK across a disease contrast) rather than estimating cell type abundance from curated cell-type signatures
- You need absolute cell fractions from a mixture model — rROMA provides relative module activity scores, not a regression-based deconvolution

## Inputs

- Normalized gene-by-sample expression matrix (gene symbols as row names, sample IDs as column names), e.g. edgeR cpm or DESeq2 vst of GSE176121 airway epithelial RNA-seq
- Cell-type signature gene sets in GMT format (e.g. Plasschaert, then Okuda airway epithelial cell-type signatures)
- Sample metadata labels (e.g. culture medium UNC vs SC, CF vs non-CF donor type)

## Outputs

- rRoma.R output object with per-module sample scores, L1 variance, shift/overdispersion statistics and p-values
- Heatmap of sample scores per cell-type module (Plot.Genesets.Samples), saved as PNG, grouped by culture medium
- Comparison of ciliated and secretory module sample scores across medium groups confirming ciliated higher in UNC and secretory higher in SC

## How to apply

Retrieve the bulk RNA-seq matrix (e.g. GSE176121 human airway epithelial cultures from CF and non-CF donors) and normalize it with edgeR cpm or DESeq2 vst, since rRoma requires normalized data. Load the cell-type signature gene sets as a module list with rRoma's ReadGMTFile function. Run rRoma.R on the normalized matrix and module list with recommended settings (UseWeights=TRUE, GeneOutDetection='L1OutExpOut', PCSignMode='UseMeanExpressionAllWeights', PCSignThr=0.9, nSamples >= 100, fixed seed for reproducibility). Visualize sample scores per cell-type module with Plot.Genesets.Samples, passing culture-medium labels (UNC vs SC) as GroupInfo and cluster_cols=TRUE, then compare ciliated and secretory module scores across groups and export heatmaps as PNGs. The rationale: rROMA assesses gene set activity by the maximum one-dimensional variance (PC1) of the signature genes, distinguishing shifted from over-dispersed sets, and its sample scores track cell type abundance in bulk data.

## Related tools

- **rROMA** (Core algorithm: computes PC1-based module activity scores for cell-type signature gene sets on the normalized bulk matrix) — https://github.com/sysbio-curie/rROMA
- **edgeR** (cpm normalization of the RNA-seq count matrix prior to rRoma)
- **DESeq2** (Alternative vst normalization of the count matrix prior to rRoma)
- **irlba** (Efficient truncated SVD used internally by rROMA to compute only the first principal component of each signature module)
- **mice** (Iterative PCA imputation of missing values in the global expression matrix before analysis)

## Examples

```
library(rRoma); rRoma.output <- rRoma.R(MatData, ReadGMTFile("plasschaert_signatures.gmt"), UseWeights=TRUE, GeneOutDetection='L1OutExpOut', PCSignMode='UseMeanExpressionAllWeights', PCSignThr=0.9, nSamples=100)
```

## Evaluation signals

- rRoma.R completes with recommended settings (UseWeights=TRUE, GeneOutDetection='L1OutExpOut', PCSignMode='UseMeanExpressionAllWeights', PCSignThr=0.9, nSamples>=100) and a fixed seed
- Ciliated cell module sample scores are higher in UNC medium samples while secretory cell module scores are higher in SC medium samples, consistent with reported overexpression patterns
- Heatmaps generated with Plot.Genesets.Samples (GroupInfo = medium labels, cluster_cols=TRUE) reproduce the expected sample grouping and export as valid PNGs
- Expression matrix passed to rRoma is normalized (cpm or vst), gene symbols in row names, samples in column names, and signature genes matched by ReadGMTFile

## Limitations

- Requires normalized input data (cpm or vst); missing values are imputed with mice, but the README recommends using datasets whose missing values have already been imputed
- PC1 orientation is ambiguous; sign methods such as PCSignMode='UseMeanExpressionAllWeights' with PCSignThr=0.9 must be set to obtain interpretable score signs
- Authors state the CF case study lacks statistical power and would require additional transcriptional datasets and testing of several gene set reference databases
- Scores are relative module activities and may reflect a cell culture bias, as noted in the rROMA cystic fibrosis analysis

## Evidence

- [readme] rRoma works with **normalized** data. Here cpm from edgeR was used for normalisation: "Warning: rRoma works with **normalized** data. Here cpm from edgeR was used for normalisation"
- [readme] ReadGMTFile for loading signatures: "In case you already have a GMT file with modules you want to test, you can also load them by using the `ReadGMTFile` function."
- [intro] rROMA for cell type abundance estimation: "we illustrate the use of rROMA in estimating cell type abundance through bulk transcriptomic data analysis"
- [intro] PC1 hypothesis for gene set activity: "assess the activity of a gene set by determining the maximum amount of one-dimensional variance, which is represented by the first principal component (PC1)"
- [discussion] irlba efficiency for PC1: "rROMA uses the R irlba package36,37, which allows to focus on the first principal component"
- [methods] GSE176121 dataset description: "This is a RNA sequencing dataset of human airway epithelial cultures from cystic fibrosis (CF) and non-CF donors from GSE176121, PMID: 34166230."
- [readme] Installing rROMA from GitHub: "devtools::install_github("sysbio-curie/rRoma")"
