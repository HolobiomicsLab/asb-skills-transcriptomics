---
name: multiple-testing-correction
description: Use when thousands of genes are tested simultaneously (e.g., normal-vs-cancer
  DESeq2 tests on KIRC, LIHC, PRAD TCGA count matrices) and raw p-values must be converted
  into adjusted p-values before thresholding DE gene sets.
when_to_use_negative:
- The tests are not simultaneous genome/transcriptome-wide tests (e.g., a single pre-specified
  gene set) where FDR correction is overly conservative
- The downstream consumer requires family-wise error rate control (e.g., Bonferroni)
  rather than FDR control, or the effect-size criterion |log2FC| > 2.0 is inappropriate
  for the biology of the comparison
- Input expression data are already normalized and significance-tested (e.g., an existing
  DE results table with padj) — correction would double-adjust the values
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: DESeq2
  role: Performs the normal-vs-cancer differential expression tests whose p-values
    are Benjamini-Hochberg corrected; the corrected values gate gene selection
- name: R/Bioconductor
  role: Runtime environment in which DESeq2 differential expression analysis and the
    multiple-testing correction are executed
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41540-021-00175-9
    title: Computational analysis of fused co-expression networks for the identification
      of candidate cancer gene biomarkers
  source_content_hash: sha256:93e2951f4f4f169cbd6652aff7ef72d2479caf27eacf52b340554ff5ecbb43cf
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DESeq2
  - R/Bioconductor
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# multiple-testing-correction

## Summary

Controls the false discovery rate when selecting thousands of genes (and miRNAs) from differential expression tests, using the Benjamini-Hochberg method so that DE genes are called at adjusted p-value < 0.05 combined with a |log2 fold change| > 2.0 cutoff. In the GeneNetFusion pipeline this corrected selection defines the differential-expression baseline signature per TCGA cancer type.

## When to use

Use when thousands of genes are tested simultaneously (e.g., normal-vs-cancer DESeq2 tests on KIRC, LIHC, PRAD TCGA count matrices) and raw p-values must be converted into adjusted p-values before thresholding DE gene sets.

## When NOT to use

- The tests are not simultaneous genome/transcriptome-wide tests (e.g., a single pre-specified gene set) where FDR correction is overly conservative
- The downstream consumer requires family-wise error rate control (e.g., Bonferroni) rather than FDR control, or the effect-size criterion |log2FC| > 2.0 is inappropriate for the biology of the comparison
- Input expression data are already normalized and significance-tested (e.g., an existing DE results table with padj) — correction would double-adjust the values

## Inputs

- Per-gene raw p-values from DESeq2 differential expression tests on normal-vs-cancer TCGA RNA-Seq count matrices
- Per-gene log2 fold changes (cancer mean over normal mean) for KIRC, LIHC, and PRAD

## Outputs

- Adjusted p-values (Benjamini-Hochberg) per gene, in [0, 1]
- Per-cancer-type DE-gene baseline signature CSV containing only genes with adjusted p-value < 0.05 and |log2FC| > 2.0 (e.g., 1059 genes / 54 miRNAs for KIRC, 434 genes / 27 miRNAs for LIHC, 218 genes / 24 miRNAs for PRAD)

## How to apply

After running DESeq2 on the TCGA RNA-Seq gene count matrices comparing normal versus cancer conditions (using shrinkage estimators for dispersion and fold change), correct the per-gene p-values with the Benjamini-Hochberg method rather than raw p-values, because the test set spans the whole transcriptome. Apply the corrected threshold jointly with an effect-size cutoff: keep only genes with adjusted p-value < 0.05 AND |log2 fold change| > 2.0, where log2FC is the ratio between expression mean values of cancer over normal samples. Write the corrected-and-thresholded DE gene list per cancer type (KIRC, LIHC, PRAD) to a CSV table as the baseline reference signature, which is subsequently reduced by LASSO into the DEL sets.

## Related tools

- **DESeq2** (Performs the normal-vs-cancer differential expression tests whose p-values are Benjamini-Hochberg corrected; the corrected values gate gene selection)
- **R/Bioconductor** (Runtime environment in which DESeq2 differential expression analysis and the multiple-testing correction are executed)

## Examples

```
Rscript -e 'library(DESeq2); dds <- DESeq(dds_obj); res <- results(dds, alpha=0.05); res$padj <- p.adjust(res$pvalue, method="BH"); sig <- subset(res, padj < 0.05 & abs(log2FoldChange) > 2)'
```

## Evaluation signals

- Every value in the adjusted p-value column lies within [0, 1]
- Every gene retained in the output CSV satisfies adjusted p-value < 0.05 AND |log2 fold change| > 2.0 simultaneously
- The per-cancer-type output CSV file exists and is non-empty for KIRC, LIHC, and PRAD
- Resulting signature sizes fall in the reported ranges (KIRC ~1059 genes/54 miRNAs, LIHC ~434 genes/27 miRNAs, PRAD ~218 genes/24 miRNAs)
- The number of retained genes is markedly smaller than the number of tested genes, consistent with joint FDR + log2FC filtering

## Limitations

- The combined padj < 0.05 and |log2FC| > 2.0 threshold is stringent; the resulting baseline sets are the ones LASSO later reduces, so single-threshold variants will change the baseline signature
- The article does not report DESeq2 package versions or additional parameterization details beyond shrinkage estimation for dispersion and fold change
- Baseline gene counts are specific to the TCGA KIRC, LIHC, and PRAD datasets; different cohorts will yield different counts

## Evidence

- [methods] DESeq2 DE-gene selection with BH correction and dual thresholds: [evidence span withheld — anchor status: not_found]
- [methods] DESeq2 as the DE gene identification tool: "We leverage the DESeq2 R/Bioconductor package to identify the differentially expressed (DE) genes"
- [intro] DE analysis as the usual baseline method for biomarker identification: "We also compare our results with the ones from differential gene expression analysis, which is the usual baseline method to identify biomarkers in biology"
