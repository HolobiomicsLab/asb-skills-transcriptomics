---
name: differential-expression-thresholding
description: Use when raw RNA-seq count data from infected and control conditions
  (e.g. GEO GSE147507 / GSE148729 lung samples) must be reduced to binary expressed/not-expressed
  gene lists under |log2FC|>1, padj<0.05 thresholds to drive rFASTCORMICS model pruning.
when_to_use_negative:
- Input is microarray .CEL data rather than RNA-seq counts — use the FASTCORMICS BARCODE
  workflow (frma + hugene/hgu133a vectors) instead
- Only FPKM/TPM-transformed values without raw counts are available, since DESeq2
  statistical testing requires raw count data
- The goal is continuous expression quantification rather than binary expressed/not-expressed
  classification for model pruning
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: DESeq2
  role: Computes differentially expressed genes per condition with absolute log2 fold
    change >1 and adjusted p-value < 0.05
- name: edgeR
  role: Filters out genes with low expression values via the filterByExpr function
    (Version 3.30.3) before DEG testing
- name: R
  role: Runtime environment for running DESeq2 and edgeR filtering
- name: rFASTCORMICS
  role: Downstream consumer of the binarized DEG data (1/-1/0) used to prune context-specific
    metabolic models
  repo: https://github.com/sysbiolux/rFASTCORMICS
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.isci.2021.103331
    title: 'DCcov: Repositioning of drugs and drug combinations for SARS-CoV-2 infected
      lung through constraint-based modeling'
  source_content_hash: sha256:817fe0598179121b7bff978c5e6fc207daedd49ca15c7e518c0f5a4976ea4ae5
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/sysbiolux/DCcov
  repo_url: https://github.com/sysbiolux/rFASTCORMICS
  tools:
  - COBRA Toolbox
  - DESeq2
  - MATLAB
  - R
  - edgeR
  - rFASTCORMICS
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# differential-expression-thresholding

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Compute condition-specific differentially expressed genes from SARS-CoV-2 infected vs mock lung RNA-seq datasets using DESeq2 with an absolute log2 fold change >1 and adjusted p-value < 0.05, after filtering low-expression genes with edgeR's filterByExpr, then binarize the result for context-specific metabolic model reconstruction.

## When to use

Use when raw RNA-seq count data from infected and control conditions (e.g. GEO GSE147507 / GSE148729 lung samples) must be reduced to binary expressed/not-expressed gene lists under |log2FC|>1, padj<0.05 thresholds to drive rFASTCORMICS model pruning.

## When NOT to use

- Input is microarray .CEL data rather than RNA-seq counts — use the FASTCORMICS BARCODE workflow (frma + hugene/hgu133a vectors) instead
- Only FPKM/TPM-transformed values without raw counts are available, since DESeq2 statistical testing requires raw count data
- The goal is continuous expression quantification rather than binary expressed/not-expressed classification for model pruning

## Inputs

- Raw RNA-seq count matrix (R genes x C samples) with gene identifiers as rownames and sample names as colnames
- Sample/condition metadata from GEO studies (e.g. GSE147507 severity study, GSE148729 time-series study)
- Gene identifier mapping table (dico) linking experimental gene IDs to genes in Recon2.04 or Recon3D_01

## Outputs

- Per-condition DEG lists with log2 fold changes and adjusted p-values passing |log2FC| > 1, padj < 0.05
- Binarized expression vectors (1 / -1 / 0) mapped onto generic-model gene identifiers for rFASTCORMICS
- Record of conditions discarded for having no differentially expressed metabolic genes

## How to apply

First filter low-expression genes from the raw count matrix using edgeR's filterByExpr (Version 3.30.3). Then compute DEGs per condition with DESeq2, keeping only genes with absolute log2 fold change >1 and adjusted p-value < 0.05. Binarize the expression onto the generic model genes (Recon2.04 or Recon3D_01) as 1 for expressed, -1 for not expressed, and 0 for unknown, since rFASTCORMICS consumes this discretized vector to prune each model per sample. Discard conditions that show no differentially expressed metabolic genes (as done for A549_0.02), and discard enrichment results based on only one gene when analyzing the DEGs downstream. This same thresholding is reapplied in a second pass described with 'an adjusted p-value threshold of 0.05, and an absolute log fold change threshold of 1', so the cutoffs are consistent across the pipeline.

## Related tools

- **DESeq2** (Computes differentially expressed genes per condition with absolute log2 fold change >1 and adjusted p-value < 0.05)
- **edgeR** (Filters out genes with low expression values via the filterByExpr function (Version 3.30.3) before DEG testing)
- **R** (Runtime environment for running DESeq2 and edgeR filtering)
- **rFASTCORMICS** (Downstream consumer of the binarized DEG data (1/-1/0) used to prune context-specific metabolic models) — https://github.com/sysbiolux/rFASTCORMICS

## Examples

```
Rscript -e 'library(edgeR); library(DESeq2); keep <- filterByExpr(counts, group); res <- results(dds); degs <- subset(res, abs(log2FoldChange) > 1 & padj < 0.05)'
```

## Evaluation signals

- Every retained gene satisfies both invariants: absolute log2 fold change > 1 and adjusted p-value < 0.05
- Low-expression genes were removed first with edgeR filterByExpr, so no ultra-low-count gene appears in the DEG list
- Conditions lacking differentially expressed metabolic genes (e.g. A549_0.02) are absent from the downstream analysis
- Binarized vectors contain only values 1, -1, and 0, and their gene identifiers map onto Recon2.04/Recon3D_01 genes via the dico table
- Feeding the binarized data to rFASTCORMICS reproduces the reported 50 models (28 infected, 22 mock) with median 3646 metabolites and 2456.5 reactions

## Limitations

- Conditions with no differentially expressed metabolic genes (e.g. A549_0.02) yield no usable input and must be discarded
- rFASTCORMICS expects RNA-seq data preferably FPKM transformed, so normalization choice affects the discretization
- The downstream analysis is limited by the modeled lung cell lines (A549, Calu-3, H1299, NHBE); findings do not automatically generalize to other tissues

## Evidence

- [readme] Binarized input format for rFASTCORMICS: [evidence span withheld — anchor status: external_doc]
