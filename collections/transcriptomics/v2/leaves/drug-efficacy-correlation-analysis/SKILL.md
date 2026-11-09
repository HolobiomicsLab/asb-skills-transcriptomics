---
name: drug-efficacy-correlation-analysis
description: Use when you have per-compound reversal scores (e.g., RS_GSFM) for LINCS
  drug perturbations and independent efficacy measurements (IC50 from ChEMBL) and
  need to test score–efficacy agreement per cell line.
when_to_use_negative:
- You have no external efficacy measurements (e.g., no ChEMBL IC50 data) for the ranked
  compounds — the correlation cannot be computed.
- You are evaluating raw gene-level reversal scores (RS-Gene) or network-level scores
  (RS-Viper) rather than the GSFM-based RSGSFM; use the corresponding score-specific
  validation instead.
- The disease of interest lacks a matched LINCS-perturbed cell line or normal samples,
  since GSFM/RSGSFM requires an aligned disease-induced FM pattern per cell line.
edam_operation: http://edamontology.org/operation_3465
edam_topics:
- http://edamontology.org/topic_3336
- http://edamontology.org/topic_0203
tools:
- name: GSFM
  role: Framework whose RSGSFM reversal score (comparing drug-perturbation FM patterns
    to disease-induced FM patterns) is the variable correlated with IC50
  repo: https://github.com/LXY1201/GSFM
- name: R 4.1.1
  role: Environment in which the RSGSFM–IC50 correlations per cancer condition are
    computed and the CSV output is assembled
- name: ChEMBL database (version 27)
  role: Source of compound IC50 efficacy data, median-summarized for duplicate compound
    entries
- name: signatureSearch R package 1.4.6
  role: Downloads the LINCS drug-induced gene expression profiles underlying the RSGSFM
    scores being validated
- name: Jupyter Notebook
  role: Runs the GSFM dockerized notebooks (e.g., BRCA_GSFM.ipynb) that produce the
    RSGSFM scores fed into the correlation analysis
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.apsb.2024.08.017
    title: 'GSFM: A genome-scale functional module transformation to represent drug
      efficacy for in silico drug discovery'
  source_content_hash: sha256:a20c8079a6ee28a2ba250f800574cef135eda47ee2bd07bb213d1975a3b17b87
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/LXY1201/GSFM
  repo_url: https://github.com/LXY1201/GSFM
  tools:
  - ChEMBL database (version 27)
  - GSFM
  - Jupyter Notebook
  - R
  - R 4.1.1
  - signatureSearch R package 1.4.6
  edam_operation: http://edamontology.org/operation_3465
  edam_topics:
  - http://edamontology.org/topic_3336
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# drug-efficacy-correlation-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

This skill quantifies whether a drug-reversal score (here, RSGSFM computed from the GSFM functional-module transformation) predicts drug efficacy by correlating it with half-maximal inhibitory concentration (IC50) per cancer cell-line condition. It validates the reversal score as a representative measure of drug efficacy across BRCA, LUAD, and CRPC retrieval settings.

## When to use

Use when you have per-compound reversal scores (e.g., RS_GSFM) for LINCS drug perturbations and independent efficacy measurements (IC50 from ChEMBL) and need to test score–efficacy agreement per cell line.

## When NOT to use

- You have no external efficacy measurements (e.g., no ChEMBL IC50 data) for the ranked compounds — the correlation cannot be computed.
- You are evaluating raw gene-level reversal scores (RS-Gene) or network-level scores (RS-Viper) rather than the GSFM-based RSGSFM; use the corresponding score-specific validation instead.
- The disease of interest lacks a matched LINCS-perturbed cell line or normal samples, since GSFM/RSGSFM requires an aligned disease-induced FM pattern per cell line.

## Inputs

- RSGSFM reversal scores per compound (from the RS-GSFM calculation step, STEP 3 drug discovery)
- Compound IC50 efficacy table from ChEMBL database (version 27), median-summarized for duplicate entries
- Compound name mapping between LINCS drug perturbation profiles and ChEMBL entries
- Cell-line assignment per cancer condition: MCF7 (BRCA), A549 (LUAD), PC3 (CRPC)

