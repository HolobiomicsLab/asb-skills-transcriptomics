---
name: te-subfamily-correlation-analysis
description: Use when comparing TE quantification methods on short-read single-cell
  data and matched long-read (scNanoRNAseq or PacBio) data exist from the same cells/tissue
  to serve as ground truth for subfamily-level accuracy validation.
when_to_use_negative:
- No matched long-read data from the same cells or tissue exists, so there is no ground
  truth for correlation-based validation
- The goal is gene expression quantification rather than TE quantification — MATES
  and this benchmark target TE loci/subfamilies
- Evaluating long autonomously expressed TEs such as L1HS spanning several thousand
  bases, where the article notes MATES quantification is limited
edam_operation: http://edamontology.org/operation_0570
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3170
tools:
- name: MATES
  role: Deep neural network-based TE quantifier (GPU-backed) run on matched short-read
    single-cell datasets to produce locus- and subfamily-level TE count matrices
  repo: https://github.com/mcgilldinglab/MATES
- name: scTE
  role: Existing single-cell TE quantification baseline run with default settings
    to produce comparable subfamily-level TE count matrices
- name: SoloTE
  role: Existing single-cell TE quantification baseline run with default settings
    to produce comparable subfamily-level TE count matrices
- name: TE_quantifier
  role: MATES module for quantifying sub-family and locus-level TE expression on short-read
    data
  repo: https://github.com/mcgilldinglab/MATES
- name: TE_quantifier_LongRead
  role: MATES module for TE quantification on long-read (nanopore/PacBio) data used
    to generate ground truth counts
  repo: https://github.com/mcgilldinglab/MATES
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-024-53114-7
    title: 'MATES: a deep learning-based model for locus-specific quantification of
      transposable elements in single cell'
  source_content_hash: sha256:bde9f12be3e14515c2170cf7d9db9ccbac887ad5afba9f4c3943bfd2e094afc4
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/mcgilldinglab/MATES
  repo_url: https://github.com/mcgilldinglab/MATES
  tools:
  - MATES
  - SoloTE
  - TE_quantifier
  - TE_quantifier_LongRead
  - scTE
  edam_operation: http://edamontology.org/operation_0570
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# te-subfamily-correlation-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark single-cell TE quantification methods (MATES, scTE, SoloTE) by correlating short-read subfamily-level TE counts against nanopore and PacBio long-read ground truth, quantifying accuracy via R² and one-sided F-tests on residual variances.

## When to use

Use when comparing TE quantification methods on short-read single-cell data and matched long-read (scNanoRNAseq or PacBio) data exist from the same cells/tissue to serve as ground truth for subfamily-level accuracy validation.

## When NOT to use

- No matched long-read data from the same cells or tissue exists, so there is no ground truth for correlation-based validation
- The goal is gene expression quantification rather than TE quantification — MATES and this benchmark target TE loci/subfamilies
- Evaluating long autonomously expressed TEs such as L1HS spanning several thousand bases, where the article notes MATES quantification is limited

## Inputs

- Short-read single-cell datasets processed by MATES, scTE, and SoloTE into subfamily-level TE count matrices
- scNanoRNAseq melanoma brain metastasis nanopore long-read single-cell data
- PacBio long-read data from postnatal mouse brain
- TE reference annotation (e.g., TE_nooverlap.bed / TE_full.bed from build_reference.py)

## Outputs

- te_quantification_benchmark_r2.csv with columns for method, platform (nanopore single-cell, nanopore pseudo-bulk, PacBio), R2, and p-value
- Per-method R2 values for each platform setting
- One-sided F-test p-values comparing each method's residual variance to MATES

## How to apply

Run MATES (GPU-backed deep neural network TE quantifier) plus scTE and SoloTE with default settings on the matched short-read single-cell datasets to produce comparable subfamily-level TE count matrices. Extract subfamily-level TE counts from scNanoRNAseq melanoma brain metastasis nanopore data and PacBio postnatal mouse brain long-read data as ground truth. Compute the coefficient of determination (R²) between each method's short-read subfamily quantification and the long-read truth, separately for single-cell nanopore, pseudo-bulk nanopore, and PacBio settings. Attach p-values via a one-sided F-test on each method's residual variance against MATES, and write results to te_quantification_benchmark_r2.csv with method, platform, R², and p-value columns.

## Related tools

- **MATES** (Deep neural network-based TE quantifier (GPU-backed) run on matched short-read single-cell datasets to produce locus- and subfamily-level TE count matrices) — https://github.com/mcgilldinglab/MATES
- **scTE** (Existing single-cell TE quantification baseline run with default settings to produce comparable subfamily-level TE count matrices)
- **SoloTE** (Existing single-cell TE quantification baseline run with default settings to produce comparable subfamily-level TE count matrices)
- **TE_quantifier** (MATES module for quantifying sub-family and locus-level TE expression on short-read data) — https://github.com/mcgilldinglab/MATES
- **TE_quantifier_LongRead** (MATES module for TE quantification on long-read (nanopore/PacBio) data used to generate ground truth counts) — https://github.com/mcgilldinglab/MATES

## Examples

```
from MATES import TE_quantifier_LongRead; mates.run() # train model and quantify subfamily-level TE counts for benchmarking against long-read ground truth
```

## Evaluation signals

- Each reproduced R2 must match the reported reference value within a tolerance of 0.01 (e.g., MATES 0.7531, scTE 0.6499, SoloTE 0.6841 for single-cell nanopore; MATES 0.9276 vs 0.8591 vs 0.8654 for pseudo-bulk nanopore; MATES 0.5096 vs 0.3537 vs 0.4199 for PacBio)
- MATES R2 must exceed both scTE and SoloTE R2 in all three settings (single-cell nanopore, pseudo-bulk nanopore, PacBio)
- One-sided F-test p < 0.05 supporting the significance of the MATES accuracy improvement
- Output CSV contains all required columns: method, platform, R2, p-value with platform values limited to nanopore single-cell, nanopore pseudo-bulk, and PacBio

## Limitations

- MATES typically requires more computational resources such as GPUs than its counterparts scTE and SoloTE for quantifying TEs
- MATES has limited ability to quantify long autonomously expressed TEs such as L1HS, which often span several thousand bases beyond the 2001 bp coverage vector
- The estimated multi-mapping rate may not be very accurate for TEs longer than 2001 bp due to incomplete flanking coverage, and identical TE copies such as HERVK isoforms are hard to distinguish

## Evidence

- [results] MATES achieved a stronger correlation with long-read TE quantification (R2 = 0.7531) than both scTE (R2 = 0.6499) and SoloTE (R2 = 0.6841): [evidence span withheld — anchor status: not_found]
- [other] Extract subfamily-level TE counts from the scNanoRNAseq melanoma brain metastasis nanopore long-read data and the PacBio postnatal mouse brain long-read data: [evidence span withheld — anchor status: not_found]
- [readme] The bam_processor module efficiently manages input BAM files by partitioning them into sub-BAM files for individual cells: [evidence span withheld — anchor status: external_doc]
