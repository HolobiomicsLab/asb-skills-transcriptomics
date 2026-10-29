---
name: circrna-regulator-candidate-validation
description: Use when a list of predicted circRNA regulators (e.g., CRIT NMF cluster
  output) must be tested for over-representation of circRNA-binding RBPs from CLIP
  databases (POSTAR3, ENCORI) via one-sided Fisher's exact tests.
when_to_use_negative:
- The goal is to predict or cluster circRNA regulators from omics features (NMF feature
  collection / cluster identify), rather than to validate an already-defined candidate
  list.
- No external circRNA-binding RBP reference list (POSTAR3/ENCORI) is available, so
  no 2x2 contingency table can be constructed.
- The question is functional validation (e.g., whether RBP knockdown changes global
  circRNA expression in shRNA/siRNA RNA-seq datasets), not statistical enrichment
  against CLIP databases.
edam_operation: http://edamontology.org/operation_3438
edam_topics:
- http://edamontology.org/topic_0099
- http://edamontology.org/topic_2640
tools:
- name: CRIT
  role: Source of the candidate regulator list (NMF cluster enriched in gold regulators)
    and the RBP_analysis.R validation script
  repo: https://github.com/BioinformaticsSTU/CRIT
- name: R
  role: Runtime for the RBP_analysis.R validation script; stats::fisher.test performs
    the one-sided Fisher's exact tests on the 2x2 tables
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.omtn.2022.10.015
    title: 'CRIT: Identifying RNA-binding protein regulator in circRNA life cycle
      via non-negative matrix factorization'
  source_content_hash: sha256:7da089bab624639755422365c7db9fb437918bbe4f8f7880f73ae962b082e4d3
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/BioinformaticsSTU/CRIT
  repo_url: https://github.com/BioinformaticsSTU/CRIT
  tools:
  - CRIT
  - R
  edam_operation: http://edamontology.org/operation_3438
  edam_topics:
  - http://edamontology.org/topic_0099
  - http://edamontology.org/topic_2640
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# circRNA-regulator-candidate-validation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reproduces the CLIP-database validation step of the CRIT pipeline: one-sided Fisher's exact tests in R that test whether CRIT's NMF-derived candidate circRNA regulators are over-represented among known circRNA-binding RBPs from POSTAR3 (112 RBPs) and ENCORI (132 down to 27 RBPs stringency sub-lists), against both a 1,344-RBP background and the total gene set.

## When to use

Use when a list of predicted circRNA regulators (e.g., CRIT NMF cluster output) must be tested for over-representation of circRNA-binding RBPs from CLIP databases (POSTAR3, ENCORI) via one-sided Fisher's exact tests.

## When NOT to use

- The goal is to predict or cluster circRNA regulators from omics features (NMF feature collection / cluster identify), rather than to validate an already-defined candidate list.
- No external circRNA-binding RBP reference list (POSTAR3/ENCORI) is available, so no 2x2 contingency table can be constructed.
- The question is functional validation (e.g., whether RBP knockdown changes global circRNA expression in shRNA/siRNA RNA-seq datasets), not statistical enrichment against CLIP databases.

## Inputs

- CRIT candidate regulator list (genes from the NMF cluster enriched in gold regulators)
- RBP_list.RData — background list of 1,344 candidate RBPs
- POSTAR3 CLIP-derived circRNA-binding RBP list (112 RBPs)
- ENCORI circRNA-binding RBP sub-lists at five stringency levels (132 down to 27 RBPs)

## Outputs

- 2x2 overlap contingency tables for each binding list x background combination
- Table of one-sided Fisher's exact test p-values (per CLIP database and per background)

## How to apply

Load the CRIT candidate regulator list — the genes from the NMF cluster enriched in gold regulators — together with the 1,344-RBP background stored in RBP_list.RData (used by the RBP_analysis.R validation script). Retrieve the POSTAR3 CLIP-derived circRNA-binding RBP list (112 RBPs) and the five ENCORI stringency sub-lists (132 decreasing to 27 RBPs). For each binding list and each background (1,344 candidate RBPs; all genes), construct 2x2 overlap contingency tables, then run one-sided Fisher's exact tests with R's stats::fisher.test to test for over-representation. Assemble the computed p-values into a table and compare them against the reported values (POSTAR3: 7.7e-5 vs RBPs and 7.8e-30 vs total genes; ENCORI stringency >=5: 3.3e-8 and 5.8e-25). The run is validated when each computed Fisher exact p-value matches its reported counterpart within an order of magnitude and the enrichment direction (significant over-representation of known circRNA-binding RBPs among CRIT candidates) is reproduced.

## Related tools

- **CRIT** (Source of the candidate regulator list (NMF cluster enriched in gold regulators) and the RBP_analysis.R validation script) — https://github.com/BioinformaticsSTU/CRIT
- **R** (Runtime for the RBP_analysis.R validation script; stats::fisher.test performs the one-sided Fisher's exact tests on the 2x2 tables)

## Examples

```
k <- length(intersect(cand, postar3_112)); m <- length(cand); stats::fisher.test(matrix(c(k, m-k, 112-k, 1344-m-112+k), 2), alternative="greater")
```

## Evaluation signals

- POSTAR3 overlap test against the 1,344-RBP background must yield p = 7.7e-5 (within an order of magnitude)
- POSTAR3 overlap test against total genes must yield p = 7.8e-30 (within an order of magnitude)
- ENCORI highest-stringency (>=5) tests must yield p = 3.3e-8 (vs 1,344 RBPs) and p = 5.8e-25 (vs total genes), each within an order of magnitude
- Enrichment direction must be over-representation (odds ratio > 1) of known circRNA-binding RBPs among CRIT candidates, not depletion
- Contingency-table marginals must match the reference list sizes: 112 POSTAR3 RBPs, ENCORI sub-lists of 132 down to 27 RBPs, and the 1,344-RBP background

## Limitations

- The acceptance criterion is agreement within one order of magnitude of the reported p-values, not exact numerical reproduction.
- Significance depends strongly on the chosen background (1,344 RBPs vs all genes): the same POSTAR3 overlap gives p = 7.7e-5 versus p = 7.8e-30, so the background must always be reported alongside any p-value.
- ENCORI sub-lists shrink from 132 to 27 RBPs as stringency rises, so high-stringency tests rest on very small gene sets.
- This skill validates consistency with CLIP binding evidence only; it does not demonstrate functional regulation, which the paper addresses separately via shRNA/siRNA knockdown RNA-seq analysis.

## Evidence

- [abstract] Analysis of circRNA-RBP interactions using two large cross-linking immunoprecipitation (CLIP) databases, we validated the consistency between CRIT prediction and the CLIP experiments.: "Analysis of circRNA-RBP interactions using two large cross-linking immunoprecipitation (CLIP) databases, we validated the consistency between CRIT prediction and the CLIP experiments."
