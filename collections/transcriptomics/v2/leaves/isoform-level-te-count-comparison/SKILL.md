---
name: isoform-level-te-count-comparison
description: Use when evaluating whether a deep-learning TE quantifier recovers locus-
  and isoform-specific counts (e.g., 300 bp simulated Alu repeats, simple repeats,
  HERVK-int isoform pairs) against a known simulated ground truth.
when_to_use_negative:
- Quantifying long autonomously expressed TEs such as L1HS spanning several thousand
  bases — the 2001-bp coverage vector gives incomplete flanking coverage and an inaccurate
  multi-mapping rate
- Data lacking multi-mapping TE reads, where MATES's multi-mapping reallocation has
  little to learn (simple-mode support was only added in version 0.1.8)
- When GPU resources are unavailable, since MATES typically requires more computational
  resources such as GPUs than scTE and SoloTE
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: MATES
  role: Deep-learning quantifier whose locus/isoform-level counts (bam_processor,
    data_processor, MATES_model, TE_quantifier, TE_quantifier_LongRead) are compared
    to simulated ground truth and baselines
  repo: https://github.com/mcgilldinglab/MATES
- name: scTE
  role: Single-cell TE quantification baseline run on the identical simulated BAMs;
    recaptured 92.57% of simulated reads vs MATES 96.14%
- name: SoloTE
  role: Single-cell TE quantification baseline for long-read comparison (strategy
    R2 = 0.1178 vs MATES 0.4923 at locus level)
- name: Polyester
  role: Read simulator producing single-cell read datasets with known ground-truth
    locus abundances for benchmarking
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-024-53114-7
    title: 'MATES: a deep learning-based model for locus-specific quantification of
      transposable elements in single cell'
  source_content_hash: sha256:4ada127b7821e81399c45ded8c173741b940ec6890b229d34ddaf721d9756e6f
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
  - MATES_model
  - Polyester
  - SoloTE
  - TE_quantifier
  - TE_quantifier_LongRead
  - bam_processor
  - data_processor
  - scTE
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6470588235294118
n_claims: 17
---

# isoform-level-te-count-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark MATES locus/isoform-level transposable element quantification against simulated ground truth (Polyester) and competing single-cell methods (scTE, SoloTE) using locus-level R2, read-recapture rates, and isoform-pair counts such as HERVK TE1/TE2.

## When to use

Use when evaluating whether a deep-learning TE quantifier recovers locus- and isoform-specific counts (e.g., 300 bp simulated Alu repeats, simple repeats, HERVK-int isoform pairs) against a known simulated ground truth.

## When NOT to use

- Quantifying long autonomously expressed TEs such as L1HS spanning several thousand bases — the 2001-bp coverage vector gives incomplete flanking coverage and an inaccurate multi-mapping rate
- Data lacking multi-mapping TE reads, where MATES's multi-mapping reallocation has little to learn (simple-mode support was only added in version 0.1.8)
- When GPU resources are unavailable, since MATES typically requires more computational resources such as GPUs than scTE and SoloTE

## Inputs

- Polyester-simulated single-cell BAM files with known ground-truth TE locus abundances (6 Alu subfamilies, 15 simple-repeat subfamilies, HERVK-int TE1/TE2 isoform pair, 100-bp reads modeled on scNanoRNA-seq)
- TE reference annotation (e.g., TE_full.bed / TE_nooverlap.bed from MATES reference building)
- Simulated BAM files processed identically for baseline methods scTE and SoloTE

## Outputs

- Locus-specific TE count matrices per cell from MATES, scTE, and SoloTE
- simulation_benchmark_results.csv with locus-level R2 correlations, percentage of simulated reads captured per method, and HERVK TE1/TE2 isoform counts
- Long-read locus-level quantification metrics (e.g., R2 = 0.4923 for MATES vs 0.1178 for SoloTE strategy)

## How to apply

Generate simulated single-cell reads with Polyester over known ground-truth locus abundances: 6 Alu subfamilies (~300 bp), 15 simple-repeat subfamilies, the HERVK-int isoform pair TE1/TE2, and 100-bp reads modeled on scNanoRNA-seq data. Process simulated BAMs with the MATES bam_processor and data_processor modules to segregate unique vs multi-mapping reads and build 2001-bp coverage vectors centered on TE start positions, then run MATES_model (TE_quantifier for short reads, TE_quantifier_LongRead for the 100-bp reads). Run scTE and SoloTE on the identical simulated BAMs as baselines. Compare all methods to the Polyester ground truth by computing locus-level R2 correlations, the percentage of simulated reads captured per method, and HERVK TE1/TE2 isoform counts, writing metrics to simulation_benchmark_results.csv. Accept the benchmark when MATES reaches Alu R2 = 0.7420 with 96.14% of simulated reads captured (vs scTE 92.57%), simple-repeat locus R2 = 0.7479, HERVK counts 4140/1295 against ground truth 4126/1282, and long-read locus-level R2 = 0.4923 (vs SoloTE strategy 0.1178).

## Related tools

- **MATES** (Deep-learning quantifier whose locus/isoform-level counts (bam_processor, data_processor, MATES_model, TE_quantifier, TE_quantifier_LongRead) are compared to simulated ground truth and baselines) — https://github.com/mcgilldinglab/MATES
- **scTE** (Single-cell TE quantification baseline run on the identical simulated BAMs; recaptured 92.57% of simulated reads vs MATES 96.14%)
- **SoloTE** (Single-cell TE quantification baseline for long-read comparison (strategy R2 = 0.1178 vs MATES 0.4923 at locus level))
- **Polyester** (Read simulator producing single-cell read datasets with known ground-truth locus abundances for benchmarking)

## Examples

```
from MATES import MATES_pipeline; mates = MATES_pipeline(TE_mode, data_mode, sample_list_file, bam_path_file); mates.preprocessing(); mates.run()
```

## Evaluation signals

- Alu locus-level R2 = 0.7420 between MATES counts and the Polyester simulated ground truth
- MATES recaptures 96.14% of simulated reads on average, exceeding scTE's 92.57%
- Simple-repeat locus-level R2 = 0.7479 against ground truth
- HERVK-int isoform counts MATES = 4140/1295 (TE1/TE2) against ground truth 4126/1282
- 100-bp long-read locus-level R2 = 0.4923 for MATES vs 0.1178 for the SoloTE strategy

## Limitations

- Challenges distinguishing identical TE copies such as HERVK isoforms due to high sequence similarity and lack of uniquely mappable flanking regions
- Multi-mapping rate estimation may be inaccurate for TEs longer than 2001 bp due to incomplete flanking coverage; parameter adjustments (e.g., coverage vector length) are not definitive solutions
- MATES typically requires more computational resources such as GPUs than scTE and SoloTE for quantifying TEs

## Evidence

- [readme] Simple-mode pipeline API for TE quantification: [evidence span withheld — anchor status: external_doc]
