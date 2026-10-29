---
name: cell-type-abundance-comparison-across-culture-media
description: Use when bulk RNA-seq samples were grown under different differentiation
  media (e.g. UNC vs SC air-liquid interface cultures) and you need to verify or estimate
  cell type composition shifts, such as ciliated vs secretory cell abundance, from
  gene signatures rather than deconvolution tools.
when_to_use_negative:
- Input data is single-cell rather than bulk transcriptomics — rROMA cell-type abundance
  estimation here targets bulk expression matrices
- The expression matrix is raw counts — rRoma works with normalized data (cpm or vst),
  so normalization must happen first
- The goal is pathway activity (e.g. MSigDB HALLMARK sets) rather than cell-type composition
  estimation from signature gene sets
edam_operation: http://edamontology.org/operation_0571
edam_topics:
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: rROMA
  role: Core R package computing PC1-based module activity scores for cell-type signature
    gene sets on the normalized bulk matrix
  repo: https://github.com/sysbio-curie/rROMA
- name: edgeR
  role: Provides cpm normalization of raw counts into the gene-by-sample matrix rRoma
    requires
- name: DESeq2
  role: Alternative normalizer via vst transformation of counts before rRoma
- name: irlba
  role: Efficient truncated SVD used by rROMA to compute the first principal component
    of each gene set
- name: mice
  role: Imputes missing values in the expression matrix via iterative PCA when NAs
    are present (FillNAMethod)
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41540-024-00331-x
    title: Representation and quantification of module activity from omics data with
      rROMA
  source_content_hash: sha256:a0ca24fded71872e1ecd93eb765685eef8d03374c5f750389b5dcbe43f275b2e
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
  edam_operation: http://edamontology.org/operation_0571
  edam_topics:
  - http://edamontology.org/topic_3518
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# cell-type-abundance-comparison-across-culture-media

## Summary

Estimate cell-type abundances in bulk RNA-seq data by running rROMA with cell-type signature gene sets (e.g. Plasschaert and Okuda airway epithelium signatures) and comparing module sample scores between culture-medium groups (UNC vs SC) to confirm expected ciliated/secretory abundance patterns.

## When to use

Use when bulk RNA-seq samples were grown under different differentiation media (e.g. UNC vs SC air-liquid interface cultures) and you need to verify or estimate cell type composition shifts, such as ciliated vs secretory cell abundance, from gene signatures rather than deconvolution tools.

## When NOT to use

- Input data is single-cell rather than bulk transcriptomics — rROMA cell-type abundance estimation here targets bulk expression matrices
- The expression matrix is raw counts — rRoma works with normalized data (cpm or vst), so normalization must happen first
- The goal is pathway activity (e.g. MSigDB HALLMARK sets) rather than cell-type composition estimation from signature gene sets

## Inputs

- Bulk RNA-seq count matrix for airway epithelial cultures (e.g. GSE176121), CF and non-CF donors
- edgeR cpm- or DESeq2 vst-normalized gene-by-sample expression matrix (rRoma requires normalized data)
- Cell-type signature GMT file(s): Plasschaert signatures, then Okuda signatures
- Sample labels with culture-medium grouping (UNC vs SC) and CF/non-CF type

## Outputs

- rRoma.R output object with sample scores per cell-type module
- Heatmap of cell-type module sample scores grouped by culture medium (PNG), one for Plasschaert and one for Okuda signatures
- Comparison of ciliated vs secretory module scores across UNC/SC groups confirming expected abundance patterns

## How to apply

Retrieve the bulk RNA-seq dataset (e.g. GSE176121 airway epithelial cultures from CF and non-CF donors) and normalize it so rRoma receives a gene-by-sample matrix of normalized values — cpm from edgeR or vst from DESeq2 both work. Load cell-type signature gene sets as a module list via ReadGMTFile from a GMT file, first with the Plasschaert signatures then with the Okuda signatures. Run rRoma.R on the normalized matrix and module list with UseWeights=TRUE, GeneOutDetection='L1OutExpOut', PCSignMode='UseMeanExpressionAllWeights', PCSignThr=0.9, at least 100 random samples for the null distribution, and a fixed seed for reproducibility. Generate a heatmap of sample scores per cell-type module with Plot.Genesets.Samples, passing culture-medium sample labels as GroupInfo and cluster_cols=TRUE. Finally, compare ciliated and secretory module scores across medium groups: correct results show ciliated scores higher in UNC medium and secretory scores higher in SC medium, consistent with the reported overexpression patterns; save both heatmaps as PNG files.

## Related tools

- **rROMA** (Core R package computing PC1-based module activity scores for cell-type signature gene sets on the normalized bulk matrix) — https://github.com/sysbio-curie/rROMA
- **edgeR** (Provides cpm normalization of raw counts into the gene-by-sample matrix rRoma requires)
- **DESeq2** (Alternative normalizer via vst transformation of counts before rRoma)
- **irlba** (Efficient truncated SVD used by rROMA to compute the first principal component of each gene set)
- **mice** (Imputes missing values in the expression matrix via iterative PCA when NAs are present (FillNAMethod))

## Examples

```
rRoma.output <- rRoma.R(MatData, AllHall)
```

## Evaluation signals

- Ciliated cell module sample scores are higher in UNC medium samples than in SC medium samples
- Secretory cell module sample scores are higher in SC medium samples than in UNC medium samples
- The same medium-dependent abundance pattern reproduces with both the Plasschaert and the Okuda signature module lists
- rRoma.R ran with a fixed seed and nSamples >= 100, so re-running the script yields reproducible module scores
- Input matrix passed to rRoma.R contains normalized values (cpm or vst), not raw counts

## Limitations

- The article's CF case study lacks statistical power; authors state it would require additional transcriptional datasets and testing of several gene set reference databases
- Missing values trigger mice-based imputation, but the README recommends using datasets for which missing values have already been imputed
- Default rROMA hyperparameter values (outlier z-score thresholds, extreme-weight percentage, ApproxSample grid) are not reported in the article, so tuning may be needed
- Sign of PC1 is ambiguous; rROMA must orient it via methods such as PCSignMode='UseMeanExpressionAllWeights' (PCSignThr=0.9), otherwise module scores may be flipped

## Evidence

- [readme] rRoma works with **normalized** data. Here cpm from edgeR was used for normalisation, but you could also use vst from DeSeq2 or any other method you deem interesting.: "Warning: rRoma works with **normalized** data. Here cpm from edgeR was used for normalisation, but you could also use vst from DeSeq2"
- [intro] we illustrate the use of rROMA in estimating cell type abundance through bulk transcriptomic data analysis: "we illustrate the use of rROMA in estimating cell type abundance through bulk transcriptomic data analysis"
- [readme] In case you already have a GMT file with modules you want to test, you can also load them by using the `ReadGMTFile` function.: "you can also load them by using the `ReadGMTFile` function"
- [discussion] rROMA uses the R irlba package, which allows to focus on the first principal component: "rROMA uses the R irlba package36,37, which allows to focus on the first principal component"
- [discussion] it would require us to include additional transcriptional dataset to take more samples into account and increase the statistical power of our analyses: "it would require us to include additional transcriptional dataset to take more samples into account and increase the statistical power of our analyses"
