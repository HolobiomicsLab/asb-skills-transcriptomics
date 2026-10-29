---
name: fusion-individual-level-weight-training
description: Decision support for benchmarking FUSION's individual-level training
  against summary-level OTTERS/PRS methods, or whenever individual-level genotype
  and expression reference data are available for TWAS Stage I weight estimation.
when_to_use_negative:
- Only summary-level eQTL data are available and no individual-level reference panel
  exists — use OTTERS summary-level training (P+T, lassosum, SDPR, PRS-CS) instead
- The reference panel is small (e.g., GTEx V6 with n=338) and the goal is maximizing
  valid GReX models — summary-level references like eQTLGen (n=31,684) yield more
  valid models
- The tissue has no individual-level genotype and expression reference data available
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3517
tools:
- name: FUSION
  role: Trains eQTL weights from individual-level GTEx V6 genotype and expression
    reference data for the same gene set as the summary-level methods
- name: OTTERS
  role: Framework against which FUSION individual-level training is benchmarked; provides
    the summary-level training pipeline and evaluation workflow
  repo: https://github.com/daiqile96/OTTERS
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-023-36862-w
    title: 'OTTERS: a powerful TWAS framework leveraging summary-level reference data'
  source_content_hash: sha256:8ee7dc9becac1d5552b9b4e10c40d24a7ba58cb699979432b734688541cb838b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/daiqile96/OTTERS
  tools:
  - FUSION
  - OTTERS
  - PRS-CS
  - SDPR
  - fdrtool
  - lassosum
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3517
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# fusion-individual-level-weight-training

## Summary

Train GReX imputation models (TWAS Stage I eQTL weights) with FUSION from individual-level reference genotype and expression data (e.g., GTEx V6) to serve as the individual-level-data baseline against summary-level PRS-based training (P+T, lassosum, SDPR, PRS-CS).

## When to use

Decision support for benchmarking FUSION's individual-level training against summary-level OTTERS/PRS methods, or whenever individual-level genotype and expression reference data are available for TWAS Stage I weight estimation.

## When NOT to use

- Only summary-level eQTL data are available and no individual-level reference panel exists — use OTTERS summary-level training (P+T, lassosum, SDPR, PRS-CS) instead
- The reference panel is small (e.g., GTEx V6 with n=338) and the goal is maximizing valid GReX models — summary-level references like eQTLGen (n=31,684) yield more valid models
- The tissue has no individual-level genotype and expression reference data available

## Inputs

- Individual-level reference genotype data (binary PLINK format, e.g., GTEx V6 whole blood, n=338)
- Matched reference gene expression data for the same tissue (GTEx V6)
- Gene annotation file (CHROM, GeneStart, GeneEnd, TargetID)
- Held-out expression/genotype samples for validation (e.g., 315 GTEx V8 whole-blood samples)

## Outputs

- Per-gene FUSION eQTL weights trained on individual-level reference data
- Per-gene test R2 from predicting expression in held-out samples
- Count of valid models (genes with R2 > 0.01) and median test R2, written to a CSV summary table

## How to apply

Train FUSION eQTL weights on the individual-level reference genotype and expression data for the same gene set used by the summary-level methods (in the OTTERS benchmark this was GTEx V6 whole blood, n=338, versus eQTLGen summary data n=31,684). Predict gene expression in held-out samples (315 GTEx V8 whole-blood samples) and compute per-gene test R2. Count valid models as genes with R2 > 0.01 and compute the median test R2 per method. In the OTTERS comparison, FUSION trained on the smaller GTEx V6 individual-level reference yielded 4,704 valid models — roughly half of PRS-CS's 10,337 — demonstrating that summary-level references with much larger effective sample sizes increase the number of valid GReX imputation models.

## Related tools

- **FUSION** (Trains eQTL weights from individual-level GTEx V6 genotype and expression reference data for the same gene set as the summary-level methods)
- **OTTERS** (Framework against which FUSION individual-level training is benchmarked; provides the summary-level training pipeline and evaluation workflow) — https://github.com/daiqile96/OTTERS

## Evaluation signals

- Per-gene test R2 computed by predicting expression in 315 held-out GTEx V8 whole-blood samples
- Valid-model count = number of genes with test R2 > 0.01; expect ~4,704 valid models for FUSION on GTEx V6 (vs 10,337 for PRS-CS) per Table 1
- Median test R2 per method reported alongside valid-model counts in the CSV summary table
- Same gene set (16,699 tested genes) used across all methods for a fair comparison

## Limitations

- FUSION requires individual-level genotype and expression reference data, which are far smaller than available summary-level references (GTEx V6 n=338 vs eQTLGen n=31,684), yielding fewer valid GReX models
- FUSION obtained only 4,704 valid models (R2 > 0.01) compared to 10,337 for PRS-CS in the OTTERS benchmark
- Training costs ~20 min per gene for all five models per gene, which may be computationally challenging for genome-wide gene sets

## Evidence

- [methods] FUSION eQTL weights on the GTEx V6 individual-level genotype and expression reference for the same gene set: [evidence span withheld — anchor status: not_found]
- [results] FUSION trained on smaller GTEx V6 data obtained only 4704: [evidence span withheld — anchor status: not_found]
- [intro] eQTLGen reference sample size is 31,684 whereas GTEx V6 is only 338: "the sample size of the summary-based eQTLGen reference sample is 31,684 for blood, whereas the sample size of the individual-level GTEx V6 reference is only 338 for the same tissue"
- [discussion] training cost ~20 min for all five training models per gene: "training GReX imputation models by all individual methods on average cost ~20 min for all five training models per gene"
- [intro] various training tools exist for fitting GReX regression models: "a variety of training tools exist for fitting GReX regression models using reference expression and genetic data, including PrediXcan19, FUSION20, and TIGAR22"
