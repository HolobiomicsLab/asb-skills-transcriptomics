---
name: rnaseq-dropout-characterization
description: Use when a cells-by-variants VAF matrix from scRNA-seq variant calling
  contains NA/missing entries and you need to quantify how much missingness is attributable
  to type I drop-out before lineage reconstruction or quality assessment.
when_to_use_negative:
- Input is not a single-cell VAF/variant matrix — e.g. bulk somatic variant calls
  or gene expression counts with no per-variant missingness structure.
- The goal is to impute or correct drop-outs via lineage-aware modeling (use SClineager's
  run_sclineager instead) rather than to characterize their prevalence.
- Missingness in the matrix encodes true homozygous reference calls rather than unobserved/low-coverage
  positions; counting all NAs would then overstate type I drop-out.
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0625
tools:
- name: SClineager
  role: Source of the toy dataset and VAF matrix structure (res_data$mutations_mat_obs);
    the drop-out characterization motivates and contextualizes SClineager's Bayesian
    lineage inference.
  repo: github.com/inmybrain/SClineager
- name: R
  role: Analysis environment used to load the RData file, compute column/row/overall
    missingness proportions, and write dropout_summary.csv.
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.celrep.2020.108589
    title: Overcoming Expressional Drop-outs in Lineage Reconstruction from Single-Cell
      RNA-Sequencing Data
  source_content_hash: sha256:3a5bc1c7125dd5eec1b0ab48b6543c46c2a4395253ff4ca53dd42b2aede40cf1
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/inmybrain/SClineager
  tools:
  - R
  - SClineager
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# rnaseq-dropout-characterization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantifies type I expressional drop-out (missing variant observations from low expression / shallow coverage) in single-cell variant allele frequency matrices, per variant, per cell, and overall. This characterizes the magnitude of missingness that motivates coverage-aware lineage inference methods such as SClineager.

## When to use

Use when a cells-by-variants VAF matrix from scRNA-seq variant calling contains NA/missing entries and you need to quantify how much missingness is attributable to type I drop-out before lineage reconstruction or quality assessment.

## When NOT to use

- Input is not a single-cell VAF/variant matrix — e.g. bulk somatic variant calls or gene expression counts with no per-variant missingness structure.
- The goal is to impute or correct drop-outs via lineage-aware modeling (use SClineager's run_sclineager instead) rather than to characterize their prevalence.
- Missingness in the matrix encodes true homozygous reference calls rather than unobserved/low-coverage positions; counting all NAs would then overstate type I drop-out.

## Inputs

- toy dataset ./data/toy_data.RData containing res_data with the raw cells-by-variants VAF matrix res_data$mutations_mat_obs
- cells-by-variants matrix of variant allele frequencies with missing (NA) entries encoding drop-outs

## Outputs

- dropout_summary.csv with per-variant drop-out proportion, per-cell drop-out proportion, overall drop-out proportion, and cells-by-variants matrix dimensions

## How to apply

Load the toy dataset with R's load() function from ./data/toy_data.RData and access the raw variant allele frequency matrix res_data$mutations_mat_obs, recording its cells-by-variants dimensions. Compute the proportion of missing (NA) observations per variant (column-wise), per cell (row-wise), and overall, treating these missings as type I drop-outs caused by low expression or shallow sequencing coverage. Write the three drop-out proportions together with the matrix dimensions to dropout_summary.csv. Rationale: single-cell variant calling inherently suffers from low coverage on many positions in many cells and allelic bias, and the paper reports up to 20% of variants can be missing due to low expression in at least one cell, with the exact proportion depending on the sequencing technique.

## Related tools

- **SClineager** (Source of the toy dataset and VAF matrix structure (res_data$mutations_mat_obs); the drop-out characterization motivates and contextualizes SClineager's Bayesian lineage inference.) — github.com/inmybrain/SClineager
- **R** (Analysis environment used to load the RData file, compute column/row/overall missingness proportions, and write dropout_summary.csv.)

## Examples

```
Rscript -e 'load("./data/toy_data.RData"); m <- res_data$mutations_mat_obs; cat(dim(m)); prop_na_var <- colMeans(is.na(m)); prop_na_cell <- rowMeans(is.na(m)); write.csv(data.frame(prop_na_var), "dropout_summary.csv")'
```

## Evaluation signals

- dropout_summary.csv exists and contains drop-out proportions per variant, per cell, and overall alongside the recorded cells-by-variants dimensions.
- Per-variant proportions lie in [0, 1] and the overall proportion is consistent with the paper's reported range (up to ~20% of variants missing in at least one cell).
- Sum of missing observations implied by per-variant proportions equals that implied by per-cell proportions and by the overall proportion (invariant: same total NA count).
- Recorded matrix dimensions match dim(res_data$mutations_mat_obs) from the loaded toy data.
- NA entries counted correspond to low-expression/shallow-coverage positions (coverage below thresholds like coverage_cutoff = 3) rather than filtered artifacts.

## Limitations

- The exact drop-out proportion depends on the sequencing technique used, so absolute values are dataset- and technology-specific.
- Drop-outs here capture low expression / shallow coverage; the paper distinguishes them from allelic bias due to true monoallelic expression or sampling bias, which this counting step does not disentangle.
- Based on the toy dataset from the SClineager package; results on real datasets may differ from the reported ~20% figure.

## Evidence

- [results] Up to 20% of variants in single cells are missing due to low expression: [evidence span withheld — anchor status: not_found]
