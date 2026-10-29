---
name: tcga-clinical-transcriptome-data-integration
description: Use when TCGA counts matrices and clinical data must be combined with
  per-sample GSVA scores (e.g., hypoxia/glycolysis signatures) to test associations
  with patient survival via K-M or Cox analysis.
when_to_use_negative:
- Input is already a processed feature table with no per-sample clinical survival
  time/status to merge
- The hypothesis concerns differential expression rather than clinical association
  of gene-set scores (use ARMT's differential analysis module instead)
- Data are not TCGA and lack the sample-ID format ARMT uses to join clinical and expression
  data
edam_operation: http://edamontology.org/operation_2403
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3361
tools:
- name: ARMT
  role: Provides built-in TCGA clinical data for 33 cancer types, TPM normalization,
    .gmt gene set creation, and the GUI for data integration and survival/Cox analysis
  repo: https://github.com/Dulab2020/ARMT
- name: GSVA
  role: Computes per-sample gene set variation analysis scores (e.g., hypoxia, glycolysis
    signatures) that are integrated with clinical data
- name: survival
  role: R package used to run Kaplan-Meier survival analysis and Cox proportional
    hazards regression on the integrated clinical-GSVA data
- name: survminer
  role: Visualizes K-M curves for the dichotomized GSVA score groups
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.csbj.2021.08.009
    title: 'ARMT: An automatic RNA-seq data mining tool based on comprehensive and
      integrative analysis in cancer research'
  source_content_hash: sha256:c9881f73195b89cb2f7285dd52f8b0a1909e9ebe9ce542fe90803e596eb146a6
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
  - survival
  - survminer
  edam_operation: http://edamontology.org/operation_2403
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3361
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# tcga-clinical-transcriptome-data-integration

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Merges TCGA clinical data (survival time/status) with GSVA scores, TPM expression and mutation profiles on common samples using ARMT's built-in TCGA clinical datasets, enabling downstream survival and Cox regression analysis of gene-set scores.

## When to use

Use when TCGA counts matrices and clinical data must be combined with per-sample GSVA scores (e.g., hypoxia/glycolysis signatures) to test associations with patient survival via K-M or Cox analysis.

## When NOT to use

- Input is already a processed feature table with no per-sample clinical survival time/status to merge
- The hypothesis concerns differential expression rather than clinical association of gene-set scores (use ARMT's differential analysis module instead)
- Data are not TCGA and lack the sample-ID format ARMT uses to join clinical and expression data

## Inputs

- TCGA gene expression counts matrix (.csv, Ensembl ID rows x sample ID columns)
- TCGA clinical data (survival time, status) from ARMT internal datasets
- Gene set definitions (.gmt file, e.g., 14-gene hypoxia and 22-gene glycolysis signatures)
- GSVA score matrix (.csv produced by ARMT's Normalization&GSVA page)
- TPM matrix (.csv, gene symbol rows x sample ID columns)
- Mutation data in standard .maf format (optional)

## Outputs

- Integrated sample-matched table of clinical, GSVA, TPM and mutation data
- Kaplan-Meier survival curves (survminer) per continuous factor (high vs low 50%)
- Table of log-rank p-values per cancer type
- Single-variable or multi-variable Cox regression results

## How to apply

Download TCGA transcriptome counts matrices and clinical data via ARMT's built-in TCGA clinical datasets covering 33 cancer types, then normalize counts to TPM and apply log2(TPM + 1) in the Normalization&GSVA page. Compute GSVA scores per gene set (.gmt files built from the 14-gene hypoxia and 22-gene glycolysis signatures) using the GSVA package. In the Integration&Analysis page, merge clinical, GSVA score, TPM and mutation data (.csv/.maf inputs) on common samples, optionally splitting cohorts by a 'Group Column' for pan-cancer analysis. For survival analysis, select the survival time and status columns; continuous factors such as GSVA scores are dichotomized by top vs bottom 50% before K-M analysis (survival/survminer) with log-rank p-values, and multiple factors can be combined in Cox proportional hazards regression.

## Related tools

- **ARMT** (Provides built-in TCGA clinical data for 33 cancer types, TPM normalization, .gmt gene set creation, and the GUI for data integration and survival/Cox analysis) — https://github.com/Dulab2020/ARMT
- **GSVA** (Computes per-sample gene set variation analysis scores (e.g., hypoxia, glycolysis signatures) that are integrated with clinical data)
- **survival** (R package used to run Kaplan-Meier survival analysis and Cox proportional hazards regression on the integrated clinical-GSVA data)
- **survminer** (Visualizes K-M curves for the dichotomized GSVA score groups)

## Examples

```
ARMT::run_app()
```

## Evaluation signals

- Integrated table contains only common samples present in both clinical and expression/GSVA inputs (row counts match after merge)
- High glycolysis and high hypoxia groups (top 50% GSVA score) show significantly poorer survival with log-rank p < 0.05, reproducing the LUAD/BLCA/CESC/HNSC finding
- Continuous survival factors yield exactly two groups at the top/bottom 50% split with no misclassified or NA survival times
- Cox regression identifies hypoxia as an independent factor while glycolysis is not, matching the paper's reported result
- TPM matrix row IDs are gene symbols (converted from Ensembl) and values correspond to log2(TPM + 1) scale

## Limitations

- Clinical data are limited to TCGA cohorts built into ARMT (33 cancer types); non-TCGA studies require user-supplied data in the same format
- Dichotomizing continuous GSVA scores at the 50% cutoff is arbitrary and may lose statistical power versus continuous modeling
- No software or ARMT version numbers are stated, so exact reproducibility of results may vary with package versions

## Evidence