## Outputs

- rsgsfm_ic50_correlation.csv containing correlation statistics (r, direction) for each cancer condition
- Per-condition correlation coefficients with p-values (reported: BRCA r = 0.61, P = 4.40e-22; LUAD r = 0.62, P = 7.66e-11; CRPC r = 0.60, P = 5.23e-08)

## How to apply

Load the computed RSGSFM scores from the RS-GSFM calculation step and the median-summarized compound IC50 table from ChEMBL (version 27), mapping compounds between LINCS and ChEMBL by drug name. Align compounds per cell line (MCF7 for BRCA, A549 for LUAD, PC3 for CRPC), using the median IC50 to summarize duplicate compound entries for the same drug. Compute the Pearson correlation r between RSGSFM and IC50 for each cancer condition in R 4.1.1, following Eq. (17) where r is the correlation between RSGSFM and IC50. Assemble the correlation statistics (r, direction, per-cancer condition) into a single table and write it to rsgsfm_ic50_correlation.csv. A correct application reproduces the reported strong positive correlations, confirming RSGSFM as a representative measure of drug efficacy.

## Related tools

- **GSFM** (Framework whose RSGSFM reversal score (comparing drug-perturbation FM patterns to disease-induced FM patterns) is the variable correlated with IC50) — https://github.com/LXY1201/GSFM
- **R 4.1.1** (Environment in which the RSGSFM–IC50 correlations per cancer condition are computed and the CSV output is assembled)
- **ChEMBL database (version 27)** (Source of compound IC50 efficacy data, median-summarized for duplicate compound entries)
- **signatureSearch R package 1.4.6** (Downloads the LINCS drug-induced gene expression profiles underlying the RSGSFM scores being validated)
- **Jupyter Notebook** (Runs the GSFM dockerized notebooks (e.g., BRCA_GSFM.ipynb) that produce the RSGSFM scores fed into the correlation analysis)

## Examples

```
cor.test(rsgsfm_scores, median_ic50, method = "pearson")  # per cell line: MCF7 (BRCA), A549 (LUAD), PC3 (CRPC) in R 4.1.1
```

## Evaluation signals

- Reproduced correlations are strong and positive across all three conditions: BRCA r = 0.61 (P = 4.40e-22), LUAD r = 0.62 (P = 7.66e-11), CRPC r = 0.60 (P = 5.23e-08).
- rsgsfm_ic50_correlation.csv contains one row per cancer condition (BRCA/MCF7, LUAD/A549, CRPC/PC3) with r and direction columns populated.
- Duplicate ChEMBL compound entries are summarized by median IC50 before correlation; check that each compound–cell-line pair appears exactly once.
- Compounds are successfully mapped between LINCS and ChEMBL by name; unmapped compounds are dropped rather than silently coerced.
- The correlation direction is positive in every condition — a negative or near-zero r indicates a misaligned score table or wrong cell-line pairing.

## Limitations

- Correlation supports RSGSFM as a representative measure of drug efficacy but was established only for the three tested cancer cell-line conditions (MCF7, A549, PC3); other contexts are unvalidated.
- Selecting functional modules from different sources can lead to slight variations in results, which may shift the correlation values.
- The method may not be applicable to certain rare diseases when normal samples (needed to define the disease-induced FM pattern) are unavailable.
- Additional experimental and clinical studies are essential to validate and further elucidate the mechanisms of identified candidates in each individual cancer.

## Evidence

- [intro] Positive correlation between RSGSFM and IC50 as efficacy measure: "Notably, a positive correlation was observed between RSGSFM and half-maximal inhibitory concentration (IC50), a key measure of drug efficacy."
- [methods] ChEMBL v27 as IC50 source: "Efficacy data for each compound (IC50) were sourced from the ChEMBL database (version 27)"
- [methods] R as analysis environment: "computational analyses were conducted in R version 4.1.1 and Python 3.9.7"
- [abstract] RSGSFM validated against drug efficacy: "we found a positive correlation between RSGSFM and drug efficacy, suggesting that RSGSFM could serve as representative measure of drug efficacy"
- [readme] README step producing RS-GSFM scores: "step3.1:Calculation of RS-GSFM"
