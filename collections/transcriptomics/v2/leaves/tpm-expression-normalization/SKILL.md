---
name: tpm-expression-normalization
description: Use when a raw gene counts matrix (Ensembl IDs, sample columns) must
  be converted to TPM/log2(TPM+1) before computing GSVA scores or integrating expression
  with TCGA clinical data.
when_to_use_negative:
- Input is already a TPM matrix (row names are gene symbols) — ARMT expects raw counts
  with Ensembl IDs for normalization and differential analysis
- The analysis goal is raw-count differential expression with edgeR, which uses the
  counts matrix rather than the TPM-normalized matrix
- Non-TCGA expression data lacking Ensembl IDs cannot be ID-transferred through the
  ARMT normalization step as described
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3512
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0085
tools:
- name: ARMT
  role: Automatically normalizes the expression counts matrix to TPM, transfers Ensembl
    IDs to symbols, and log2-transforms for GSVA via its Normalization&GSVA page
  repo: https://github.com/Dulab2020/ARMT
- name: GSVA
  role: Consumes the log2(TPM + 1) normalized matrix to compute per-sample gene-set
    variation analysis scores
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.csbj.2021.08.009
    title: 'ARMT: An automatic RNA-seq data mining tool based on comprehensive and
      integrative analysis in cancer research'
  source_content_hash: sha256:92154708b599ebe838b37e9a5ee137e14203b338ce609aaf24239570b7a9bfd8
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
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3512
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0085
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# tpm-expression-normalization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Normalize a raw RNA-seq counts matrix to a TPM matrix and transform to log2(TPM + 1) using ARMT, producing a gene-symbol-level expression matrix suitable as input for GSVA scoring and downstream integration with clinical and mutation data.

## When to use

Use when a raw gene counts matrix (Ensembl IDs, sample columns) must be converted to TPM/log2(TPM+1) before computing GSVA scores or integrating expression with TCGA clinical data.

## When NOT to use

- Input is already a TPM matrix (row names are gene symbols) — ARMT expects raw counts with Ensembl IDs for normalization and differential analysis
- The analysis goal is raw-count differential expression with edgeR, which uses the counts matrix rather than the TPM-normalized matrix
- Non-TCGA expression data lacking Ensembl IDs cannot be ID-transferred through the ARMT normalization step as described

## Inputs

- Gene counts matrix as .csv with row names = gene Ensembl IDs and column names = sample IDs
- TCGA-LUAD (or other TCGA) RNA-seq counts matrix

## Outputs

- TPM-normalized expression matrix as .csv with row names = gene symbol IDs and column names = sample IDs
- log2(TPM + 1) transformed matrix used as GSVA input

## How to apply

Load the counts matrix as a .csv file whose row names are gene Ensembl IDs and whose column names are sample IDs, then let ARMT automatically normalize it to a TPM matrix and transform to log2(TPM + 1) as required for GSVA input. During normalization ARMT transfers Ensembl IDs to gene symbols, since the downstream TPM matrix must carry symbol-level row names to be integrable with clinical, GSVA, and mutation data. If the input is from TCGA, choose tumor, normal, or all samples for normalization (default: tumor). The resulting TPM/log2(TPM+1) matrix is then used to compute per-sample GSVA scores for gene sets such as the 14-gene hypoxia and 22-gene glycolysis signatures, and can be merged with TCGA-LUAD clinical data (survival time, status, gender, stage, age) through common sample IDs.

## Related tools

- **ARMT** (Automatically normalizes the expression counts matrix to TPM, transfers Ensembl IDs to symbols, and log2-transforms for GSVA via its Normalization&GSVA page) — https://github.com/Dulab2020/ARMT
- **GSVA** (Consumes the log2(TPM + 1) normalized matrix to compute per-sample gene-set variation analysis scores)

## Examples

```
devtools::install_github("Dulab2020/ARMT"); ARMT::run_app()  # use 'Normalization&GSVA' page to normalize counts to TPM/log2(TPM+1)
```

## Evaluation signals

- Output TPM matrix is a .csv with gene symbol row names and sample-ID column names, ready for integration with clinical/GSVA/mutation data
- Values are log2(TPM + 1) transformed, matching the stated GSVA input requirement
- If TCGA input, the tumor/normal/all sample selection matches the intended analysis (default: tumor)
- Downstream GSVA scores (e.g., 14-gene hypoxia, 22-gene glycolysis signatures) can be merged with TCGA clinical data by common sample IDs without ID mismatches

## Limitations

- Input must be an Ensembl-ID counts matrix in .csv format; other ID types require prior conversion
- No software version numbers for ARMT or the normalization step are stated in the article or README
- The README truncates before fully documenting the GSVA score matrix output format

## Evidence
