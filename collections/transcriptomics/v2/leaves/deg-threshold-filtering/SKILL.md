---
name: deg-threshold-filtering
description: Use when raw RNA-seq read counts with infected-vs-mock sample metadata
  are available and a stringent list of differentially expressed genes (|log2FC| >
  1, adjusted p-value < 0.05) is needed for downstream pathway or modeling analyses.
when_to_use_negative:
- Input is already a normalized expression matrix or FPKM/TPM table rather than raw
  counts, since DESeq2 expects raw integer counts for its dispersion model
- The goal is unsupervised sample clustering or PCA rather than a contrast-based DEG
  list; PCA of severity study samples was done separately in this pipeline
- The contrast of interest has no replication or a condition yields no differentially
  expressed metabolic genes (e.g., A549_0.02 was discarded), so threshold filtering
  cannot produce a usable DEG set
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: DESeq2
  role: Performs negative-binomial differential expression testing on filtered raw
    counts for the infected vs mock contrast
- name: edgeR
  role: Its filterByExpr function (Version 3.30.3) removes low-expression genes before
    DESeq2 testing
- name: R
  role: Environment in which the count matrix, metadata, edgeR filtering, and DESeq2
    testing are executed (installed via apt install r-base)
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.isci.2021.103331
    title: 'DCcov: Repositioning of drugs and drug combinations for SARS-CoV-2 infected
      lung through constraint-based modeling'
  source_content_hash: sha256:bf34a80c56ae4fbb5a83abae1118c6cc060b14a3653afeef3946db8a5033b199
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/sysbiolux/DCcov
  tools:
  - DESeq2
  - R
  - edgeR
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 5
---

# deg-threshold-filtering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Applies DESeq2 differential expression testing followed by dual significance thresholds (absolute log2 fold change >1 and adjusted p-value <0.05) to extract a robust DEG set from RNA-seq count data, as used in the DCcov pipeline on the SARS-CoV-2 severity study GSE147507.

## When to use

Use when raw RNA-seq read counts with infected-vs-mock sample metadata are available and a stringent list of differentially expressed genes (|log2FC| > 1, adjusted p-value < 0.05) is needed for downstream pathway or modeling analyses.

## When NOT to use

- Input is already a normalized expression matrix or FPKM/TPM table rather than raw counts, since DESeq2 expects raw integer counts for its dispersion model
- The goal is unsupervised sample clustering or PCA rather than a contrast-based DEG list; PCA of severity study samples was done separately in this pipeline
- The contrast of interest has no replication or a condition yields no differentially expressed metabolic genes (e.g., A549_0.02 was discarded), so threshold filtering cannot produce a usable DEG set

## Inputs

- Raw RNA-seq read count matrix (genes x samples) from NCBI GEO (e.g., GSE147507)
- Sample metadata describing infected and mock conditions for building the DESeq2 design table

## Outputs

- Filtered DEG table (gene identifiers, log2 fold change, adjusted p-value, base mean) written to DEG_infected_vs_mock.csv
- Set of genes passing |log2FC| > 1 and adjusted p-value < 0.05, used for downstream metabolic pathway analysis

## How to apply

Download the raw read count matrix and sample metadata (e.g., GSE147507 from GEO) and assemble the infected vs mock contrast design table in R. First remove poorly expressed genes using edgeR's filterByExpr function (Version 3.30.3) so testing is restricted to well-expressed genes. Run DESeq2 differential expression testing on the filtered counts for the infected vs mock comparison, then retain only genes meeting both cutoffs: absolute log2 fold change >1 and adjusted p-value <0.05. Write the resulting filtered table (gene identifiers, log2 fold change, adjusted p-value, base mean) to a CSV such as DEG_infected_vs_mock.csv. Decision point: if a condition (e.g., A549_0.02) yields no differentially expressed metabolic genes, it should be discarded from downstream DEG-based metabolic pathway analysis.

## Related tools

- **DESeq2** (Performs negative-binomial differential expression testing on filtered raw counts for the infected vs mock contrast)
- **edgeR** (Its filterByExpr function (Version 3.30.3) removes low-expression genes before DESeq2 testing)
- **R** (Environment in which the count matrix, metadata, edgeR filtering, and DESeq2 testing are executed (installed via apt install r-base))

## Examples

```
res <- DESeq2::results(dds); deg <- subset(res, abs(log2FoldChange) > 1 & padj < 0.05); write.csv(deg, "DEG_infected_vs_mock.csv")
```

## Evaluation signals

- Every retained DEG must satisfy both thresholds: absolute log2 fold change > 1 AND adjusted p-value < 0.05
- Output CSV must contain the four required columns: gene identifiers, log2 fold change, adjusted p-value, base mean
- The tested gene universe should shrink after edgeR filterByExpr filtering compared to the raw count matrix rows
- For the A549_0.02 condition in GSE147507, the expected result is zero differentially expressed metabolic genes, triggering exclusion from downstream DEG metabolic pathway analysis
- Sanity-check that the design table correctly pairs infected and mock samples from the GSE147507 metadata before testing

## Limitations

- The A549_0.02 condition showed no differentially expressed metabolic genes and was discarded from the DEGs metabolic pathway analysis, showing the thresholds can yield empty sets for some conditions
- Thresholds (adjusted p-value 0.05, |logFC| 1) are analysis-specific; applying them mechanically may discard biologically relevant moderate fold changes
- Filtering relies on edgeR filterByExpr (Version 3.30.3); different filtering choices before testing will change the final DEG set

## Evidence
