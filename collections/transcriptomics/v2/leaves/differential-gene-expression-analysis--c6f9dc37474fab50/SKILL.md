---
name: differential-gene-expression-analysis
description: Use when a raw read count matrix plus sample metadata (e.g. GSE147507
  severity study) must be tested for infected-vs-mock differential expression under
  |log2FC|>1 and adj p<0.05 thresholds.
when_to_use_negative:
- Input is already normalized expression (FPKM/TPM) rather than raw counts — DESeq2
  requires raw counts; FPKM/TPM are the input format for rFASTCORMICS, not for this
  DEG testing step
- The aim is pathway enrichment on pre-computed gene lists rather than testing for
  differential expression itself (use Enrichr/GSEApy instead)
- The contrast involves no biological replicates, so dispersion cannot be estimated
  for DESeq2 testing
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: DESeq2
  role: Differential expression testing on filtered counts for the infected vs mock
    comparison
- name: edgeR
  role: filterByExpr (Version 3.30.3) to filter out genes with low expression values
    before DESeq2 testing
- name: R
  role: Execution environment (r-base) for loading counts/metadata, design table assembly,
    and DEG testing
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.isci.2021.103331
    title: 'DCcov: Repositioning of drugs and drug combinations for SARS-CoV-2 infected
      lung through constraint-based modeling'
  source_content_hash: sha256:da6dc7fe695d11e0c260b67dcea85b5d07da81cc613eea94261e0966ccd4c33f
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
  - R/Bioconductor
  - edgeR
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# differential-gene-expression-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes differentially expressed genes (DEGs) between SARS-CoV-2-infected and mock lung cell conditions on a raw RNA-seq count matrix (e.g. GEO GSE147507) using DESeq2, after filtering low-expression genes with edgeR's filterByExpr, and applies the significance thresholds |log2 fold change| > 1 and adjusted p-value < 0.05.

## When to use

Use when a raw read count matrix plus sample metadata (e.g. GSE147507 severity study) must be tested for infected-vs-mock differential expression under |log2FC|>1 and adj p<0.05 thresholds.

## When NOT to use

- Input is already normalized expression (FPKM/TPM) rather than raw counts — DESeq2 requires raw counts; FPKM/TPM are the input format for rFASTCORMICS, not for this DEG testing step
- The aim is pathway enrichment on pre-computed gene lists rather than testing for differential expression itself (use Enrichr/GSEApy instead)
- The contrast involves no biological replicates, so dispersion cannot be estimated for DESeq2 testing

## Inputs

- Raw RNA-seq read count matrix from NCBI GEO (e.g. GSE147507 severity study)
- Sample metadata table with infected/mock condition labels (e.g. A549, Calu-3, H1299, NHBE cell types and viral load conditions)

## Outputs

- Filtered DEG table as CSV (DEG_infected_vs_mock.csv) with columns: gene identifiers, log2 fold change, adjusted p-value, base mean
- Condition-level DEG sets used as input to downstream metabolic pathway analysis (Enrichr / Recon3D_01 subSystems mapping)

## How to apply

Download the raw read count matrix and sample metadata from NCBI GEO (e.g. GSE147507), assemble the infected-vs-mock contrast design table in R (r-base), and filter out genes with low expression values using edgeR's filterByExpr (edgeR Version 3.30.3) before testing. Run DESeq2 on the filtered counts for the infected-vs-mock comparison, then apply the DEG thresholds of adjusted p-value < 0.05 and absolute log2 fold change > 1. Write the filtered DEG table (gene identifiers, log2 fold change, adjusted p-value, base mean) to a CSV such as DEG_infected_vs_mock.csv. Check condition-specific results: if a condition (e.g. A549_0.02) yields no differentially expressed metabolic genes, discard it from downstream DEG metabolic pathway analysis.

## Related tools

- **DESeq2** (Differential expression testing on filtered counts for the infected vs mock comparison)
- **edgeR** (filterByExpr (Version 3.30.3) to filter out genes with low expression values before DESeq2 testing)
- **R** (Execution environment (r-base) for loading counts/metadata, design table assembly, and DEG testing)

## Examples

```
dds <- DESeqDataSetFromMatrix(countData=filtered_counts, colData=metadata, design=~condition); res <- results(dds, contrast=c("condition","infected","mock"))
```

## Evaluation signals

- Output CSV contains exactly the columns gene identifiers, log2 fold change, adjusted p-value, base mean
- All retained DEGs satisfy absolute log2 fold change > 1 and adjusted p-value < 0.05
- Low-expression genes were removed with edgeR filterByExpr (Version 3.30.3) prior to DESeq2, so gene counts tested are reduced relative to the raw matrix
- Conditions yielding no differentially expressed metabolic genes (e.g. A549_0.02) are identified and excluded from downstream DEG metabolic pathway analysis
- The design table encodes the infected vs mock contrast from the sample metadata

## Limitations

- The A549_0.02 condition showed no differentially expressed metabolic genes and had to be discarded from the DEGs metabolic pathway analysis
- Analysis limited by the modeled lung cell lines (A549, Calu-3, H1299, NHBE) in the GSE147507 severity study
- DEG results depend on the chosen thresholds (adj p 0.05, |logFC| 1); conditions with few DE metabolic genes cannot support pathway-level conclusions

## Evidence
