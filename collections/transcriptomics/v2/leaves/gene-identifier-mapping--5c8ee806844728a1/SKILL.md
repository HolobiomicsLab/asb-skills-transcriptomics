---
name: gene-identifier-mapping
description: Use when an RNA-seq counts matrix carries Ensembl gene IDs but downstream
  GSVA, .gmt gene-set matching, TPM integration, or differential analysis in ARMT
  requires gene-symbol identifiers.
when_to_use_negative:
- The expression matrix already uses gene symbol row names and only needs TPM normalization
  or GSVA scoring without identifier conversion
- The input is mutation annotation (.maf) or clinical data, which ARMT consumes directly
  without gene identifier mapping
- You need de novo identifier mapping outside ARMT's shiny GUI workflow (e.g. for
  miRNA or non-Homo sapiens annotations, which the paper's org.Hs.eg.db-based setup
  does not cover)
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: ARMT
  role: Performs the Ensembl-to-Symbol transfer during normalization on the 'Normalization&GSVA'
    page and builds .gmt files from two-column .csv gene set files on the 'Data' page
  repo: https://github.com/Dulab2020/ARMT
- name: GSVA
  role: Consumes the symbol-mapped TPM matrix and the .gmt gene sets to compute GSVA
    scores
- name: GSVAdata
  role: Source of curated gene set collections (e.g. c2BroadSets) used to construct
    custom .gmt files with symbol-level members
- name: org.Hs.eg.db
  role: Bioconductor annotation package shipped as an ARMT dependency, supporting
    Ensembl ID to gene Symbol transfer
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.csbj.2021.08.009
    title: 'ARMT: An automatic RNA-seq data mining tool based on comprehensive and
      integrative analysis in cancer research'
  source_content_hash: sha256:77618e2cd2d06b9aefb90cff766772711b2679aeb586e42d5a7b1932763aca84
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/Dulab2020/ARMT
  repo_url: https://github.com/Dulab2020/ARMT
  tools:
  - ARMT
  - GSVA
  - GSVAdata
  - org.Hs.eg.db
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# gene-identifier-mapping

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Maps gene identifiers between Ensembl IDs and HGNC gene symbols within the ARMT RNA-seq pipeline, so that raw counts matrices (Ensembl ID rownames) can be normalized to TPM/log2(TPM+1) and matched against gene-set (.gmt) inputs that use gene symbols.

## When to use

Use when an RNA-seq counts matrix carries Ensembl gene IDs but downstream GSVA, .gmt gene-set matching, TPM integration, or differential analysis in ARMT requires gene-symbol identifiers.

## When NOT to use

- The expression matrix already uses gene symbol row names and only needs TPM normalization or GSVA scoring without identifier conversion
- The input is mutation annotation (.maf) or clinical data, which ARMT consumes directly without gene identifier mapping
- You need de novo identifier mapping outside ARMT's shiny GUI workflow (e.g. for miRNA or non-Homo sapiens annotations, which the paper's org.Hs.eg.db-based setup does not cover)

## Inputs

- Counts matrix in .csv with row names as gene Ensembl IDs and column names as sample IDs
- Gene set file in two-column .csv format (column 1: geneset names, column 2: genes of the corresponding geneset), e.g. sets extracted from c2BroadSets in GSVAdata
- Optional existing .gmt file of arbitrary gene sets for GSVA input

## Outputs

- TPM matrix in .csv with gene symbol row names and sample ID column names (log2(TPM+1) used for GSVA)
- .gmt file of arbitrary gene sets produced via ARMT's 'Data' page
- GSVA score matrix keyed by gene sets whose members are gene symbols

## How to apply

Load the counts matrix as a .csv file whose rows are gene Ensembl IDs and whose columns are sample IDs, then run ARMT's 'Normalization&GSVA' page: the tool normalizes counts to a TPM matrix and uses log2(TPM) for GSVA, converting Ensembl IDs to Symbols through normalization (dependencies such as org.Hs.eg.db and GSEAbase from Bioconductor support this). The output TPM matrix must carry gene symbol rownames so it can be integrated with clinical, GSVA-score, and mutation (.maf) data on common samples, and so gene sets supplied as a two-column .csv (first column: geneset names, second column: member genes) can be written to a .gmt file whose symbol-level members match the expression matrix. Validate that the symbol-mapped TPM rows and .gmt gene members overlap, and that the .gmt parses with GSEABase/GSVA before score calculation.

## Related tools

- **ARMT** (Performs the Ensembl-to-Symbol transfer during normalization on the 'Normalization&GSVA' page and builds .gmt files from two-column .csv gene set files on the 'Data' page) — https://github.com/Dulab2020/ARMT
- **GSVA** (Consumes the symbol-mapped TPM matrix and the .gmt gene sets to compute GSVA scores)
- **GSVAdata** (Source of curated gene set collections (e.g. c2BroadSets) used to construct custom .gmt files with symbol-level members)
- **org.Hs.eg.db** (Bioconductor annotation package shipped as an ARMT dependency, supporting Ensembl ID to gene Symbol transfer)

## Examples

```
devtools::install_github("Dulab2020/ARMT"); ARMT::run_app()  # use 'Data' page to build .gmt from symbol .csv, then 'Normalization&GSVA' for Ensembl->Symbol TPM conversion
```

## Evaluation signals

- The TPM output .csv has gene symbol row names while the input counts .csv had Ensembl ID row names, with sample column names preserved
- The generated .gmt file parses with GSEABase/GSVA's gene-set reader and its gene set names and member genes match the two-column .csv (or c2BroadSets) source entries
- Gene members of the .gmt overlap gene symbols present in the TPM matrix, so GSVA score calculation proceeds without dropping all sets
- TPM values are normalized and log2(TPM+1)-transformed before GSVA, per ARMT's stated pipeline
- Integrated table (clinical + GSVA + TPM + mutation) merges only on common samples after symbol-based alignment

## Limitations

- ARMT expects the input expression matrix to use Ensembl IDs; other identifier types are not documented as supported for the automatic transfer
- No software version numbers are stated for ARMT or its dependency versions beyond 'R >=4.0.2'
- The .csv used to build a .gmt file must strictly contain two columns (genesets, genes); deviations are not described as handled

## Evidence
