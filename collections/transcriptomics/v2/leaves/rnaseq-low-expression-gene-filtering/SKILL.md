---
name: rnaseq-low-expression-gene-filtering
description: Use when a raw RNA-seq count matrix plus sample metadata (e.g., GEO GSE147507
  infected vs mock lung cells) must be filtered for low-expression genes before DESeq2
  DEG testing.
when_to_use_negative:
- Input expression data are already FPKM/TPM normalized (rFASTCORMICS-style input)
  rather than raw counts — filterByExpr expects raw count data
- The design lacks biological replicates or a defined group/contrast, so filterByExpr's
  library-size-aware filtering cannot be applied per the infected vs mock design
- The goal is context-specific metabolic model reconstruction from binarized expression
  rather than DEG calling — skip DEG filtering and go straight to rFASTCORMICS
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: edgeR
  role: Provides the filterByExpr function (Version 3.30.3) used to remove low-expression
    genes from the raw count matrix before DESeq2 testing
- name: DESeq2
  role: Runs differential expression testing on the filterByExpr-retained counts for
    the infected vs mock contrast
- name: R
  role: Execution environment (r-base) for loading the count matrix/metadata and running
    the filtering and DESeq2 steps
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.isci.2021.103331
    title: 'DCcov: Repositioning of drugs and drug combinations for SARS-CoV-2 infected
      lung through constraint-based modeling'
  source_content_hash: sha256:8a5db39e5f6fe692282149e32be0041fb6846258012d8142aa459aad6ff441d1
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
n_claims: 9
---

# rnaseq-low-expression-gene-filtering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Pre-testing cleanup of raw RNA-seq count matrices in which weakly expressed genes are removed with edgeR's filterByExpr (Version 3.30.3) so that downstream DESeq2 differential expression testing (e.g., SARS-CoV-2 infected vs mock in GSE147507) runs only on well-expressed genes.

## When to use

Use when a raw RNA-seq count matrix plus sample metadata (e.g., GEO GSE147507 infected vs mock lung cells) must be filtered for low-expression genes before DESeq2 DEG testing.

## When NOT to use

- Input expression data are already FPKM/TPM normalized (rFASTCORMICS-style input) rather than raw counts — filterByExpr expects raw count data
- The design lacks biological replicates or a defined group/contrast, so filterByExpr's library-size-aware filtering cannot be applied per the infected vs mock design
- The goal is context-specific metabolic model reconstruction from binarized expression rather than DEG calling — skip DEG filtering and go straight to rFASTCORMICS

## Inputs

- Raw RNA-seq read count matrix (genes x samples) from NCBI GEO (GSE147507 severity study)
- Sample metadata table defining infected vs mock conditions per sample

## Outputs

- Filtered count matrix containing only genes retained by edgeR filterByExpr
- DEG_infected_vs_mock.csv with gene identifiers, log2 fold change, adjusted p-value, and base mean for genes passing |log2FC| > 1 and adjusted p-value < 0.05

## How to apply

Load the raw read count matrix and sample metadata in R (r-base) and assemble the infected vs mock contrast design table. Apply edgeR's filterByExpr function (Version 3.30.3) to the raw counts, using the design groups, to keep only well-expressed genes and drop low-expression genes before testing. Run DESeq2 differential expression on the filtered counts for the infected vs mock comparison, then apply the significance thresholds of absolute log2 fold change >1 and adjusted p-value <0.05. As a sanity decision point, note that a condition such as A549_0.02 yielded no differentially expressed metabolic genes and was discarded from downstream DEG metabolic pathway analysis. Write the filtered DEG table (gene identifiers, log2 fold change, adjusted p-value, base mean) to DEG_infected_vs_mock.csv.

## Related tools

- **edgeR** (Provides the filterByExpr function (Version 3.30.3) used to remove low-expression genes from the raw count matrix before DESeq2 testing)
- **DESeq2** (Runs differential expression testing on the filterByExpr-retained counts for the infected vs mock contrast)
- **R** (Execution environment (r-base) for loading the count matrix/metadata and running the filtering and DESeq2 steps)

## Examples

```
keep <- edgeR::filterByExpr(counts, group=samples$condition); counts_filt <- counts[keep, ]
```

## Evaluation signals

- All genes in the filtered matrix pass filterByExpr; genes below the expression cutoff computed from library sizes and the infected/mock design groups are absent
- DEG_infected_vs_mock.csv contains exactly the columns gene identifiers, log2 fold change, adjusted p-value, and base mean
- Every reported DEG satisfies |log2 fold change| > 1 and adjusted p-value < 0.05
- Downstream check: the A549_0.02 condition is expected to show no differentially expressed metabolic genes and to be excluded from DEG metabolic pathway analysis
- Filtering is applied before DESeq2 testing, not after — verify the DESeq2 object was built from the filtered counts

## Limitations

- filterByExpr behavior depends on the design/groups supplied; an incorrect infected vs mock contrast table changes which genes are retained
- A condition with no differentially expressed metabolic genes after filtering and testing (e.g., A549_0.02) cannot contribute to DEG-based metabolic pathway analysis
- The DEG analysis is limited to the modeled lung cell lines (A549, Calu-3, H1299, NHBE) in the severity study

## Evidence
