---
name: rna-seq-gene-id-mapping
description: Use when the input expression matrix is a .csv counts file with Ensembl
  gene IDs as row names and TCGA-style sample IDs as columns, and downstream GSVA,
  integration, or correlation analysis requires gene-symbol-keyed TPM data.
when_to_use_negative:
- The expression matrix already uses gene symbols (e.g. the TPM matrix in Figure19
  format) — no Ensembl-to-symbol mapping is needed before integration
- You need gene-level differential expression (DEG) counts analysis with edgeR, which
  operates on the counts matrix, not the TPM/log2-transformed values
- 'The transcriptome is non-human: ARMT relies on Bioconductor human resources such
  as org.Hs.eg.db, so Ensembl-to-symbol mapping is not supported for other organisms
  in this tool'
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0092
tools:
- name: ARMT
  role: 'Performs the normalization pipeline: counts → TPM → log2(TPM+1), converting
    Ensembl IDs to gene symbols within the Normalization&GSVA GUI page'
  repo: https://github.com/Dulab2020/ARMT
- name: GSVA
  role: Consumes the gene-symbol log2(TPM+1) matrix together with the .gmt genesets
    to produce per-sample GSVA scores
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.csbj.2021.08.009
    title: 'ARMT: An automatic RNA-seq data mining tool based on comprehensive and
      integrative analysis in cancer research'
  source_content_hash: sha256:7279d0f8bae5bd2b605c779200019c4da0ca48ef740fca57fbbac33a739ce147
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# rna-seq-gene-id-mapping

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Converts a raw RNA-seq counts matrix with Ensembl gene IDs into a log2(TPM+1) matrix keyed by gene symbols, the format ARMT requires for downstream GSVA score calculation. This ID conversion is embedded in ARMT's Normalization&GSVA step so that counts matrices can be joined with genesets and clinical data on gene symbols.

## When to use

Use when the input expression matrix is a .csv counts file with Ensembl gene IDs as row names and TCGA-style sample IDs as columns, and downstream GSVA, integration, or correlation analysis requires gene-symbol-keyed TPM data.

## When NOT to use

- The expression matrix already uses gene symbols (e.g. the TPM matrix in Figure19 format) — no Ensembl-to-symbol mapping is needed before integration
- You need gene-level differential expression (DEG) counts analysis with edgeR, which operates on the counts matrix, not the TPM/log2-transformed values
- The transcriptome is non-human: ARMT relies on Bioconductor human resources such as org.Hs.eg.db, so Ensembl-to-symbol mapping is not supported for other organisms in this tool

## Inputs

- Gene expression counts matrix as .csv (row names = Ensembl gene IDs, column names = sample IDs)
- Geneset .csv with geneset and gene columns (convertible to .gmt via ARMT 'Data' page)
- Choice of TCGA sample subset: normal, tumor, or all (default tumor)

## Outputs

- TPM matrix transformed to log2(TPM + 1), saved as .csv with gene symbols as row names and sample IDs as columns
- GSVA score matrix (.csv, rows = samples, columns = geneset names) computed on the log2(TPM+1) matrix

## How to apply

Load the counts matrix (rows = Ensembl gene IDs, columns = sample IDs) into ARMT's 'Normalization&GSVA' page. ARMT automatically normalizes the counts to a TPM matrix, applies the log2(TPM + 1) transformation, and transfers Ensembl IDs to gene symbols during normalization, per the paper's methods. If the data are from TCGA, choose normal, tumor, or all samples for the normalization+GSVA step (default: tumor). The resulting gene-symbol TPM matrix (.csv) is the artifact that can be integrated with clinical, GSVA, and mutation data on common samples, and it is the direct input for GSVA with the .gmt genesets (e.g. hypoxia and glycolysis signatures from a two-column geneset .csv converted via ARMT's Data page).

## Related tools

- **ARMT** (Performs the normalization pipeline: counts → TPM → log2(TPM+1), converting Ensembl IDs to gene symbols within the Normalization&GSVA GUI page) — https://github.com/Dulab2020/ARMT
- **GSVA** (Consumes the gene-symbol log2(TPM+1) matrix together with the .gmt genesets to produce per-sample GSVA scores)

## Examples

```
devtools::install_github("Dulab2020/ARMT"); ARMT::run_app()
```

## Evaluation signals

- Row names of the output TPM matrix are gene symbols, not Ensembl IDs (e.g. no 'ENSG...' prefixes remain)
- Every output value equals log2(TPM + 1), i.e. non-negative TPM values transformed as specified in the ARMT methods
- Column names (sample IDs) are preserved unchanged from the input counts matrix
- GSVA run on the converted matrix with the hypoxia/glycolysis .gmt genesets reproduces the paper's LUAD case: glycolysis score difference between hypoxia-high and hypoxia-low groups at adj.p = 2.45*10^-128
- The normalized matrix merges with clinical and mutation data through common samples in ARMT's Integration step without ID-mismatch errors

## Limitations

- No software version numbers for the conversion tooling are stated (only R >= 4.0.2 and Bioconductor packages listed)
- The Ensembl-to-symbol transfer happens automatically inside normalization; no standalone mapping function or mapping-quality report is exposed, so ambiguous/one-to-many ID cases are not documented
- The README truncates before describing the GSVA score matrix output format in full

## Evidence
