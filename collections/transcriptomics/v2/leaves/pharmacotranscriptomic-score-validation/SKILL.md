---
name: pharmacotranscriptomic-score-validation
description: Use when a computed reversal/ranking score for drug perturbation signatures
  must be checked against an external efficacy ground truth (IC50) before it is trusted
  for in silico drug candidate selection.
when_to_use_negative:
- The disease context lacks normal samples or a disease-induced FM pattern (e.g.,
  rare diseases), so the reversal score itself cannot be computed for validation
- The goal is de novo candidate identification (e.g., ranking compounds for CRPC)
  rather than checking whether an existing score correlates with efficacy
- Compounds cannot be mapped between LINCS perturbation signatures and ChEMBL by name,
  leaving no paired score–IC50 observations
edam_topics:
- http://edamontology.org/topic_0208
tools:
- name: GSFM
  role: Framework whose RS-GSFM output score is the subject of the IC50 correlation
    validation
  repo: https://github.com/LXY1201/GSFM
- name: R 4.1.1
  role: Environment in which the RSGSFM–IC50 correlations are computed and the correlation
    CSV is written
- name: signatureSearch R package 1.4.6
  role: Provides the LINCS drug-induced expression profiles underlying the perturbation
    signatures whose scores are validated
- name: Jupyter Notebook
  role: Notebook interface used to run the GSFM pipeline steps feeding the validated
    score
provenance:
  source_task_ids:
  - task_005
  - task_003
  - task_008
  source_papers:
  - doi: 10.1016/j.apsb.2024.08.017
    title: 'GSFM: A genome-scale functional module transformation to represent drug
      efficacy for in silico drug discovery'
  source_content_hash: sha256:a3a039a2761d8c4036a06b5afbac0ab9ecb895d23bd4ab3d244e162584c7bbe2
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
  - Docker
  - GSFM
  - Jupyter Notebook
  - Python
  - R
  - R 4.1.1
  - dplyr
  - readr
  - signatureSearch
  - signatureSearch R package 1.4.6
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0208
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5333333333333333
n_claims: 15
---

# pharmacotranscriptomic-score-validation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Validates a pharmacotranscriptomic reversal score (RS-GSFM, and comparators RS-Gene/RS-Viper) against measured drug efficacy by correlating scores with ChEMBL v27 median IC50 values across MCF7 (BRCA), A549 (LUAD), and PC3 (CRPC) cell lines, establishing the score as a representative measure of drug efficacy.

## When to use

Use when a computed reversal/ranking score for drug perturbation signatures must be checked against an external efficacy ground truth (IC50) before it is trusted for in silico drug candidate selection.

## When NOT to use

- The disease context lacks normal samples or a disease-induced FM pattern (e.g., rare diseases), so the reversal score itself cannot be computed for validation
- The goal is de novo candidate identification (e.g., ranking compounds for CRPC) rather than checking whether an existing score correlates with efficacy
- Compounds cannot be mapped between LINCS perturbation signatures and ChEMBL by name, leaving no paired score–IC50 observations

## Inputs

- RSGSFM reversal scores (and optionally RS-Gene, RS-Viper comparators) per compound per cancer type (BRCA, LUAD, CRPC), from reversal_scores.csv
- ChEMBL database (version 27) compound IC50 values, median-summarized for duplicate entries, mapped to LINCS compounds by name
- Per-cell-line compound alignment keyed to MCF7 (BRCA), A549 (LUAD), PC3 (CRPC)

## Outputs

- rsgsfm_ic50_correlation.csv containing per-cancer-condition correlation statistics (r, direction, condition)
- Validation verdict that RSGSFM shows a positive correlation with IC50, supporting it as a representative measure of drug efficacy
- Optional ROC-based retrieval assessment discriminating effective (IC50 < 10 μmol/L) vs. ineffective (IC50 > 10 μmol/L) compounds

## How to apply

Load the computed RSGSFM scores and the median-summarized compound IC50 table from ChEMBL v27, mapping compounds between LINCS and ChEMBL by name. Align compounds per cell line (MCF7 for BRCA, A549 for LUAD, PC3 for CRPC), using the median IC50 to summarize duplicate compound entries. Compute the Pearson correlation r between RSGSFM and IC50 per cancer condition in R 4.1.1 (Eq. 17), and assemble r and direction per condition into rsgsfm_ic50_correlation.csv. Rationale: a positive correlation (expected r ~ 0.60–0.62, P < 1e-7) confirms RSGSFM — computed via the RS method adapted from RGES (Eqs. 12–16) on the GSFM active matrix — tracks IC50, a key measure of drug efficacy; the same IC50 annotation supports ROC-based retrieval with effective (IC50 < 10 μmol/L) vs. ineffective (IC50 > 10 μmol/L) grouping.

## Related tools

- **GSFM** (Framework whose RS-GSFM output score is the subject of the IC50 correlation validation) — https://github.com/LXY1201/GSFM
- **R 4.1.1** (Environment in which the RSGSFM–IC50 correlations are computed and the correlation CSV is written)
- **signatureSearch R package 1.4.6** (Provides the LINCS drug-induced expression profiles underlying the perturbation signatures whose scores are validated)
- **Jupyter Notebook** (Notebook interface used to run the GSFM pipeline steps feeding the validated score)

## Examples

```
Rscript -e 'r <- cor(df$RSGSFM, df$IC50); write.csv(data.frame(condition=c("BRCA","LUAD","CRPC"), r=r), "rsgsfm_ic50_correlation.csv")'
```

## Evaluation signals

- Per-condition correlation direction must be positive for RSGSFM vs. IC50 in all three conditions: BRCA (r = 0.61, P = 4.40e-22), LUAD (r = 0.62, P = 7.66e-11), CRPC (r = 0.60, P = 5.23e-08)
- Duplicate compound IC50 entries must be collapsed with the median before correlation, per the methods
- Correlation output table must be written to rsgsfm_ic50_correlation.csv with columns for r, direction, and cancer condition
- RSGSFM should additionally discriminate effective (IC50 < 10 μmol/L) from ineffective (IC50 > 10 μmol/L) compounds in ROC-based retrieval assessment (AUC/PR via pROC 1.17.0.1 / ROCR 1.0.11)
- RS-Gene and RS-Viper comparator scores should be evaluated under the same rank-based reversal scheme so the FM-level score's relative performance is interpretable

## Limitations

- Selecting functional modules from different sources can lead to slight variations in results; disease-related modules must be well aligned with the disease under study
- The method may not be applicable to certain rare diseases when normal samples are unavailable
- Additional experimental and clinical studies are needed to validate and elucidate the mechanisms of identified candidates in each individual cancer

## Evidence
