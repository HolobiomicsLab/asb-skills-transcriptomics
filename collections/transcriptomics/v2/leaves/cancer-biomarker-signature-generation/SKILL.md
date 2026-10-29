---
name: cancer-biomarker-signature-generation
description: Use when raw TCGA-style RNA-Seq gene count matrices for paired normal
  vs cancer samples (e.g., KIRC, LIHC, PRAD) must be turned into a differential-expression
  baseline signature to benchmark other gene sets by classification.
when_to_use_negative:
- 'The goal is the paper''s proposed IC gene sets: those are extracted from fused
  co-expression networks via the disparity filter and the Gephi modularity method,
  not from DESeq2 - differential expression is only the comparison baseline here'
- Input is already-normalized or filtered expression (e.g., zFPKM-normalized values)
  rather than raw integer gene counts, since DESeq2's dispersion model requires count
  data
- 'The task is finding survival-associated biomarkers: the article reports only 6
  DE genes significantly associated with survival versus 44 IC genes, making the DE
  baseline the weaker source for that goal'
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: DESeq2
  role: Runs the normal-versus-cancer differential expression test on the TCGA gene
    count matrices to produce the baseline DE signature
- name: R/Bioconductor
  role: Runtime environment in which the DESeq2 package is executed for DE gene identification
- name: PyGMQL
  role: Retrieves the TCGA gene expression data that are later loaded as the count
    matrices for the DE analysis
  repo: https://pygmql.readthedocs.io/en/latest/
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41540-021-00175-9
    title: Computational analysis of fused co-expression networks for the identification
      of candidate cancer gene biomarkers
  source_content_hash: sha256:8a5a5b360c8921908f7f89959a4d057eb9c85240eaa791e29eb33ce2e897e712
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DESeq2
  - PyGMQL
  - R/Bioconductor
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# cancer-biomarker-signature-generation

## Summary

Generates the typically-used differential-expression baseline gene signature by running DESeq2 (R/Bioconductor) on TCGA RNA-Seq gene count matrices for normal vs cancer samples of KIRC, LIHC and PRAD, yielding 1059 genes (54 miRNAs), 434 (27) and 218 (24) respectively. These DE sets, reduced via LASSO into DEL sets, benchmark the paper's fused co-expression IC gene sets in normal/cancer classification.

## When to use

Use when raw TCGA-style RNA-Seq gene count matrices for paired normal vs cancer samples (e.g., KIRC, LIHC, PRAD) must be turned into a differential-expression baseline signature to benchmark other gene sets by classification.

## When NOT to use

- The goal is the paper's proposed IC gene sets: those are extracted from fused co-expression networks via the disparity filter and the Gephi modularity method, not from DESeq2 - differential expression is only the comparison baseline here
- Input is already-normalized or filtered expression (e.g., zFPKM-normalized values) rather than raw integer gene counts, since DESeq2's dispersion model requires count data
- The task is finding survival-associated biomarkers: the article reports only 6 DE genes significantly associated with survival versus 44 IC genes, making the DE baseline the weaker source for that goal

## Inputs

- Preprocessed TCGA RNA-Seq gene count matrices for normal and cancer samples, per cancer type (KIRC, LIHC, PRAD)
- Sample condition labels (normal vs cancer) defining the DESeq2 comparison
- Gene symbol-to-gene type mapping (matrix_GTypeGName.xlsx) to separate RNA genes from miRNAs when counting signature members

## Outputs

- Per-cancer-type DE-gene baseline reference signature CSV, restricted to genes with adjusted p-value < 0.05 and |log2FC| > 2.0
- LASSO-reduced DEL (differentially expressed) gene sets per cancer type used as classification features
- Reported signature sizes per cancer type: 1059 genes/54 miRNAs (KIRC), 434/27 (LIHC), 218/24 (PRAD)

## How to apply

Load the preprocessed TCGA RNA-Seq gene count matrices for normal and cancer samples of each cancer type (KIRC, LIHC, PRAD); the data are retrieved via GMQL/PyGMQL and pre-filtered to transcripts with mean zFPKM higher than -3.0. Run the DESeq2 R/Bioconductor package on the count data comparing normal versus cancer conditions, using shrinkage estimators for dispersion and fold change, and compute the log2 fold change as the ratio between the expression mean values of cancer over normal samples. Correct for multiple testing with Benjamini-Hochberg and retain only genes with adjusted p-value < 0.05 and |log2 fold change| > 2.0. Write one DE-gene baseline reference signature per cancer type to a CSV table. The resulting DE sets - 1059 genes (54 miRNAs) for KIRC, 434 (27) for LIHC, 218 (24) for PRAD - are then LASSO-reduced into DEL sets and compared against the fused-network IC gene sets by normal/cancer classification, whose AUC/accuracy/F1 values are collected in the repository's Auc_acc_f1 tables.

## Related tools

- **DESeq2** (Runs the normal-versus-cancer differential expression test on the TCGA gene count matrices to produce the baseline DE signature)
- **R/Bioconductor** (Runtime environment in which the DESeq2 package is executed for DE gene identification)
- **PyGMQL** (Retrieves the TCGA gene expression data that are later loaded as the count matrices for the DE analysis) — https://pygmql.readthedocs.io/en/latest/

## Examples

```
Rscript -e 'library(DESeq2); dds <- DESeq(DESeqDataSetFromMatrix(countData, colData, ~condition)); res <- results(dds, alpha=0.05)'
```

## Evaluation signals

- The output CSV per cancer type must exist and be non-empty (workflow validation: 'the output CSV must exist')
- Each cancer-type DE gene list contains only genes passing adjusted p-value < 0.05 and |log2 fold change| > 2.0; no borderline genes retained
- All adjusted p-values in the output lie within 0-1
- Signature sizes approximate the reported baselines: 1059 genes (54 miRNAs) for KIRC, 434 (27) for LIHC, 218 (24) for PRAD
- Feeding the LASSO-reduced DEL set through extraction_classification.py yields AUC/accuracy/F1 entries comparable to the repository's Auc_acc_f1 performance tables for the IC gene sets

## Limitations

- The DE signature is a comparator, not the proposed biomarker: only 6 DE genes were significantly associated with the survival event probability versus 44 IC genes, so it underperforms for survival relevance
- The adjusted p < 0.05 and |log2FC| > 2.0 cutoffs and the per-cancer counts (1059/434/218 genes) are specific to these TCGA KIRC/LIHC/PRAD datasets and the zFPKM > -3.0 pre-filtering; other cohorts or cancers require recalibration
- Provenance caveat: the 0.05/2.0 thresholds and CSV validation rules are documented in the task workflow card, while the article excerpt itself only names DESeq2 (R/Bioconductor) as the DE tool

## Evidence

- [methods] DESeq2 R/Bioconductor identifies the differentially expressed genes: "We leverage the DESeq2 R/Bioconductor package to identify the differentially expressed (DE) genes"
- [intro] Differential expression analysis is the usual baseline method for biomarker identification: "We also compare our results with the ones from differential gene expression analysis, which is the usual baseline method to identify biomarkers in biology"
- [abstract] Relevance validated by normal/cancer classification against DE gene signatures: "comparing the performance of normal/cancer classification for our identified gene sets and other gene signatures, including the typically-used differentially expressed genes"
- [readme] Repository exposes classification performance tables used in the baseline comparison: "one folder [Auc_acc_f1](https://github.com/DEIB-GECO/GeneNetFusion/tree/master/Auc_acc_f1) that contains the tables with the performances values"
