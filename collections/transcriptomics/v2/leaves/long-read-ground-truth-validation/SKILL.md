---
name: long-read-ground-truth-validation
description: Use when you need to validate or benchmark a short-read single-cell TE
  quantification method and matched long-read data (nanopore scNanoRNAseq or PacBio)
  from the same sample exists to serve as ground truth at the TE subfamily level.
when_to_use_negative:
- When no matched long-read (nanopore or PacBio) dataset from the same sample exists
  — there is no independent ground truth to correlate against
- When quantifying long autonomously expressed TEs such as L1HS spanning several thousand
  bases, since MATES's 2001 bp coverage vector and multi-mapping rate estimation are
  inaccurate for TEs longer than 2001 bp
- When the goal is locus-level resolution for highly identical TE copies (e.g., HERVK
  isoforms) that lack uniquely mappable flanking regions
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_0091
tools:
- name: MATES
  role: Deep neural network-based TE quantifier (GPU-backed) producing short-read
    locus- and subfamily-level TE count matrices under test
  repo: https://github.com/mcgilldinglab/MATES
- name: scTE
  role: Existing single-cell TE quantification comparator run with default settings
    on the same short-read data
- name: SoloTE
  role: Existing single-cell TE quantification comparator run with default settings
    on the same short-read data
- name: TE_quantifier_LongRead
  role: MATES module for quantifying TE expression directly from long-read BAM files
    to generate the ground truth counts
  repo: https://github.com/mcgilldinglab/MATES
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-024-53114-7
    title: 'MATES: a deep learning-based model for locus-specific quantification of
      transposable elements in single cell'
  source_content_hash: sha256:b093557f577c5507ad7119c51660c46fe6a2073bdca158ea810762cdd445aad9
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# long-read-ground-truth-validation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark short-read single-cell TE quantification methods (MATES, scTE, SoloTE) against nanopore and PacBio long-read sequencing from the same cells, using subfamily-level R² against long-read counts as ground truth. This validates that MATES's deep-learning multi-mapping read allocation genuinely improves TE quantification accuracy on real short-read data.

## When to use

Use when you need to validate or benchmark a short-read single-cell TE quantification method and matched long-read data (nanopore scNanoRNAseq or PacBio) from the same sample exists to serve as ground truth at the TE subfamily level.

## When NOT to use

- When no matched long-read (nanopore or PacBio) dataset from the same sample exists — there is no independent ground truth to correlate against
- When quantifying long autonomously expressed TEs such as L1HS spanning several thousand bases, since MATES's 2001 bp coverage vector and multi-mapping rate estimation are inaccurate for TEs longer than 2001 bp
- When the goal is locus-level resolution for highly identical TE copies (e.g., HERVK isoforms) that lack uniquely mappable flanking regions

## Inputs

- Short-read single-cell sequencing datasets (e.g., 10X scRNA-seq BAM files) shared with the long-read sample
- scNanoRNAseq melanoma brain metastasis nanopore long-read single-cell sequencing data
- PacBio long-read data from postnatal mouse brain
- TE and gene reference files (e.g., TE_full.bed / TE_nooverlap.bed) built via MATES build_reference.py
- Subfamily-level TE count matrices from MATES, scTE, and SoloTE

## Outputs

- te_quantification_benchmark_r2.csv with columns: method, platform (nanopore single-cell, nanopore pseudo-bulk, PacBio), R2, p-value
- Subfamily-level R² coefficients per method per platform (reference: MATES 0.7531 / scTE 0.6499 / SoloTE 0.6841 single-cell nanopore; MATES 0.9276 / scTE 0.8591 / SoloTE 0.8654 pseudo-bulk nanopore; MATES 0.5096 / scTE 0.3537 / SoloTE 0.4199 PacBio)
- One-sided F-test p-values for residual variance of each method against MATES

## How to apply

Run MATES (GPU-backed deep neural network TE quantifier) and the comparator methods scTE and SoloTE with default settings on the matched short-read single-cell datasets to obtain locus- and subfamily-level TE count matrices. Extract subfamily-level TE counts from the scNanoRNAseq melanoma brain metastasis nanopore long-read data and the PacBio postnatal mouse brain long-read data as ground truth. Compute the coefficient of determination (R²) between each method's short-read subfamily-level quantification and the long-read truth, separately for single-cell nanopore, pseudo-bulk nanopore, and PacBio settings. Use MATES's TE_quantifier_LongRead module to handle long-read BAM inputs when quantifying from long-read data. Attach one-sided F-test p-values on residual variances (each method vs MATES) to the accuracy differences. Write all values to te_quantification_benchmark_r2.csv with columns method, platform, R2, and p-value.

## Related tools

- **MATES** (Deep neural network-based TE quantifier (GPU-backed) producing short-read locus- and subfamily-level TE count matrices under test) — https://github.com/mcgilldinglab/MATES
- **scTE** (Existing single-cell TE quantification comparator run with default settings on the same short-read data)
- **SoloTE** (Existing single-cell TE quantification comparator run with default settings on the same short-read data)
- **TE_quantifier_LongRead** (MATES module for quantifying TE expression directly from long-read BAM files to generate the ground truth counts) — https://github.com/mcgilldinglab/MATES

## Examples

```
from MATES import bam_processor, TE_quantifier_LongRead
```

## Evaluation signals

- Each reproduced R² matches the reported reference value within a tolerance of 0.01 (e.g., MATES 0.7531, scTE 0.6499, SoloTE 0.6841 for single-cell nanopore)
- MATES R² > scTE R² and MATES R² > SoloTE R² in all three settings (nanopore single-cell, nanopore pseudo-bulk, PacBio)
- One-sided F-test on residual variances yields p < 0.05 for the MATES improvement in each setting
- Output CSV te_quantification_benchmark_r2.csv contains exactly the columns method, platform, R2, p-value and rows covering all three platforms
- Long-read ground truth counts are subfamily-level and derived from the same cells/sample as the short-read data being benchmarked

## Limitations

- MATES typically requires more computational resources such as GPUs than its counterparts scTE and SoloTE for quantifying TEs
- Multi-mapping rate estimation may be inaccurate for TEs longer than 2001 bp due to incomplete flanking coverage of the coverage vector
- Identical TE copies such as HERVK isoforms are hard to distinguish due to high sequence similarity and lack of uniquely mappable flanking regions
- Overall correlations are lower on PacBio mouse brain data (MATES R² 0.5096) than on nanopore data, so cross-platform generalization should be interpreted cautiously
- Trimming TE-gene overlaps (exclusive mode) could unintentionally distort TE quantification and no quantitative evaluation of when to opt out is provided

## Evidence

- [readme] long-read quantification workflow in MATES: [evidence span withheld — anchor status: external_doc]
