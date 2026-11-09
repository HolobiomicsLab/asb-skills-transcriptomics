---
name: gsva-geneset-scoring
description: Use when you have a raw RNA-seq counts matrix (Ensembl IDs, e.g. TCGA-LUAD)
  and a curated gene set (e.g. a 14-gene hypoxia or 22-gene glycolysis signature)
  and need per-sample pathway/hallmark scores for downstream grouping, survival or
  differential analysis.
when_to_use_negative:
- The input is already a TPM matrix with gene Symbol row names — the Normalization&GSVA
  module expects a raw counts matrix with Ensembl IDs and would mis-handle pre-normalized
  symbol-level input.
- The goal is single-gene differential expression (DEG calling) — that should go through
  ARMT's differential analysis with edgeR on the counts matrix, not GSVA scoring.
- The gene set is undefined or you have no .csv/.gmt gene set and no interest in any
  of the 9 internal MSigDb gene sets — GSVA scoring cannot proceed without a gene
  set definition.
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: ARMT
  role: shiny-based GUI that normalizes counts to TPM, runs GSVA, and integrates the
    resulting score matrix with clinical/TPM/maf data for survival, Cox, differential
    and correlation analysis
  repo: https://github.com/Dulab2020/ARMT
- name: GSVA
  role: Bioconductor package invoked by ARMT's Normalization&GSVA module to compute
    the per-sample gene set variation scores
- name: GSVAdata
  role: Bioconductor companion data package installed with ARMT for gene set resources
provenance:
  source_task_ids:
  - task_001
  - task_005
  source_papers:
  - doi: 10.1016/j.csbj.2021.08.009
    title: 'ARMT: An automatic RNA-seq data mining tool based on comprehensive and
      integrative analysis in cancer research'
  source_content_hash: sha256:287e8d3a99b345ec588990a180b9af3b022b97f5d7c985588bd33f1a043bca07
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
  - R
  - survminer
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# gsva-geneset-scoring

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Compute gene set variation analysis (GSVA) scores per sample for custom or MSigDb gene sets on TCGA-style RNA-seq data, using ARMT's Normalization&GSVA module to transform a raw counts matrix into a log2(TPM + 1) matrix before scoring. Scores then feed survival, Cox, differential and correlation analyses in cancer research.

## When to use

Use when you have a raw RNA-seq counts matrix (Ensembl IDs, e.g. TCGA-LUAD) and a curated gene set (e.g. a 14-gene hypoxia or 22-gene glycolysis signature) and need per-sample pathway/hallmark scores for downstream grouping, survival or differential analysis.

## When NOT to use

- The input is already a TPM matrix with gene Symbol row names — the Normalization&GSVA module expects a raw counts matrix with Ensembl IDs and would mis-handle pre-normalized symbol-level input.
- The goal is single-gene differential expression (DEG calling) — that should go through ARMT's differential analysis with edgeR on the counts matrix, not GSVA scoring.
- The gene set is undefined or you have no .csv/.gmt gene set and no interest in any of the 9 internal MSigDb gene sets — GSVA scoring cannot proceed without a gene set definition.

## Inputs

- Raw gene expression counts matrix as .csv, row names = gene Ensembl ID, column names = sample ID (e.g. TCGA-LUAD transcriptome counts from GDC)
- Gene set definition as two-column .csv (geneset, genes) converted to a .gmt file, e.g. the 14-gene hypoxia signature (ALDOA, MIF, TUBB6, P4HA1, SLC2A1, PGAM1, ENO1, LDHA, CDKN3, TPI1, NDRG1, VEGFA, ACOT7, ADM) or the 22-gene glycolysis signature
- Optionally, ARMT-internal MSigDb gene sets (9 available) selected directly in the GUI
- Sample-type choice for TCGA input (normal, tumor or all; default tumor)

## Outputs

- TPM-normalized, log2(TPM + 1) transformed expression matrix with gene Symbol row names
- GSVA score matrix (.csv) with one score per sample per gene set, reusable in the Integration&Analysis page
- Downstream products: K-M survival curves with log-rank p-values, Cox regression results, and differential analysis of GSVA score (volcano-style screening by p-value, logFC, FDR)

## How to apply

First run ARMT's Normalization&GSVA module: it normalizes the counts matrix to TPM and transforms to log2(TPM + 1) before calling the GSVA package, converting Ensembl IDs to gene Symbols. Provide the gene set as a .gmt file built from a two-column .csv (geneset name, genes) via ARMT's Data page, or pick one of the 9 MSigDb gene sets available in ARMT's internal datasets; for TCGA input, choose normal, tumor or all samples (default: tumor). The output GSVA score matrix (.csv) is then integrated with clinical, TPM and mutation data through common samples in the Integration&Analysis page. Use the score as a continuous factor in survival analysis (samples dichotomized into high and low groups, e.g. top and bottom 50%) or as the difference factor in differential analysis of GSVA score, where numeric factors are split into high/low with an adjustable cutoff 't' (upper and lower t*100%). Rationale: the paper combines GSVA with molecular characteristic and prognostic analysis to reveal biological processes of cancer samples, e.g. showing that higher hypoxia and glycolysis scores associate with poorer survival in LUAD.

## Related tools

- **ARMT** (shiny-based GUI that normalizes counts to TPM, runs GSVA, and integrates the resulting score matrix with clinical/TPM/maf data for survival, Cox, differential and correlation analysis) — https://github.com/Dulab2020/ARMT
- **GSVA** (Bioconductor package invoked by ARMT's Normalization&GSVA module to compute the per-sample gene set variation scores)
- **GSVAdata** (Bioconductor companion data package installed with ARMT for gene set resources)

## Examples

```
if (!requireNamespace("devtools", quietly=TRUE)) install.packages("devtools"); devtools::install_github("Dulab2020/ARMT"); ARMT::run_app()  # use Data page to build .gmt, then Normalization&GSVA page to compute scores
```

## Evaluation signals

- The normalized matrix is TPM-based and log2(TPM + 1)-transformed before GSVA, matching the ARMT/GSVA convention used in both source tasks.
- A GSVA score exists for every sample in the counts matrix for each gene set (e.g. 14-gene hypoxia signature), exportable as .csv from the Normalization&GSVA page.
- Sample grouping by score follows the expected scheme: top/bottom 50% for survival analysis, or adjustable t*100% high/low cutoffs for differential analysis of GSVA score.
- Reproducing the paper's pipeline, higher hypoxia and glycolysis GSVA scores should show significantly poorer survival (log-rank p < 0.05) in the tested TCGA cohorts, consistent with the reported LUAD finding.
- Hypoxia-high vs hypoxia-low stratification via top/bottom 30% of the GSVA score distribution should reproduce the reported DEG/up-regulated counts (6684 and 1174) when followed by edgeR differential analysis.

## Limitations

- The counts matrix must carry gene Ensembl IDs as row names; the ID is transferred to Symbol during normalization, so symbol-keyed or non-standard inputs are not directly supported.
- No tool or ARMT version numbers are stated; only R >= 4.0.2 and automatic dependency installation are documented.
- For TCGA input, the sample subset processed is selectable (normal, tumor, all) but defaults to tumor — mismatched sample scope vs the clinical data will break the sample-wise integration.
- The paper notes that different differential analysis tools based on different statistical principles may lead to different results, so score-stratified downstream DEG results are tool-dependent.

## Evidence
