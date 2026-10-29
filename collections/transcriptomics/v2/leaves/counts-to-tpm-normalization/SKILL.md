---
name: counts-to-tpm-normalization
description: Use when the input is a raw counts matrix (.csv, rows = Ensembl gene
  IDs, columns = sample IDs) that must be normalized to TPM and log2(TPM+1) before
  GSVA, correlation analysis, or integration with clinical/mutation data.
when_to_use_negative:
- The input matrix is already a TPM or log2(TPM+1) expression matrix with gene symbol
  row names (ARMT expects a raw counts matrix with Ensembl IDs for this step)
- The goal is differential gene expression analysis starting from raw counts that
  should be handled directly with edgeR on the counts matrix rather than TPM-normalized
  data
- Data are not gene expression counts (e.g. mutation .maf files or GSVA score matrices),
  which take other ARMT input routes
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: ARMT
  role: Performs the TPM normalization, log2(TPM+1) transformation, and Ensembl-to-symbol
    ID conversion automatically via its Normalization&GSVA GUI page
  repo: https://github.com/Dulab2020/ARMT
- name: GSVA
  role: Downstream consumer of the normalized log2(TPM+1) matrix; the normalization
    step exists to satisfy GSVA's within-sample processing requirement
  repo: https://github.com/Dulab2020/ARMT
provenance:
  source_task_ids:
  - task_006
  - task_002
  source_papers:
  - doi: 10.1016/j.csbj.2021.08.009
    title: 'ARMT: An automatic RNA-seq data mining tool based on comprehensive and
      integrative analysis in cancer research'
  source_content_hash: sha256:ffe8682d4010f84c62cb31d445b61bbff922fe91f3f1ada734087c0dc2c6f314
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
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# counts-to-tpm-normalization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Normalize a raw RNA-seq counts matrix to TPM and transform to log2(TPM+1), converting Ensembl gene IDs to gene symbols, as the standard preprocessing step in ARMT that prepares expression data for GSVA and downstream gene-set analysis.

## When to use

Use when the input is a raw counts matrix (.csv, rows = Ensembl gene IDs, columns = sample IDs) that must be normalized to TPM and log2(TPM+1) before GSVA, correlation analysis, or integration with clinical/mutation data.

## When NOT to use

- The input matrix is already a TPM or log2(TPM+1) expression matrix with gene symbol row names (ARMT expects a raw counts matrix with Ensembl IDs for this step)
- The goal is differential gene expression analysis starting from raw counts that should be handled directly with edgeR on the counts matrix rather than TPM-normalized data
- Data are not gene expression counts (e.g. mutation .maf files or GSVA score matrices), which take other ARMT input routes

## Inputs

- Counts matrix .csv file (row names = gene Ensembl ID, column names = sample ID)
- Optional: TCGA sample-type selection (normal, tumor, or all)

## Outputs

- TPM matrix .csv file (row names = gene symbol ID, column names = sample ID)
- log2(TPM + 1) transformed expression matrix used for GSVA and integration

## How to apply

In ARMT's Normalization&GSVA page, supply the counts matrix as a .csv with gene Ensembl IDs as row names and sample IDs as column names; ARMT normalizes it to a TPM matrix and transforms to log2(TPM + 1). During normalization the Ensembl IDs are automatically converted to gene symbols, which is required for GSVA and for integration with mutation/clinical data. For TCGA input, choose whether to process normal, tumor, or all samples (default: tumor). The resulting log2(TPM+1) matrix, with gene symbol row names and sample ID column names, is the input for GSVA score calculation and can be merged with other data through common samples; it also feeds differential analysis of gene expression. Rationale: within-sample TPM normalization is required by ARMT before GSVA processing, and raw integer counts are unsuitable for that step.

## Related tools

- **ARMT** (Performs the TPM normalization, log2(TPM+1) transformation, and Ensembl-to-symbol ID conversion automatically via its Normalization&GSVA GUI page) — https://github.com/Dulab2020/ARMT
- **GSVA** (Downstream consumer of the normalized log2(TPM+1) matrix; the normalization step exists to satisfy GSVA's within-sample processing requirement) — https://github.com/Dulab2020/ARMT

## Examples

```
devtools::install_github("Dulab2020/ARMT"); ARMT::run_app()  # upload counts .csv (Ensembl ID rows) in Normalization&GSVA page
```

## Evaluation signals

- Output matrix row names are gene symbols (not Ensembl IDs) and column names are sample IDs
- Output values are log2(TPM + 1), i.e. non-negative before log transform and comparable across samples within-gene
- The normalized matrix loads into the Normalization&GSVA page and produces a GSVA score matrix (rows = sample IDs, columns = geneset names) as a .csv
- Downstream GSVA differential analysis reproduces reported results, e.g. glycolysis GSVA difference between hypoxia-high/-low LUAD groups at adj.p = 2.45*10^-128
- For TCGA input, the chosen sample subset (normal/tumor/all, default tumor) is reflected in the output columns

## Limitations

- ARMT's normalization expects the counts matrix input to have Ensembl gene IDs as row names; other ID types are not stated as supported
- No software version numbers or normalization implementation details are stated for ARMT's TPM calculation
- Counts matrix and TPM matrix must be supplied as .csv files; other formats require conversion

## Evidence
