---
name: log-rank-survival-testing-of-gene-modules
description: Use when fRNC modules/RNCs (e.g., from runmodule local search on TCGA
  ESCA data) must be stratified by clinical outcome, requiring per-module Kaplan-Meier
  curves and log-rank p-values from TCGA survival annotations.
when_to_use_negative:
- The analysis has no clinical follow-up data — e.g., the BRCA protein-level case
  (case 2) run for global RNC detection with FDR=1e-20 without survival annotation
  has nothing for the log-rank test to stratify
- Modules were detected from non time-to-event data (e.g., Alzheimer's-style expression-only
  comparisons), where Kaplan-Meier/log-rank testing is not meaningful
- Only a single global maximum RNC (runmodule(method='global', FDR=1e-5)) was requested
  and no per-module survival characterization is needed
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_3053
tools:
- name: fRNC
  role: Provides both the local-search RNC detection whose modules are tested (runmodule,
    savelocalM) and the Kaplan-Meier/log-rank survival functions (survival.miR_rna,
    survival.km) applied to module members and clinical data
  repo: https://github.com/BioinformaticsSTU/fRNC
- name: edgeR
  role: Performs the differential expression analysis (DEGs on miRNA, mRNA, lncRNA)
    that produces the logFC/PValue node scores feeding the modules under survival
    testing
- name: R
  role: Runtime environment in which the fRNC package and its survival-testing functions
    are executed
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.csbj.2023.03.035
    title: '<i>f</i> RNC: Uncovering the dynamic and condition-specific RBP-ncRNA
      circuits from multi-omics data'
  source_content_hash: sha256:c1b219b2f44ac12f30290138a770ee506b64a4423e5e80c28d1dce7b320f222a
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
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_3053
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# log-rank-survival-testing-of-gene-modules

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Assesses clinical relevance of detected RBP-ncRNA circuits (RNCs) by joining each module's member expression profiles with cancer clinical data and computing Kaplan-Meier survival curves with log-rank testing, as demonstrated for 1761 ESCA-specific RNCs found by fRNC local search.

## When to use

Use when fRNC modules/RNCs (e.g., from runmodule local search on TCGA ESCA data) must be stratified by clinical outcome, requiring per-module Kaplan-Meier curves and log-rank p-values from TCGA survival annotations.

## When NOT to use

- The analysis has no clinical follow-up data — e.g., the BRCA protein-level case (case 2) run for global RNC detection with FDR=1e-20 without survival annotation has nothing for the log-rank test to stratify
- Modules were detected from non time-to-event data (e.g., Alzheimer's-style expression-only comparisons), where Kaplan-Meier/log-rank testing is not meaningful
- Only a single global maximum RNC (runmodule(method='global', FDR=1e-5)) was requested and no per-module survival characterization is needed

## Inputs

- Detected RNC module list from fRNC runmodule(method='local', maxsize=15) seeded over all nodes, as exported by savelocalM
- ESCA tumor/normal miRNA-seq and mRNA-seq count matrices (161 tumors, 11 normals from TCGA) used for edgeR DEGs
- Gene-to-weight table from combinp(dataNo[,c('type','logFC','PValue')], islog=TRUE)
- ENCORI RBP-ncRNA interaction network (transcription type, human, high stringency: CLIP evidence >= 3)
- TCGA ESCA clinical data with survival time and vital/censoring status

## Outputs

- Kaplan-Meier survival curves for each RNC and its individual member nodes
- Per-RNC log-rank test statistic and p-value table (esca_rnc_survival_table.csv)

## How to apply

First enumerate the locally maximum scoring RNCs by running fRNC's greedy local search seeded from every network node, e.g. runmodule(network, gene2weight, method='local', maxsize=15), on the ENCORI high-stringency network (CLIP experimental evidence >= 3) built with interStringency(type='transcription', spec='hg', stringency='high'), and export each module with savelocalM. For each detected RNC, extract the module member expression profiles from the tumor samples (161 ESCA tumors) and join them with the TCGA clinical survival data (time-to-event and censoring status). Use the fRNC survival functions (survival.miR_rna / survival.km) to compute Kaplan-Meier curves per RNC and its individual nodes, obtaining a log-rank statistic and p-value that quantify how strongly the module separates patient survival groups. Assemble the per-RNC statistics into a single table and write it out (e.g., esca_rnc_survival_table.csv) so survival associations can be ranked and inspected alongside module membership.

## Related tools

- **fRNC** (Provides both the local-search RNC detection whose modules are tested (runmodule, savelocalM) and the Kaplan-Meier/log-rank survival functions (survival.miR_rna, survival.km) applied to module members and clinical data) — https://github.com/BioinformaticsSTU/fRNC
- **edgeR** (Performs the differential expression analysis (DEGs on miRNA, mRNA, lncRNA) that produces the logFC/PValue node scores feeding the modules under survival testing)
- **R** (Runtime environment in which the fRNC package and its survival-testing functions are executed)

## Evaluation signals

- The local search over all nodes reproduces the reported 1761 ESCA-specific RNCs before survival testing
- The survival table contains a Kaplan-Meier log-rank statistic/p-value for every detected RNC row (no missing rows)
- All recorded log-rank p-values fall within [0, 1] and the table columns include module membership plus survival statistics
- Each RNC's KM curve is computed from module members joined to the 161 ESCA tumor samples' clinical records (sample counts consistent with available survival data)
- Output is a single assembled CSV (esca_rnc_survival_table.csv) rather than scattered per-module results

## Limitations

- The public README example only demonstrates local search from a single seed node (seletN = c('MIMAT0000089')); seeding from every node to reach 1761 RNCs is a heavier computation not directly shown in the README
- Network coverage is restricted by the high-stringency ENCORI filter (CLIP experimental evidence >= 3), so RNCs — and hence survival tests — only cover interactions with strong CLIP support
- The fRNC survival functions (survival.miR_rna / survival.km) are not documented in the README example, so their exact call signatures must be confirmed from the package reference manual in the docs directory

## Evidence
