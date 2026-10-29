---
name: differential-gene-expression-analysis
description: Use when comparing gene expression between two groups of RNA-seq samples
  (e.g., high vs low GSVA-score strata or tumor vs normal) and you need DEGs filtered
  by p-value, FDR and logFC thresholds.
when_to_use_negative:
- Input is already a TPM matrix processed by ARMT's Normalization&GSVA page — for
  differential analysis the counts matrix must be entered
- Data are raw sequencing reads requiring alignment/quantification — ARMT performs
  only downstream statistical analysis of existing count matrices
- The goal is mutation or GSVA-score differential analysis rather than gene expression
  DEGs — ARMT exposes separate modes for those inputs
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: ARMT
  role: Shiny GUI that integrates clinical/GSVA/TPM data and performs differential
    analysis between user-defined sample groups
  repo: https://github.com/Dulab2020/ARMT
- name: edgeR
  role: Backend package ARMT employs to detect DEGs (TMM normalization) for high efficiency
- name: limma
  role: Cross-check DEG detection; edgeR and limma are expected to yield consistent
    results for verification
- name: GSVA
  role: Produces the hypoxia/glycolysis gene-set scores used as the continuous grouping
    factor for DEG comparison
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.csbj.2021.08.009
    title: 'ARMT: An automatic RNA-seq data mining tool based on comprehensive and
      integrative analysis in cancer research'
  source_content_hash: sha256:d34dbd202f75b64def1f62ca4932b95fb33769aee8325073b8709aebfb6d4e8f
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
  - edgeR
  - limma
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# differential-gene-expression-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Detect differentially expressed genes (DEGs) between sample groups (e.g., hypoxia-high vs hypoxia-low TCGA-LUAD tumors) using ARMT's differential analysis, which employs edgeR on a counts matrix, with limma as a concordant verification method.

## When to use

Use when comparing gene expression between two groups of RNA-seq samples (e.g., high vs low GSVA-score strata or tumor vs normal) and you need DEGs filtered by p-value, FDR and logFC thresholds.

## When NOT to use

- Input is already a TPM matrix processed by ARMT's Normalization&GSVA page — for differential analysis the counts matrix must be entered
- Data are raw sequencing reads requiring alignment/quantification — ARMT performs only downstream statistical analysis of existing count matrices
- The goal is mutation or GSVA-score differential analysis rather than gene expression DEGs — ARMT exposes separate modes for those inputs

## Inputs

- Gene counts matrix (.csv, rows = Ensembl IDs, columns = sample IDs)
- Integrated table merging clinical data, GSVA score, TPM expression and mutation data on common samples
- Group/difference factor column (continuous score or categorical values)
- Adjustable group cutoff parameter t (upper and lower t*100%)

## Outputs

- DEG table with p-value, logFC and FDR (adj.p), filterable and exportable as deg_results.csv
- Volcano plot of differential expression results
- DEG list for downstream GO/KEGG enrichment analysis

## How to apply

Prepare the raw counts matrix as a .csv with Ensembl IDs as row names and sample IDs as columns, and merge it with clinical and GSVA-score data through common samples in the ARMT Integration&Analysis page. Select a difference factor from the integrated table: if it is a continuous variable (e.g., hypoxia GSVA score), samples are split into high and low groups with an adjustable cutoff t (upper and lower t*100%); otherwise choose two values for experiment vs control groups. ARMT then runs edgeR (TMM normalization) for high efficiency to detect DEGs, with a limma run as an expected-consistent cross-check, since both are among the generally acknowledged R packages for RNA-seq differential expression. Screen the result by p-value, logFC and FDR(adj.p) — e.g., p < 0.05, -log10(FDR) > 2, logFC > 1, with a stringent up-regulated criterion of -log10(FDR) > 20 — and write the filtered table (e.g., deg_results.csv) for comparison against reported DEG counts.

## Related tools

- **ARMT** (Shiny GUI that integrates clinical/GSVA/TPM data and performs differential analysis between user-defined sample groups) — https://github.com/Dulab2020/ARMT
- **edgeR** (Backend package ARMT employs to detect DEGs (TMM normalization) for high efficiency)
- **limma** (Cross-check DEG detection; edgeR and limma are expected to yield consistent results for verification)
- **GSVA** (Produces the hypoxia/glycolysis gene-set scores used as the continuous grouping factor for DEG comparison)

## Examples

```
if (!requireNamespace("devtools", quietly=TRUE)) install.packages("devtools"); devtools::install_github("Dulab2020/ARMT"); ARMT::run_app()
```

## Evaluation signals

- Running edgeR and limma on the same hypoxia-high vs hypoxia-low TCGA-LUAD groups yields consistent DEG sets
- Filtered DEG count at p < 0.05, -log10(FDR) > 2, logFC > 1 matches the reported 6684 DEGs
- Stringent up-regulated list (p < 0.05, -log10(FDR) > 20, logFC > 1) matches the reported 1174 genes
- Counts matrix input uses Ensembl row IDs and sample-ID columns in .csv format per README Figure18
- Volcano plot renders in the ARMT main panel and p-value/logFC/FDR filters are applied without error

## Limitations

- Different differential analysis tools based on different statistical principles may lead to different results, so tool choice affects DEG sets
- No software version numbers for edgeR/limma/R packages are stated, complicating exact reproduction
- The difference-factor grouping cutoff t is adjustable, so group composition (and thus DEGs) depends on the chosen percentile

## Evidence
