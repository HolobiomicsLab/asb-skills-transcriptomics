---
name: cross-method-benchmark-reproduction
description: 'Use when a claim that ''method A outperforms methods B and C'' must
  be reproduced end-to-end: re-running all quantifiers on the same mapped reads, rebuilding
  feature matrices under multiple perspectives, and statistically testing the performance
  gap.'
when_to_use_negative:
- You only need to run MATES itself (e.g. quantification on a new dataset) rather
  than compare it against scTE/SoloTE baselines
- The TE of interest is a long autonomously expressed element like L1HS spanning several
  thousand bases, where MATES's own quantification is known to be limited and benchmarks
  on it would be confounded
- Input is already a processed feature/count table — the skill requires re-quantification
  from mapped reads under identical conditions for all methods
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: MATES
  role: Deep neural network-based TE quantifier (GPU-backed) that allocates multi-mapping
    reads to specific TE loci; the method under test
  repo: https://github.com/mcgilldinglab/MATES
- name: scTE
  role: Baseline single-cell TE quantification method run with default settings for
    the head-to-head comparison
- name: SoloTE
  role: Baseline single-cell TE quantification method run with default settings for
    the head-to-head comparison
- name: TE_quantifier
  role: MATES module for quantifying sub-family and locus-level TE expression in the
    short-read pipeline
  repo: https://github.com/mcgilldinglab/MATES
- name: TE_quantifier_LongRead
  role: MATES module used when quantifying long-read data in the accuracy-validation
    arm
  repo: https://github.com/mcgilldinglab/MATES
provenance:
  source_task_ids:
  - task_001
  - task_006
  source_papers:
  - doi: 10.1038/s41467-024-53114-7
    title: 'MATES: a deep learning-based model for locus-specific quantification of
      transposable elements in single cell'
  source_content_hash: sha256:941cf8c2108d02cd70c0f14e47868f78a536b8cd644ec3df72633058eace703f
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
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# cross-method-benchmark-reproduction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A skill for reproducing head-to-head benchmark comparisons between a new method (MATES) and competing single-cell TE quantifiers (scTE, SoloTE), covering both downstream cell-clustering performance (ARI/NMI) and long-read-validated quantification accuracy (R² with one-sided F-tests).

## When to use

Use when a claim that 'method A outperforms methods B and C' must be reproduced end-to-end: re-running all quantifiers on the same mapped reads, rebuilding feature matrices under multiple perspectives, and statistically testing the performance gap.

## When NOT to use

- You only need to run MATES itself (e.g. quantification on a new dataset) rather than compare it against scTE/SoloTE baselines
- The TE of interest is a long autonomously expressed element like L1HS spanning several thousand bases, where MATES's own quantification is known to be limited and benchmarks on it would be confounded
- Input is already a processed feature/count table — the skill requires re-quantification from mapped reads under identical conditions for all methods

## Inputs

- Raw single-cell FASTQs or BAMs: 10x mouse chemical reprogramming scRNA-seq, Smart-Seq2 human glioblastoma scRNA-seq, 10x adult mouse brain scATAC-seq
- STAR/STARsolo alignment outputs with unique- and multi-mapping read annotations
- Short-read quantification outputs (locus- and subfamily-level TE count matrices) from MATES, scTE, and SoloTE
- Long-read ground truth: scNanoRNAseq melanoma brain metastasis nanopore data and PacBio postnatal mouse brain data (subfamily-level TE counts)
- Known cell labels for ARI/NMI computation

## Outputs

- Cell-clustering ARI and NMI distributions per method per perspective (Gene/Peak+TE, TE-only, Multi-mapping-TE-only) over 10 random seeds, with one-sided Student's t-test p-values
- Locus-level MATES-vs-SoloTE ARI comparison (~10.52% higher ARI, P ≈ 2.60e-12)
- te_quantification_benchmark_r2.csv with columns: method, platform (nanopore single-cell, nanopore pseudo-bulk, PacBio), R², p-value
- Reproduced R² values: MATES 0.7531 / scTE 0.6499 / SoloTE 0.6841 (single-cell nanopore), MATES 0.9276 / 0.8591 / 0.8654 (pseudo-bulk), MATES 0.5096 / 0.3537 / 0.4199 (PacBio)

