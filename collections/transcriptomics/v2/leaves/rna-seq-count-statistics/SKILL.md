---
name: rna-seq-count-statistics
description: Use when you have raw TCGA RNA-Seq gene count matrices split into normal
  vs cancer samples (e.g. KIRC, LIHC, PRAD) and need a DE-gene baseline signature
  to benchmark a novel biomarker-extraction method.
when_to_use_negative:
- Input is already a filtered gene signature or community list (e.g. IC genes or network-extracted
  genes) rather than raw count matrices — DE analysis would not add a baseline here
- Goal is the paper's co-expression network fusion pipeline (disparity filter, SNF
  fusion, Gephi modularity communities) rather than a DE-based baseline signature
- Data are already normalized expression values (e.g. TPM) rather than gene counts,
  since DESeq2 requires count data for its dispersion and shrinkage estimators
edam_operation: http://edamontology.org/operation_0314
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: DESeq2
  role: Identifies the differentially expressed (DE) genes from TCGA count data, with
    shrinkage estimators for dispersion and fold change, producing the baseline signature
- name: R/Bioconductor
  role: Platform in which the DESeq2 differential expression analysis of normal vs
    cancer conditions is executed
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41540-021-00175-9
    title: Computational analysis of fused co-expression networks for the identification
      of candidate cancer gene biomarkers
  source_content_hash: sha256:cdf53d60b9111c1cdfaaae7b917a88362af3e204af7c32970e4376b14b581b4c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DESeq2
  - R/Bioconductor
  edam_operation: http://edamontology.org/operation_0314
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# rna-seq-count-statistics

## Summary

Establishes the differentially expressed (DE) gene baseline signature from TCGA RNA-Seq count matrices using the DESeq2 R/Bioconductor package, applying Benjamini-Hochberg correction and |log2FC| > 2.0 cutoffs. The resulting DE gene sets serve as the typically-used baseline against which the paper's IC gene sets are benchmarked via normal/cancer classification.

## When to use

Use when you have raw TCGA RNA-Seq gene count matrices split into normal vs cancer samples (e.g. KIRC, LIHC, PRAD) and need a DE-gene baseline signature to benchmark a novel biomarker-extraction method.

## When NOT to use

- Input is already a filtered gene signature or community list (e.g. IC genes or network-extracted genes) rather than raw count matrices — DE analysis would not add a baseline here
- Goal is the paper's co-expression network fusion pipeline (disparity filter, SNF fusion, Gephi modularity communities) rather than a DE-based baseline signature
- Data are already normalized expression values (e.g. TPM) rather than gene counts, since DESeq2 requires count data for its dispersion and shrinkage estimators

## Inputs

- Preprocessed TCGA RNA-Seq gene count matrices (raw counts) for normal and cancer samples, per cancer type (KIRC, LIHC, PRAD)
- Sample condition metadata mapping samples to normal vs cancer groups
- zFPKM-filtered expression data (genes with mean zFPKM > −3.0 retained during preprocessing)

## Outputs

- Per-cancer-type CSV table of DE genes (adjusted p < 0.05, |log2FC| > 2.0) forming the baseline reference signature
- Baseline DE gene sets: 1059 genes (54 miRNAs) for KIRC, 434 genes (27 miRNAs) for LIHC, 218 genes (24 miRNAs) for PRAD
- LASSO-reduced DEL sets derived from the DE gene sets for classification comparison

## How to apply

Load the preprocessed TCGA RNA-Seq gene count matrices for normal and cancer samples of each cancer type (KIRC, LIHC, PRAD), where preprocessed RNA-Seq data have a mean zFPKM higher than −3.0. Run the DESeq2 R/Bioconductor package on the count data comparing normal versus cancer conditions, using shrinkage estimators for dispersion and fold change. Compute the log2 fold change as the ratio between the expression mean values of cancer over normal samples, then correct for multiple testing with the Benjamini-Hochberg method. Select DE genes with adjusted p-value < 0.05 and |log2 fold change| > 2.0, and write one DE-gene baseline reference signature per cancer type to a CSV table. These DE gene sets (optionally reduced via LASSO into DEL sets) are then used as the usual baseline method for biomarker identification, compared against the extracted IC gene sets through normal/cancer classification performance.

## Related tools

- **DESeq2** (Identifies the differentially expressed (DE) genes from TCGA count data, with shrinkage estimators for dispersion and fold change, producing the baseline signature)
- **R/Bioconductor** (Platform in which the DESeq2 differential expression analysis of normal vs cancer conditions is executed)

## Examples

```
dds <- DESeqDataSetFromMatrix(counts, coldata, ~condition); dds <- DESeq(dds); res <- lfcShrink(dds, contrast=c("condition","cancer","normal"))
```

## Evaluation signals

- The output CSV table exists for each cancer type (KIRC, LIHC, PRAD)
- Every gene in each cancer-type DE list passes adjusted p-value < 0.05 and |log2FC| > 2.0 under Benjamini-Hochberg correction
- Adjusted p-values in each output table fall within 0–1
- Gene and miRNA counts match the reported baseline sizes: 1059 genes (54 miRNAs) for KIRC, 434 genes (27 miRNAs) for LIHC, 218 genes (24 miRNAs) for PRAD
- The DE sets feed LASSO-reduced DEL sets whose normal/cancer classification performance can be compared against the IC gene sets

## Limitations

- The strict cutoffs (adjusted p < 0.05 and |log2FC| > 2.0) restrict the baseline signature, e.g. only 218 genes for PRAD, which may exclude moderate-effect genes
- The DE baseline underperforms on survival relevance: only 6 differentially expressed genes were significantly associated with survival versus 44 IC genes
- Preprocessing restricts inputs to RNA-Seq data with mean zFPKM > −3.0, so lowly expressed genes are excluded before DE testing

## Evidence

- [methods] We leverage the DESeq2 R/Bioconductor package to identify the differentially expressed (DE) genes: "We leverage the DESeq2 R/Bioconductor package to identify the differentially expressed (DE) genes"
- [methods] zFPKM preprocessing threshold: "The selected RNA-Seq data have a mean value of normalized fragments per kilobase of transcript per million mapped reads (zFPKM) higher than −3.0"
- [intro] DE analysis as usual baseline: "We also compare our results with the ones from differential gene expression analysis, which is the usual baseline method to identify biomarkers in biology"
- [abstract] Classification comparison against DE genes: "comparing the performance of normal/cancer classification for our identified gene sets and other gene signatures, including the typically-used differentially expressed genes"
- [readme] Repository scope (LIHC data and scripts): "This repository contains all data, scripts and results related to the LIHC tumor analysis."
