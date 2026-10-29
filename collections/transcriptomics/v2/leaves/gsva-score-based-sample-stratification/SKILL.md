---
name: gsva-score-based-sample-stratification
description: Use when a continuous process signature (e.g., hypoxia or glycolysis
  gene set) must be turned into patient sample groups — e.g., to compare mutation
  burdens or survival between hypoxia-high and hypoxia-low TCGA-LUAD tumors.
when_to_use_negative:
- The factor of interest is already a discrete clinical variable (e.g., stage, sex)
  — no GSVA stratification is needed; group directly.
- You need gene-level differential expression rather than pathway-level sample scoring
  — use ARMT's edgeR DEG workflow instead.
edam_operation: http://edamontology.org/operation_3221
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: ARMT
  role: Performs TPM normalization, GSVA score computation, sample grouping by score
    cutoff, and exports the score matrix via its shiny GUI
  repo: https://github.com/Dulab2020/ARMT
- name: GSVA
  role: Underlying method computing sample-level gene-set enrichment scores used for
    stratification
- name: limma
  role: Differential analysis of the decimal-valued GSVA scores between high/low strata
provenance:
  source_task_ids:
  - task_003
  - task_002
  source_papers:
  - doi: 10.1016/j.csbj.2021.08.009
    title: 'ARMT: An automatic RNA-seq data mining tool based on comprehensive and
      integrative analysis in cancer research'
  source_content_hash: sha256:4c712763115e1469ecc624ec616dffb21563080d76b65a3abcaccd46c6cf7f7c
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
  - R
  - limma
  edam_operation: http://edamontology.org/operation_3221
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 14
---

# GSVA-score-based sample stratification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Stratifies transcriptome samples into high/low groups of a biological process (e.g., hypoxia) by ranking sample-level GSVA scores computed from a curated gene-set signature in ARMT, enabling downstream differential mutation, survival, and Cox analyses.

## When to use

Use when a continuous process signature (e.g., hypoxia or glycolysis gene set) must be turned into patient sample groups — e.g., to compare mutation burdens or survival between hypoxia-high and hypoxia-low TCGA-LUAD tumors.

## When NOT to use

- The factor of interest is already a discrete clinical variable (e.g., stage, sex) — no GSVA stratification is needed; group directly.
- You need gene-level differential expression rather than pathway-level sample scoring — use ARMT's edgeR DEG workflow instead.

## Inputs

- Raw gene expression counts matrix (.csv, Ensembl ID rows x sample ID columns)
- Gene-set signature as .csv (two columns: geneset name, genes) converted to .gmt
- Optional TCGA clinical data built into ARMT for 33 cancer types

## Outputs

- Per-sample GSVA score matrix (.csv, sample IDs x geneset names)
- High/low sample group assignments based on score percentile cutoff (e.g., top/bottom 30%)
- Differential GSVA score table between groups (limma, p-value/logFC/FDR)

## How to apply

Normalize the TCGA-LUAD counts matrix to TPM transformed to log2(TPM + 1), as required for ARMT's within-sample GSVA processing. Build a .gmt file from a two-column .csv (geneset column, gene column) containing the signature — e.g., the 22-gene glycolysis set (SLC2A1, HK1, HK2, LDHA, ...) or the 14-gene hypoxia set (ALDOA, MIF, VEGFA, ADM, ...) — and compute GSVA scores for every sample via the Normalization&GSVA page of ARMT's shiny GUI. Rank samples by the process GSVA score and split into high and low strata; in the paper's case study the top and bottom 30% of hypoxia GSVA scores defined hypoxia-high and hypoxia-low groups (the differential-analysis 't' cutoff parameter is adjustable in ARMT). Export the GSVA score matrix as a .csv (sample IDs as rows, geneset names as columns) and feed it into the Integration&Analysis page, where it merges with clinical, TPM, and .maf mutation data through common samples for downstream tests. Differential analysis of GSVA scores between strata uses limma rather than edgeR because scores are decimal values incompatible with edgeR's integer count input.

## Related tools

- **ARMT** (Performs TPM normalization, GSVA score computation, sample grouping by score cutoff, and exports the score matrix via its shiny GUI) — https://github.com/Dulab2020/ARMT
- **GSVA** (Underlying method computing sample-level gene-set enrichment scores used for stratification)
- **limma** (Differential analysis of the decimal-valued GSVA scores between high/low strata)

## Examples

```
ARMT::run_app()  # use Normalization&GSVA page with log2(TPM+1) matrix and hypoxia/glycolysis .gmt, then group samples at t=30%
```

## Evaluation signals

- Differential analysis of glycolysis GSVA score between hypoxia-high and hypoxia-low groups reproduces the reported adjusted p-value of 2.45e-128.
- GSVA score matrix .csv has sample IDs as rows and geneset names as columns.
- Group sizes match the chosen percentile cutoff (e.g., top and bottom 30% of hypoxia GSVA scores).
- Survival analysis on the stratified groups recovers the finding that higher hypoxia score is significantly associated with poorer LUAD survival.
- TPM matrix values are log2(TPM + 1), confirming the correct normalization step ran before GSVA.

## Limitations

- GSVA scores are decimal values, so edgeR (integer counts) cannot be used for between-group score comparison; limma must be substituted.
- The group cutoff 't' (upper and lower t*100%) is a user-adjustable parameter, so different cutoff choices change strata composition and results.
- Counts matrices must have Ensembl IDs that ARMT converts to symbols during normalization; non-TCGA data must be reshaped to the same .csv format.

## Evidence

- [methods] hypoxia-high and hypoxia-low grouping at top/bottom 30%: [evidence span withheld — anchor status: not_found]
- [methods] limma used because GSVA scores are decimals: [evidence span withheld — anchor status: not_found]
