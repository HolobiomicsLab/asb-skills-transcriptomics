---
name: gene-set-variation-analysis-scoring
description: Use when you have a raw RNA-seq counts matrix (Ensembl IDs, .csv) and
  want gene-set-level activity scores (e.g. hypoxia/glycolysis) per sample to enable
  group comparisons, KM/Cox analysis or correlation of pathway activity.
when_to_use_negative:
- The goal is gene-level differential expression analysis — ARMT uses edgeR on the
  raw counts matrix for DEGs, not GSVA scores
- You only need single-gene characteristics (survival, correlation for one gene) and
  no gene-set/pathway activity summary
- The expression matrix is already a gene-symbol TPM table and cannot be re-run through
  ARMT's Ensembl-ID-based normalization path
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: ARMT
  role: Shiny GUI that performs TPM normalization, Ensembl-to-symbol conversion, and
    GSVA score computation; exports the GSVA score matrix as CSV
  repo: https://github.com/Dulab2020/ARMT
- name: GSVA
  role: Bioconductor package providing the gene set variation analysis method ARMT
    calls on the log2(TPM+1) matrix
- name: limma
  role: Used downstream on the exported GSVA score matrix for differential analysis
    between high/low groups, since GSVA scores are decimal values incompatible with
    edgeR's integer input
provenance:
  source_task_ids:
  - task_002
  - task_004
  - task_006
  source_papers:
  - doi: 10.1016/j.csbj.2021.08.009
    title: 'ARMT: An automatic RNA-seq data mining tool based on comprehensive and
      integrative analysis in cancer research'
  source_content_hash: sha256:894810e4f804d3812678d21996cfffaf9785e9467e868c299a345df12efa8c68
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
  - limma
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# gene-set-variation-analysis-scoring

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes per-sample gene set variation analysis (GSVA) scores for arbitrary gene sets from RNA-seq expression data, as implemented in ARMT, converting gene sets (e.g. 22-gene glycolysis and 14-gene hypoxia signatures) into continuous per-sample activity values that feed downstream differential, survival and correlation analyses.

## When to use

Use when you have a raw RNA-seq counts matrix (Ensembl IDs, .csv) and want gene-set-level activity scores (e.g. hypoxia/glycolysis) per sample to enable group comparisons, KM/Cox analysis or correlation of pathway activity.

## When NOT to use

- The goal is gene-level differential expression analysis — ARMT uses edgeR on the raw counts matrix for DEGs, not GSVA scores
- You only need single-gene characteristics (survival, correlation for one gene) and no gene-set/pathway activity summary
- The expression matrix is already a gene-symbol TPM table and cannot be re-run through ARMT's Ensembl-ID-based normalization path

## Inputs

- Gene expression counts matrix as .csv (row names = Ensembl gene IDs, column names = sample IDs)
- Gene set file: two-column .csv (geneset, gene) converted to a .gmt file in ARMT's 'Data' page
- Sample-type selection for TCGA data (normal, tumor, or all; default tumor)
- Optionally one of the 9 internal MSigDb gene sets instead of a custom .gmt

## Outputs

- GSVA score matrix exported as .csv (rows = sample IDs, columns = geneset names)
- Normalized log2(TPM + 1) TPM matrix indexed by gene symbols (also saved as .csv for integration)

## How to apply

Normalize the counts matrix to a TPM matrix and transform to log2(TPM + 1), which is the input ARMT's Normalization&GSVA module requires for GSVA; the counts matrix must be a .csv with row names as Ensembl gene IDs and columns as sample IDs, with Ensembl IDs converted to gene symbols during normalization. Define the gene sets as a .gmt file built in ARMT's 'Data' page from a two-column .csv (first column = geneset name, second column = genes), e.g. the 22-gene glycolysis signature (SLC2A1, HK1, HK2, HK3, GPI, PFKL, PFKM, PFKP, ALDOA, ALDOB, ALDOC, TPI1, GAPDH, PGK1, PGAM1, PGAM4, ENO1, ENO2, ENO3, PKLR, PKM, LDHA) and the 14-gene hypoxia signature (ALDOA, MIF, TUBB6, P4HA1, SLC2A1, PGAM1, ENO1, LDHA, CDKN3, TPI1, NDRG1, VEGFA, ACOT7, ADM). Run GSVA on the log2(TPM+1) matrix via the Normalization&GSVA page, selecting either one of the 9 built-in MSigDb gene sets or the custom .gmt file; for TCGA input choose normal, tumor or all samples (default: tumor). Export the resulting GSVA score matrix as a .csv with sample IDs as rows and geneset names as columns, which downstream ARMT pages (Integration&Analysis) consume for survival analysis, Cox regression, limma-based differential analysis of scores, and correlation analysis.

## Related tools

- **ARMT** (Shiny GUI that performs TPM normalization, Ensembl-to-symbol conversion, and GSVA score computation; exports the GSVA score matrix as CSV) — https://github.com/Dulab2020/ARMT
- **GSVA** (Bioconductor package providing the gene set variation analysis method ARMT calls on the log2(TPM+1) matrix)
- **limma** (Used downstream on the exported GSVA score matrix for differential analysis between high/low groups, since GSVA scores are decimal values incompatible with edgeR's integer input)

## Examples

```
devtools::install_github("Dulab2020/ARMT"); ARMT::run_app()
```

## Evaluation signals

- Output score matrix .csv has one row per sample and one column per geneset (e.g. hypoxia, glycolysis), as required by the Integration&Analysis differential-analysis input
- Input counts matrix carries Ensembl IDs while the exported TPM matrix carries gene symbols, confirming the normalization converted IDs correctly
- For TCGA input, tumor samples were used unless the user explicitly changed the normal/tumor/all default
- Reproduction check: differential analysis of the glycolysis GSVA score between hypoxia-high and hypoxia-low LUAD groups (top/bottom 30% hypoxia scores) reproduces the reported adjusted p-value of 2.45e-128
- The .gmt file contains exactly the expected genes per set: 22 genes for glycolysis and 14 for hypoxia

## Limitations

- No version numbers for ARMT or the GSVA package are stated in the article or README, so exact reproducibility across software versions is not documented
- The normalization/GSVA path expects an Ensembl-ID counts matrix; pre-normalized gene-symbol expression tables do not fit the documented input format
- GSVA scores are continuous decimal values, so edgeR cannot be applied to them downstream — limma must be used for differential analysis of scores

## Evidence
