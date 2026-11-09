---
name: clinical-survival-data-integration
description: Use when detected RNCs/modules from tumor expression data (e.g., 161
  TCGA ESCA tumors) must be associated with clinical survival outcomes, requiring
  a per-module Kaplan-Meier log-rank statistic.
when_to_use_negative:
- The cohort or module members lack matched clinical survival metadata (e.g., only
  the 11 normal ESCA samples), so no KM split/log-rank test can be computed
- The analysis goal is differential expression scoring (edgeR DEGs / combinp) rather
  than outcome association of already-detected circuits
- Modules come from the global runmodule(method='global', FDR=1e-5) case where the
  target is a single maximum-scoring circuit, not a per-RNC survival screen
tools:
- name: fRNC
  role: Provides the survival functions (survival.miR_rna / survival.km) for Kaplan-Meier
    curve computation and log-rank testing of RNC modules against clinical data
  repo: https://github.com/BioinformaticsSTU/fRNC
- name: R
  role: Runtime environment in which fRNC's survival and module-export functions are
    executed to build the per-RNC survival table
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.csbj.2023.03.035
    title: '<i>f</i> RNC: Uncovering the dynamic and condition-specific RBP-ncRNA
      circuits from multi-omics data'
  source_content_hash: sha256:04bfcc526d271b234c268e516b7399ea9ee1f7633f05c1dee15bacb41cfd6828
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/BioinformaticsSTU/fRNC
  repo_url: https://github.com/BioinformaticsSTU/fRNC
  tools:
  - R
  - edgeR
  - fRNC
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# clinical-survival-data-integration

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Links fRNC-detected RBP-ncRNA circuits (RNCs) to patient outcome by joining each module's member expression profiles with tumor clinical data and computing Kaplan-Meier survival curves with log-rank testing per RNC, as demonstrated on the TCGA ESCA case study.

## When to use

Use when detected RNCs/modules from tumor expression data (e.g., 161 TCGA ESCA tumors) must be associated with clinical survival outcomes, requiring a per-module Kaplan-Meier log-rank statistic.

## When NOT to use

- The cohort or module members lack matched clinical survival metadata (e.g., only the 11 normal ESCA samples), so no KM split/log-rank test can be computed
- The analysis goal is differential expression scoring (edgeR DEGs / combinp) rather than outcome association of already-detected circuits
- Modules come from the global runmodule(method='global', FDR=1e-5) case where the target is a single maximum-scoring circuit, not a per-RNC survival screen

## Inputs

- fRNC local-search RNC module list (output of runmodule(method='local', maxsize=15), exported via savelocalM)
- TCGA ESCA tumor expression profiles for module member miRNAs and mRNAs/lncRNAs (161 tumors)
- TCGA ESCA clinical data table with survival time/status per patient

## Outputs

- Per-RNC Kaplan-Meier survival curves with log-rank test statistics
- A per-RNC survival statistics table (esca_rnc_survival_table.csv) with one log-rank p-value per detected RNC

## How to apply

After enumerating locally maximum RNCs with runmodule(network, gene2weight, method='local', maxsize=15) seeded from every node, export each module with savelocalM(res.list_local). For each RNC, join the expression profiles of its member miRNAs/mRNAs/lncRNAs with the ESCA clinical follow-up table, restricting to tumor samples that carry survival metadata. Compute Kaplan-Meier curves with log-rank testing using the fRNC survival functions (survival.miR_rna / survival.km), splitting patients into groups by module member expression and recording the log-rank p-value per RNC. Assemble per-RNC survival statistics into a single table (e.g., esca_rnc_survival_table.csv) so every detected RNC row carries its own association test.

## Related tools

- **fRNC** (Provides the survival functions (survival.miR_rna / survival.km) for Kaplan-Meier curve computation and log-rank testing of RNC modules against clinical data) — https://github.com/BioinformaticsSTU/fRNC
- **R** (Runtime environment in which fRNC's survival and module-export functions are executed to build the per-RNC survival table)

## Evaluation signals

- The survival table contains exactly one row per detected RNC (e.g., 1761 ESCA-specific RNCs from the local search) with a non-missing log-rank statistic/p-value
- Every reported p-value lies in [0,1] and each KM analysis used only tumor samples with survival metadata
- Each RNC and its individual nodes are assessed (module-level and member-level KM curves), matching the reported case-study design
- The local search upstream of survival testing reproduces the reported set of 1761 ESCA-specific RNCs
- Output CSV (esca_rnc_survival_table.csv) is parseable with consistent per-RNC schema

## Limitations

- Log-rank p-values across 1761 RNCs are unadjusted in the reported workflow, so multiple-testing burden should be considered before declaring significant associations
- Results depend on the upstream local-search parameters (maxsize=15, high-stringency ENCORI interactions with CLIP-seq evidence >= 3); changing these alters which RNCs reach survival testing
- Only tumor samples with clinical follow-up can be used; the 11 normal ESCA samples contribute no survival information

## Evidence

- [results] Local search and survival validation design: [evidence span withheld — anchor status: not_found]
