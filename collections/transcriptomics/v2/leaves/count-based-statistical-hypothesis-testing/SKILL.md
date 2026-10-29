---
name: count-based-statistical-hypothesis-testing
description: Use when you have a raw RNA-seq count matrix plus sample metadata with
  infected vs mock conditions (e.g., GEO GSE147507) and need DEGs for downstream pathway
  or metabolic model mapping.
when_to_use_negative:
- Input is already-normalized expression (FPKM/TPM) rather than raw counts — DESeq2
  requires raw counts; FPKM/TPM data is instead the input for rFASTCORMICS model reconstruction.
- The goal is single-sample context-specific metabolic model reconstruction rather
  than statistical comparison of conditions.
- The condition of interest shows no differentially expressed metabolic genes (like
  A549_0.02), in which case it should be discarded from downstream DEG pathway analysis
  rather than forced through.
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: DESeq2
  role: negative-binomial differential expression testing of infected vs mock counts
    with thresholds absolute log2FC > 1 and adjusted p-value < 0.05
- name: edgeR
  role: filterByExpr (Version 3.30.3) to remove low-expression genes before DESeq2
    testing
- name: R
  role: runtime environment for loading counts/metadata and running the DESeq2/edgeR
    pipeline
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.isci.2021.103331
    title: 'DCcov: Repositioning of drugs and drug combinations for SARS-CoV-2 infected
      lung through constraint-based modeling'
  source_content_hash: sha256:06372effd110e07ecf75b7dbfc7c9233a74c47b4a41abc95dc1696cad80046bb
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
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# count-based-statistical-hypothesis-testing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Compute differentially expressed genes from RNA-seq read counts using DESeq2 negative-binomial testing on the GSE147507 severity study (SARS-CoV-2 infected vs mock lung cells), after low-expression filtering with edgeR's filterByExpr.

## When to use

Use when you have a raw RNA-seq count matrix plus sample metadata with infected vs mock conditions (e.g., GEO GSE147507) and need DEGs for downstream pathway or metabolic model mapping.

## When NOT to use

- Input is already-normalized expression (FPKM/TPM) rather than raw counts — DESeq2 requires raw counts; FPKM/TPM data is instead the input for rFASTCORMICS model reconstruction.
- The goal is single-sample context-specific metabolic model reconstruction rather than statistical comparison of conditions.
- The condition of interest shows no differentially expressed metabolic genes (like A549_0.02), in which case it should be discarded from downstream DEG pathway analysis rather than forced through.

## Inputs

- raw RNA-seq read count matrix from GSE147507 (severity study)
- sample metadata for the infected vs mock contrast (GEO GSE147507)

## Outputs

- DEG_infected_vs_mock.csv with gene identifiers, log2 fold change, adjusted p-value, and base mean
- set of DEGs at adjusted p-value < 0.05 and |log2FC| > 1 (e.g., the A549_0.02 condition yielded no differentially expressed metabolic genes)

## How to apply

Download the raw read count matrix and sample metadata of the severity study GSE147507 from NCBI GEO, then in R assemble the infected vs mock contrast design table. Filter low-expression genes with edgeR's filterByExpr (Version 3.30.3) so only well-expressed genes enter testing. Run DESeq2 on the filtered counts for the infected vs mock comparison and apply the significance thresholds of adjusted p-value < 0.05 and absolute log2 fold change > 1. Write the filtered DEG table (gene identifiers, log2 fold change, adjusted p-value, base mean) to DEG_infected_vs_mock.csv; note the decision point that the A549_0.02 condition showed no differentially expressed metabolic genes and was discarded from the DEGs metabolic pathway analysis.

## Related tools

- **DESeq2** (negative-binomial differential expression testing of infected vs mock counts with thresholds absolute log2FC > 1 and adjusted p-value < 0.05)
- **edgeR** (filterByExpr (Version 3.30.3) to remove low-expression genes before DESeq2 testing)
- **R** (runtime environment for loading counts/metadata and running the DESeq2/edgeR pipeline)

## Examples

```
Rscript -e 'counts <- read.csv("GSE147507_counts.csv"); keep <- edgeR::filterByExpr(counts, design); res <- DESeq2::results(DESeq2::DESeqDataSetFromMatrix(counts[keep,], meta, ~condition)); write.csv(subset(res, padj<0.05 & abs(log2FoldChange)>1), "DEG_infected_vs_mock.csv")'
```

## Evaluation signals

- Output CSV DEG_infected_vs_mock.csv contains exactly the columns: gene identifiers, log2 fold change, adjusted p-value, base mean.
- All retained DEGs satisfy adjusted p-value < 0.05 and absolute log2 fold change > 1.
- Genes removed by filterByExpr (edgeR 3.30.3) are absent from the final DEG table.
- The A549_0.02 condition yields no differentially expressed metabolic genes and is excluded from DEG metabolic pathway analysis.
- PCA of severity study samples separates clearly by cell type, confirming the design grouping used in the contrast.

## Limitations

- The A549_0.02 condition can show no differentially expressed metabolic genes, blocking downstream pathway analysis for that condition.
- Results are limited to the modeled lung cell lines (A549, Calu-3, H1299, NHBE) in the severity study.
- DEG-based pathway conclusions depend on the edgeR filterByExpr step and the chosen |log2FC| > 1, adjusted p < 0.05 cutoffs.

## Evidence
