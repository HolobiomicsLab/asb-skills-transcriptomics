---
name: mcri-index-computation-per-sample
description: Use when assessing whether knocking down a candidate circRNA regulator
  (RBP) alters global circRNA expression, given paired knockdown/control RNA-seq datasets
  with raw FASTQ files available from GEO/ENCODE.
when_to_use_negative:
- When the goal is identifying candidate circRNA regulators de novo from cancer cohorts
  — that is CRIT's NMF-based feature pipeline (ρm, Zp, Zs, Zg, C value), not this
  validation step
- When input data are already processed circRNA count matrices without raw FASTQ —
  CIRI2 and find_circ must be run from raw alignments per dataset
- When only a single circRNA detection tool can be run and dual-tool concordance is
  required for the claim
edam_operation: http://edamontology.org/operation_0289
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: CIRI2
  role: circRNA detection algorithm run on raw alignments of each knockdown/control
    RNA-seq dataset, one of the two tools whose concordance validates a hit
- name: find_circ
  role: second, complementary circRNA detection algorithm used to require concordance
    with CIRI2 calls
- name: R
  role: implementation environment for computing per-sample mCRI and running the group
    t tests between knockdown and control conditions
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.omtn.2022.10.015
    title: 'CRIT: Identifying RNA-binding protein regulator in circRNA life cycle
      via non-negative matrix factorization'
  source_content_hash: sha256:42e6471627b666048b87015e7cb046ccff10eaa7ff9b88d2cea81ec66f6c763e
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/BioinformaticsSTU/CRIT
  tools:
  - CIRI2
  - CRIT
  - R
  - find_circ
  edam_operation: http://edamontology.org/operation_0289
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# mcri-index-computation-per-sample

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes the normalized modified circRNA index (mCRI) per sample from shRNA/siRNA knockdown RNA-seq data — quantifying global circRNA abundance relative to linear RNA expression — and tests whether an RBP knockdown shifts global circRNA expression via group t tests between knockdown and control conditions.

## When to use

Use when assessing whether knocking down a candidate circRNA regulator (RBP) alters global circRNA expression, given paired knockdown/control RNA-seq datasets with raw FASTQ files available from GEO/ENCODE.

## When NOT to use

- When the goal is identifying candidate circRNA regulators de novo from cancer cohorts — that is CRIT's NMF-based feature pipeline (ρm, Zp, Zs, Zg, C value), not this validation step
- When input data are already processed circRNA count matrices without raw FASTQ — CIRI2 and find_circ must be run from raw alignments per dataset
- When only a single circRNA detection tool can be run and dual-tool concordance is required for the claim

## Inputs

- Raw FASTQ files of shRNA/siRNA knockdown RNA-seq datasets (from GEO/ENCODE), one or more datasets per candidate RBP
- Knockdown/control sample pairing information (e.g., Table S5 of the CRIT paper listing the 27 datasets and 22 candidate RBPs)
- Reference genome/annotation for circRNA detection from raw alignments
- Positive control (QKI) and negative control (SIGLEC15) knockdown datasets

## Outputs

- Per-sample mCRI table (normalized modified circRNA index per knockdown/control sample, per circRNA-detecting tool)
- Significance summary of group t tests per RBP (knockdown vs control mCRI)
- Dual-tool concordance list of RBPs supported by both CIRI2 and find_circ (expected: SRSF5, HNRNPA2B1, HNRNPF, KHSRP, PSIP1)
- mcri_validation_results.csv

## How to apply

Download raw FASTQ files for each knockdown/control pair (e.g., the 27 datasets from Table S5 covering 22 candidate RBPs, plus QKI as positive control and SIGLEC15 as negative control). Detect circRNAs independently in each dataset with two complementary algorithms, CIRI2 and find_circ, run from the raw alignments, requiring concordance between the two tools' calls. For each sample compute the normalized mCRI, quantifying global circRNA abundance relative to linear RNA expression. Run group t tests comparing per-sample mCRI between knockdown and control conditions for each RBP; a hit requires a statistically significant mCRI change (supported by at least one circRNA detector) and the strongest candidates are those supported by both CIRI2 and find_circ. Write the per-sample mCRI table and significance summary to mcri_validation_results.csv.

## Related tools

- **CIRI2** (circRNA detection algorithm run on raw alignments of each knockdown/control RNA-seq dataset, one of the two tools whose concordance validates a hit)
- **find_circ** (second, complementary circRNA detection algorithm used to require concordance with CIRI2 calls)
- **R** (implementation environment for computing per-sample mCRI and running the group t tests between knockdown and control conditions)

## Examples

```
t.test(mcri_values[condition == 'knockdown'], mcri_values[condition == 'control'])  # group t test of per-sample mCRI, run separately per RBP and per tool (CIRI2, find_circ)
```

## Evaluation signals

- 13 of the 22 candidate RBPs show a statistically significant mCRI change upon knockdown (group t test)
- QKI knockdown is significant (positive control must pass)
- SIGLEC15 knockdown is non-significant (negative control must pass)
- The dual-tool concordance list (RBPs significant under both CIRI2 and find_circ) matches SRSF5, HNRNPA2B1, HNRNPF, KHSRP, and PSIP1
- mcri_validation_results.csv contains the per-sample mCRI table and significance summary in the expected schema

## Limitations

- Only 13 of 22 putative regulators had suitable knockdown RNA-seq data; RBPs lacking public shRNA/siRNA datasets cannot be assessed this way
- Support by a single circRNA-detecting tool is weaker evidence; only RBPs significant under both CIRI2 and find_circ (5 of 13) were counted as confidently supported
- mCRI is a global (per-sample) index, so it detects global circRNA abundance shifts and cannot attribute effects to individual circRNAs

## Evidence

- [other] Compute the normalized modified circRNA index (mCRI) per sample, quantifying global circRNA abundance relative to linear RNA expression for each knockdown/control pair.: [evidence span withheld — anchor status: not_found]
- [other] Run group t tests comparing per-sample mCRI between knockdown and control conditions for each RBP, including QKI as a positive control and SIGLEC15 as a negative control.: [evidence span withheld — anchor status: not_found]
- [other] Require concordance between CIRI2 and find_circ calls, verifying that SRSF5, HNRNPA2B1, HNRNPF, KHSRP, and PSIP1 are supported by both circRNA-detecting tools.: [evidence span withheld — anchor status: not_found]
