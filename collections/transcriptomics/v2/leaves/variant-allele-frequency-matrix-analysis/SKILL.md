---
name: variant-allele-frequency-matrix-analysis
description: Use when a cells-by-variants VAF matrix from scRNA-seq variant calling
  contains missing entries, and the goal is to measure type I expressional drop-out
  (low expression / shallow coverage) before lineage reconstruction.
when_to_use_negative:
- Input is already cleaned/filtered mutation data with no missing VAF entries (e.g.
  after read_sclineager with coverage_cutoff filtering), where drop-out rates are
  no longer measurable
- The goal is lineage inference itself rather than drop-out characterization; use
  run_sclineager / sclineager_internal instead
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_3170
tools:
- name: SClineager
  role: Source of the R package and toy dataset (./data/toy_data.RData) whose res_data$mutations_mat_obs
    VAF matrix is analyzed for drop-out; the measured drop-out motivates its Bayesian
    imputation approach
  repo: https://github.com/inmybrain/SClineager
- name: R
  role: Environment (ver 4.0.3 in the paper) used to load the data, compute missingness
    proportions, and write dropout_summary.csv
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.celrep.2020.108589
    title: Overcoming Expressional Drop-outs in Lineage Reconstruction from Single-Cell
      RNA-Sequencing Data
  source_content_hash: sha256:fee000fea74248f8778aac1c9c8ef6d6317c30608784db9d801313adea5b9482
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/inmybrain/SClineager
  repo_url: https://github.com/inmybrain/SClineager
  tools:
  - R
  - SClineager
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# variant-allele-frequency-matrix-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantifies expressional drop-out in single-cell genetic variant data by computing the proportion of missing variant allele frequency (VAF) observations per variant, per cell, and overall from a cells-by-variants mutation matrix, as described in the SClineager workflow (Lu et al., Cell Reports 2021).

## When to use

Use when a cells-by-variants VAF matrix from scRNA-seq variant calling contains missing entries, and the goal is to measure type I expressional drop-out (low expression / shallow coverage) before lineage reconstruction.

## When NOT to use

- Input is already cleaned/filtered mutation data with no missing VAF entries (e.g. after read_sclineager with coverage_cutoff filtering), where drop-out rates are no longer measurable
- The goal is lineage inference itself rather than drop-out characterization; use run_sclineager / sclineager_internal instead

## Inputs

- RData file containing res_data with the raw variant allele frequency matrix mutations_mat_obs (cells x variants)
- coverage matrix from SClineager preprocessing (same cells and variants as mutations_mat)

## Outputs

- dropout_summary.csv with per-variant, per-cell, and overall drop-out proportions plus matrix dimensions
- Cells-by-variants dimension record of mutations_mat_obs

## How to apply

Load the toy or real dataset into R (e.g. with load('./data/toy_data.RData') from the SClineager package data) and access the raw variant allele frequency matrix res_data$mutations_mat_obs, recording its cells-by-variants dimensions. Compute the fraction of missing observations for each variant (column-wise), for each cell (row-wise), and across the entire matrix; these missing entries characterize type I drop-outs caused by low expression or shallow sequencing coverage. Write the drop-out proportions and matrix dimensions to dropout_summary.csv for reporting. The published analysis found that up to 20% of variants in single cells are missing due to low expression in at least one cell, with the exact proportion depending on the sequencing technique used, so per-technique comparison is meaningful. This quantification motivates the downstream SClineager Bayesian model, which borrows information from related cells to overcome expressional drop-outs rather than treating missing VAFs as informative zeros.

## Related tools

- **SClineager** (Source of the R package and toy dataset (./data/toy_data.RData) whose res_data$mutations_mat_obs VAF matrix is analyzed for drop-out; the measured drop-out motivates its Bayesian imputation approach) — https://github.com/inmybrain/SClineager
- **R** (Environment (ver 4.0.3 in the paper) used to load the data, compute missingness proportions, and write dropout_summary.csv)

## Examples

```
load('./data/toy_data.RData'); m <- res_data$mutations_mat_obs; prop_missing <- mean(is.na(m)); write.csv(dim(m), 'dropout_summary.csv')
```

## Evaluation signals

- dropout_summary.csv exists and contains per-variant, per-cell, and overall drop-out proportions plus the recorded cells-by-variants dimensions of mutations_mat_obs
- Overall drop-out proportion falls within the reported range of up to ~20% missing variant observations due to low expression in at least one cell
- Sum of per-variant missing proportions divided by matrix dimensions equals the reported overall drop-out proportion (internal consistency)
- Reported matrix dimensions match dim(res_data$mutations_mat_obs) in the loaded RData object

## Limitations

- The measured drop-out proportion depends on the sequencing technique used, so absolute values are not comparable across platforms without caution
- Drop-out here captures low expression / shallow coverage; allelic bias (true monoallelic expression or sampling bias) is a separate issue not quantified by missingness alone
- Based on a toy dataset (toy_data.RData); real datasets may show different drop-out magnitudes

## Evidence

- [results] Type I drop-out prevalence: [evidence span withheld — anchor status: not_found]
- [intro] Low coverage as inherent issue: [evidence span withheld — anchor status: not_found]
