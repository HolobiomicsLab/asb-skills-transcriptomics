---
name: dual-tool-circrna-call-concordance-analysis
description: Use when public shRNA/siRNA knockdown RNA-seq datasets (e.g., GEO/ENCODE
  FASTQ) are available for candidate RBPs and you must decide which regulators affect
  global circRNA expression via mCRI change and dual-tool (CIRI2 + find_circ) concordance.
when_to_use_negative:
- Input is already a circRNA/feature table and no raw alignments exist for independent
  CIRI2 and find_circ calling, so concordance cannot be established
- The analysis goal is circRNA discovery in tumor/tissue compendia (e.g., MiOncoCirc)
  rather than assessing global circRNA change upon RBP knockdown
- No paired knockdown/control samples exist for an RBP, so the mCRI group t test between
  conditions cannot be performed
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: CIRI2
  role: circRNA-detecting algorithm run on raw alignments of each knockdown RNA-seq
    dataset, one of the two tools whose concordance is required
- name: find_circ
  role: second complementary circRNA-detecting algorithm run per dataset to confirm
    calls made by CIRI2
- name: R
  role: environment for computing per-sample mCRI, running group t tests between knockdown
    and control, and writing mcri_validation_results.csv
- name: survival
  role: R package used elsewhere in CRIT for statistical testing (log-rank survival
    analysis of gene alterations)
- name: CRIT
  role: NMF-based pipeline that surfaced the candidate RBPs subsequently validated
    by this knockdown/mCRI analysis
  repo: https://github.com/BioinformaticsSTU/CRIT
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.omtn.2022.10.015
    title: 'CRIT: Identifying RNA-binding protein regulator in circRNA life cycle
      via non-negative matrix factorization'
  source_content_hash: sha256:2783aa46bca5de755d6ac44c790be07a19d93b4fe29b3bbe363a54e0a1e7a480
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
  - CIRI2
  - CRIT
  - R
  - find_circ
  - survival
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# dual-tool-circrna-call-concordance-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A validation skill from the CRIT pipeline that tests whether RBP knockdown alters global circRNA expression by detecting circRNAs in public shRNA/siRNA RNA-seq data with two complementary algorithms (CIRI2 and find_circ), computing the normalized modified circRNA index (mCRI), and requiring concordance between the two detectors before calling a regulator validated.

## When to use

Use when public shRNA/siRNA knockdown RNA-seq datasets (e.g., GEO/ENCODE FASTQ) are available for candidate RBPs and you must decide which regulators affect global circRNA expression via mCRI change and dual-tool (CIRI2 + find_circ) concordance.

## When NOT to use

- Input is already a circRNA/feature table and no raw alignments exist for independent CIRI2 and find_circ calling, so concordance cannot be established
- The analysis goal is circRNA discovery in tumor/tissue compendia (e.g., MiOncoCirc) rather than assessing global circRNA change upon RBP knockdown
- No paired knockdown/control samples exist for an RBP, so the mCRI group t test between conditions cannot be performed

## Inputs

- Raw FASTQ files of shRNA/siRNA knockdown RNA-seq datasets from GEO/ENCODE (e.g., the 27 datasets listed in Table S5 for 22 candidate RBPs)
- Raw alignment (BAM) files from which CIRI2 and find_circ call circRNAs
- Knockdown/control sample metadata mapping datasets to RBPs and conditions

## Outputs

- Per-sample mCRI table (global circRNA abundance relative to linear RNA per knockdown/control pair)
- mcri_validation_results.csv containing per-sample mCRI values and the group t test significance summary
- Dual-tool concordance list of RBPs supported by both CIRI2 and find_circ (expected: SRSF5, HNRNPA2B1, HNRNPF, KHSRP, PSIP1)

## How to apply

Download raw FASTQ files of the shRNA/siRNA knockdown RNA-seq datasets for each candidate RBP (e.g., from GEO/ENCODE), and detect circRNAs per dataset from the raw alignments independently with two complementary algorithms, CIRI2 and find_circ. For each knockdown/control pair, compute the normalized modified circRNA index (mCRI), a per-sample measure of global circRNA abundance relative to linear RNA expression. Run group t tests comparing per-sample mCRI between knockdown and control conditions for each RBP, always including QKI as a positive control and SIGLEC15 as a negative control. Require concordance between CIRI2 and find_circ calls before accepting a regulator as supported, and write the per-sample mCRI table and significance summary to mcri_validation_results.csv.

## Related tools

- **CIRI2** (circRNA-detecting algorithm run on raw alignments of each knockdown RNA-seq dataset, one of the two tools whose concordance is required)
- **find_circ** (second complementary circRNA-detecting algorithm run per dataset to confirm calls made by CIRI2)
- **R** (environment for computing per-sample mCRI, running group t tests between knockdown and control, and writing mcri_validation_results.csv)
- **survival** (R package used elsewhere in CRIT for statistical testing (log-rank survival analysis of gene alterations))
- **CRIT** (NMF-based pipeline that surfaced the candidate RBPs subsequently validated by this knockdown/mCRI analysis) — https://github.com/BioinformaticsSTU/CRIT

## Evaluation signals

- 13 of the 22 RBPs show a statistically significant per-sample mCRI change upon knockdown by group t test
- QKI (positive control) is significant while SIGLEC15 (negative control) is non-significant
- The dual-tool concordance list matches exactly SRSF5, HNRNPA2B1, HNRNPF, KHSRP, and PSIP1 (supported by both CIRI2 and find_circ)
- mcri_validation_results.csv contains both the per-sample mCRI table and the significance summary
- mCRI values are computed per sample as global circRNA abundance relative to linear RNA expression for each knockdown/control pair

## Limitations

- Only a subset of candidate RBPs have suitable public knockdown RNA-seq data; among 22 putative regulators with suitable data, 13 showed evidence and only 5 were supported by both circRNA-detecting tools
- Validation requires concordance between CIRI2 and find_circ calls, so regulators detected by only one circRNA-detecting tool are not accepted
- The skill depends on raw FASTQ/alignments of shRNA/siRNA datasets; generalization of this validation strategy to other conditions (e.g., other diseases) is untested per the article's discussion

## Evidence