## How to apply

Map raw reads (10x mouse chemical reprogramming scRNA-seq, Smart-Seq2 human glioblastoma scRNA-seq, 10x adult mouse brain scATAC-seq) with STAR/STARsolo and segregate unique- vs multi-mapping reads, then quantify TE loci with MATES (which allocates multi-mapping reads using a 2001 bp coverage vector centered on each TE start), scTE, and SoloTE. Build feature matrices under three perspectives per method — Gene/Peak+TE, TE-only, and Multi-mapping-TE-only — cluster cells (Louvain/Leiden in Scanpy), and compute Adjusted Rand Index and Normalized Mutual Information against known labels over N=10 random seeds, comparing methods with one-sided Student's t-tests. For accuracy validation, run all three methods on matched short-read data and compute R² against subfamily-level ground truth from scNanoRNAseq melanoma brain metastasis nanopore data and PacBio postnatal mouse brain data, plus one-sided F-tests on residual variances. Accept the clustering reproduction if the locus-level comparison shows MATES ~10.52% higher ARI than SoloTE (one-sided t-test P ≈ 2.60e-12) and MATES ranking first across the three perspectives; accept the accuracy reproduction if each R² matches within a 0.01 tolerance and MATES > scTE and MATES > SoloTE in all three settings with F-test p < 0.05.

## Related tools

- **MATES** (Deep neural network-based TE quantifier (GPU-backed) that allocates multi-mapping reads to specific TE loci; the method under test) — https://github.com/mcgilldinglab/MATES
- **scTE** (Baseline single-cell TE quantification method run with default settings for the head-to-head comparison)
- **SoloTE** (Baseline single-cell TE quantification method run with default settings for the head-to-head comparison)
- **TE_quantifier** (MATES module for quantifying sub-family and locus-level TE expression in the short-read pipeline) — https://github.com/mcgilldinglab/MATES
- **TE_quantifier_LongRead** (MATES module used when quantifying long-read data in the accuracy-validation arm) — https://github.com/mcgilldinglab/MATES

## Examples

```
from MATES import MATES_pipeline; mates = MATES_pipeline(TE_mode, data_mode, sample_list_file, bam_path_file); mates.preprocessing(); mates.run()
```

## Evaluation signals

- Locus-level MATES-vs-SoloTE comparison reproduces ~10.52% higher ARI for MATES with one-sided t-test P ≈ 2.60e-12
- MATES ranks first across all three benchmarking perspectives (Gene/Peak+TE, TE-only, Multi-mapping-TE-only) as in Fig. 6a, b
- Each reproduced R² matches the reported reference value within a tolerance of 0.01 (e.g. MATES single-cell nanopore R² = 0.7531)
- MATES > scTE and MATES > SoloTE in all three long-read settings (single-cell nanopore, pseudo-bulk nanopore, PacBio)
- One-sided F-test p < 0.05 for the MATES accuracy improvement in each setting; output CSV has schema method, platform, R², p-value

## Limitations

- MATES requires more computational resources (GPUs) than scTE and SoloTE for quantifying TEs
- MATES has limited ability to quantify long autonomously expressed TEs such as L1HS spanning several thousand bases; the 2001 bp coverage vector makes multi-mapping rate estimation inaccurate for TEs longer than 2001 bp
- Identical TE copies such as HERVK isoforms are hard to distinguish due to high sequence similarity and lack of uniquely mappable flanking regions
- Trimming TE-gene overlaps (exclusive TE_mode) may unintentionally distort TE quantification when TEs intersect neighboring genes, and no quantitative guidance for opting out is provided

## Evidence

- [results] MATES consistently outperformed scTE and SoloTE in cell clustering across the three benchmarking perspectives: [evidence span withheld — anchor status: not_found]
- [results] Validated against nanopore long-read single-cell ground truth, MATES achieved a stronger correlation with long-read TE quantification (R2 = 0.7531): [evidence span withheld — anchor status: not_found]
