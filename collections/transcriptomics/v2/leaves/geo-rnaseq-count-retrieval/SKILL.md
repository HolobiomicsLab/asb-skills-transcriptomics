---
name: geo-rnaseq-count-retrieval
description: Use when an analysis needs raw (unnormalized) RNA-seq count data plus
  sample metadata from a public GEO series to run a bulk-RNA-seq contrast such as
  SARS-CoV-2-infected vs mock lung cells.
when_to_use_negative:
- The input data are already FPKM/TPM-transformed or otherwise normalized — DESeq2
  requires raw counts, and normalized expression is used by rFASTCORMICS-style model
  reconstruction instead of DEG testing
- The target data are microarray .CEL files — these follow a BARCODE/fRMA normalization
  route (FASTCORMICS), not GEO raw-count retrieval with DESeq2
- The goal is single-cell or spliced-alignment processing rather than a bulk count-matrix
  contrast, since no alignment or quantification steps are defined in this workflow
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: DESeq2
  role: Runs the infected-vs-mock differential expression testing on the retrieved
    and filtered count matrix (absolute log2 fold change >1, adjusted p-value < 0.05)
- name: edgeR
  role: Provides the filterByExpr function (Version 3.30.3) used to remove low-expression
    genes from the retrieved count matrix before DESeq2 testing
- name: R
  role: Runtime environment (installed via apt install r-base r-base-core r-recommended
    r-base-dev) in which the count matrix, metadata, and design table are loaded and
    assembled
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.isci.2021.103331
    title: 'DCcov: Repositioning of drugs and drug combinations for SARS-CoV-2 infected
      lung through constraint-based modeling'
  source_content_hash: sha256:4e256c31066600f75d9f8b68f45f94b2bf9bacb4b97a829e5fa56e0b887ebd9a
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
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# geo-rnaseq-count-retrieval

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Retrieve a raw RNA-seq read count matrix and sample metadata from an NCBI GEO accession (e.g. the severity study GSE147507) and assemble them into an R count matrix and design table ready for DESeq2 differential expression testing.

## When to use

Use when an analysis needs raw (unnormalized) RNA-seq count data plus sample metadata from a public GEO series to run a bulk-RNA-seq contrast such as SARS-CoV-2-infected vs mock lung cells.

## When NOT to use

- The input data are already FPKM/TPM-transformed or otherwise normalized — DESeq2 requires raw counts, and normalized expression is used by rFASTCORMICS-style model reconstruction instead of DEG testing
- The target data are microarray .CEL files — these follow a BARCODE/fRMA normalization route (FASTCORMICS), not GEO raw-count retrieval with DESeq2
- The goal is single-cell or spliced-alignment processing rather than a bulk count-matrix contrast, since no alignment or quantification steps are defined in this workflow

## Inputs

- NCBI GEO accession of a bulk RNA-seq study (e.g. GSE147507 severity study) with a raw read count matrix
- Sample metadata table mapping sample identifiers to condition labels (infected vs mock, cell type, viral load)
- R session with DESeq2 and edgeR (Version 3.30.3) installed

## Outputs

- R count matrix (genes x samples, raw integer counts) aligned to a design table for the infected-vs-mock contrast
- Filtered DEG table written to DEG_infected_vs_mock.csv with columns: gene identifiers, log2 fold change, adjusted p-value, base mean

## How to apply

Download the raw read count matrix and sample metadata of the target study (e.g. GSE147507 from https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSE147507) and load both in R (r-base). Assemble an infected-vs-mock contrast design table from the metadata, matching sample IDs to condition labels (cell type and viral load, e.g. A549, Calu-3, NHBE conditions in GSE147507). Keep raw integer counts — do not supply FPKM/TPM, since DESeq2 models raw counts; FPKM/TPM inputs are reserved for tools like rFASTCORMICS. Before testing, filter out genes with low expression values using edgeR's filterByExpr (edgeR Version 3.30.3) to retain only well-expressed genes, then hand the filtered counts and design table to DESeq2 with significance thresholds of adjusted p-value < 0.05 and absolute log2 fold change > 1. Write the filtered DEG table (gene identifiers, log2 fold change, adjusted p-value, base mean) to DEG_infected_vs_mock.csv.

## Related tools

- **DESeq2** (Runs the infected-vs-mock differential expression testing on the retrieved and filtered count matrix (absolute log2 fold change >1, adjusted p-value < 0.05))
- **edgeR** (Provides the filterByExpr function (Version 3.30.3) used to remove low-expression genes from the retrieved count matrix before DESeq2 testing)
- **R** (Runtime environment (installed via apt install r-base r-base-core r-recommended r-base-dev) in which the count matrix, metadata, and design table are loaded and assembled)

## Examples

```
Rscript -e 'dge <- DGEList(counts); keep <- filterByExpr(dge, design); dge <- dge[keep,]; dds <- DESeqDataSetFromMatrix(counts(dge), coldata, design)'
```

## Evaluation signals

- The downloaded matrix contains raw integer counts (no negative or fractional values) and its columns match the sample identifiers in the metadata table one-to-one
- After filterByExpr filtering, only well-expressed genes remain and the design table resolves the infected vs mock contrast without dropped or mislabeled samples
- The DEG table respects the thresholds: every reported gene has adjusted p-value < 0.05 and absolute log2 fold change > 1
- DEG_infected_vs_mock.csv contains the required columns: gene identifiers, log2 fold change, adjusted p-value, base mean
- Conditions with no detectable signal are handled explicitly (e.g. in the DCcov analysis, the A549_0.02 condition showed no differentially expressed metabolic genes and was discarded from the DEGs metabolic pathway analysis)

## Limitations

- Downstream DEG-based pathway analysis can be empty for particular conditions — in the severity study GSE147507, the A549_0.02 condition had no differentially expressed metabolic genes and was discarded
- The DCcov workflow applied this retrieval only to modeled lung cell lines (A549, Calu-3, H1299, NHBE), so generalization to other tissues or cell types is untested
- No alignment or quantification from FASTQ is performed here; the skill depends on the GEO series already providing a raw count matrix

## Evidence

- [other] Download the raw read count matrix and sample metadata of the severity study GSE147507 from NCBI GEO: [evidence span withheld — anchor status: not_found]
