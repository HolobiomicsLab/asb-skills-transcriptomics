---
name: tcga-rnaseq-counts-normalization
description: Use when a TCGA-LUAD (or other TCGA) raw gene-level counts matrix (.csv
  with Ensembl ID rows and sample ID columns) must be converted to a TPM/log2(TPM
  + 1) matrix before GSVA, DEG, survival, or correlation analysis in ARMT.
when_to_use_negative:
- Input is already a TPM or log2(TPM + 1) expression matrix keyed by gene symbol —
  re-normalizing counts-style data would be inappropriate; such matrices are for the
  Integration page, not re-normalization
- Analysis requires count-based statistical models that need raw counts (e.g., DESeq2's
  negative binomial framework) rather than TPM-based comparisons
- The input matrix does not use Ensembl IDs as row names, which ARMT requires for
  the Ensembl-to-symbol ID transfer during normalization
edam_operation: http://edamontology.org/operation_3232
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: ARMT
  role: Automatically normalizes the counts matrix to TPM and transforms to log2(TPM
    + 1) via its 'Normalization&GSVA' page, transferring Ensembl IDs to gene symbols
  repo: https://github.com/Dulab2020/ARMT
- name: R
  role: Runtime environment (>= 4.0.2) in which ARMT is installed from GitHub with
    devtools/remotes and run through the shiny GUI
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.csbj.2021.08.009
    title: 'ARMT: An automatic RNA-seq data mining tool based on comprehensive and
      integrative analysis in cancer research'
  source_content_hash: sha256:2424da70b35e0609dcf8610b7fdea16cf6aaeb329f796cdca514f6c17dc8a922
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
  - R
  edam_operation: http://edamontology.org/operation_3232
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# tcga-rnaseq-counts-normalization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Normalize a raw RNA-seq counts matrix to a TPM matrix and transform to log2(TPM + 1) as prescribed by the ARMT workflow, producing the expression matrix required for downstream GSVA scoring and differential analysis in TCGA cancer studies.

## When to use

Use when a TCGA-LUAD (or other TCGA) raw gene-level counts matrix (.csv with Ensembl ID rows and sample ID columns) must be converted to a TPM/log2(TPM + 1) matrix before GSVA, DEG, survival, or correlation analysis in ARMT.

## When NOT to use

- Input is already a TPM or log2(TPM + 1) expression matrix keyed by gene symbol — re-normalizing counts-style data would be inappropriate; such matrices are for the Integration page, not re-normalization
- Analysis requires count-based statistical models that need raw counts (e.g., DESeq2's negative binomial framework) rather than TPM-based comparisons
- The input matrix does not use Ensembl IDs as row names, which ARMT requires for the Ensembl-to-symbol ID transfer during normalization

## Inputs

- Raw gene-level RNA-seq counts matrix as .csv (rows = gene Ensembl IDs, columns = sample IDs, e.g. TCGA-LUAD transcriptome counts from GDC)
- Choice of sample subset for TCGA data: normal, tumor, or all samples (default: tumor)

## Outputs

- TPM-normalized expression matrix as .csv (rows = gene symbol IDs, columns = sample IDs)
- log2(TPM + 1)-transformed matrix used for downstream GSVA and differential analysis

## How to apply

Provide the raw counts matrix as a .csv whose row names are gene Ensembl IDs and column names are sample IDs, as required by ARMT's 'Normalization&GSVA' page. ARMT automatically normalizes the counts matrix to a TPM matrix and transforms it to log2(TPM + 1); for TCGA inputs you can choose to process normal, tumor, or all samples, with tumor as the default. During normalization, Ensembl IDs are transferred to gene symbols, yielding a TPM matrix keyed by gene symbol that can be integrated with clinical, GSVA-score, and mutation (.maf) data through common samples. Rationale: ARMT's downstream analyses (GSVA score calculation, differential analysis via edgeR, correlation analysis) operate on this log2(TPM + 1) convention, so skipping or deviating from the normalization step breaks consistency with the paper's reported DEG counts (e.g., 6684 DEGs at p < 0.05, -log10(FDR) > 2, logFC > 1 for hypoxia-high vs hypoxia-low LUAD).

## Related tools

- **ARMT** (Automatically normalizes the counts matrix to TPM and transforms to log2(TPM + 1) via its 'Normalization&GSVA' page, transferring Ensembl IDs to gene symbols) — https://github.com/Dulab2020/ARMT
- **R** (Runtime environment (>= 4.0.2) in which ARMT is installed from GitHub with devtools/remotes and run through the shiny GUI)

## Examples

```
if (!requireNamespace("devtools", quietly = TRUE)) install.packages("devtools"); devtools::install_github("Dulab2020/ARMT"); ARMT::run_app()
```

## Evaluation signals

- Output TPM matrix is a .csv with gene symbol row names (not Ensembl IDs) and sample ID column names, confirming the ID transfer occurred
- Expression values are non-negative TPM values, and downstream analysis values follow log2(TPM + 1) scaling per the ARMT/GSVA convention
- Sample set of the normalized matrix matches the chosen subset (tumor by default for TCGA) and can be merged with clinical and GSVA data on common samples without ID mismatches
- Downstream DEG analysis on the normalized data reproduces comparable DEG counts to the paper's reported values (6684 DEGs and 1174 up-regulated genes under the stated thresholds)

## Limitations

- No software version numbers for the normalization step or ARMT itself are stated; dependencies are only pinned at R >= 4.0.2
- The input counts matrix must use Ensembl IDs; matrices keyed by other identifier types require prior conversion
- For TCGA input, the default processes tumor samples only; users must explicitly change the setting to include normal or all samples

## Evidence
