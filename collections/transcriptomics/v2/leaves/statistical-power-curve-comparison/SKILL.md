---
name: statistical-power-curve-comparison
description: Use when you need to quantify whether modeling replicates confers robustness
  against a single outlier sample versus pooling counts, by tracing detection power
  as sample size increases under simulated data.
when_to_use_negative:
- The goal is real-data ASAS detection from aligned RNA-seq reads rather than a controlled
  simulation benchmark — use the full PAIRADISE pipeline (personalization, mapping,
  counting) instead of the simulation harness.
- There are no replicates and no outlier concern; a single-sample allelic test suffices
  and the pooled comparison adds no information.
- The comparison method must handle overdispersion rather than the specific replicate-versus-pooling
  robustness question; Fisher's pooled test is only the paper's naive baseline.
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3517
tools:
- name: PAIRADISE
  role: Replicate-aware statistical model under evaluation; run on each simulated
    dataset and significant events called at FDR <= 10%
  repo: https://github.com/Xinglab/PAIRADISE
- name: Fisher's exact test (pooled reads)
  role: Baseline comparison method run on reads pooled across replicates for the same
    simulated datasets
- name: nloptr
  role: R optimization package used by the PAIRADISE statistical model fitting
- name: R
  role: Language in which the PAIRADISE statistical model and power simulation are
    executed
- name: Python
  role: Used for generating simulated count data and tabulating/plotting power results
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.ajhg.2020.07.005
    title: Detecting Allele-Specific Alternative Splicing from Population-Scale RNA-Seq
      Data
  source_content_hash: sha256:22b89799a14cd9956be983719115b3fc57a8b72659fa446f79a30e029fd3767e
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/Xinglab/PAIRADISE
  repo_url: https://github.com/Xinglab/PAIRADISE
  tools:
  - Fisher's exact test (pooled reads)
  - PAIRADISE
  - Python
  - R
  - nloptr
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3517
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# statistical-power-curve-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark the statistical power of a replicate-aware model (PAIRADISE) against a naive read-pooling alternative (Fisher's exact test on pooled reads) across increasing replicate numbers under an injected-outlier simulation, producing a power-versus-sample-size comparison.

## When to use

Use when you need to quantify whether modeling replicates confers robustness against a single outlier sample versus pooling counts, by tracing detection power as sample size increases under simulated data.

## When NOT to use

- The goal is real-data ASAS detection from aligned RNA-seq reads rather than a controlled simulation benchmark — use the full PAIRADISE pipeline (personalization, mapping, counting) instead of the simulation harness.
- There are no replicates and no outlier concern; a single-sample allelic test suffices and the pooled comparison adds no information.
- The comparison method must handle overdispersion rather than the specific replicate-versus-pooling robustness question; Fisher's pooled test is only the paper's naive baseline.

## Inputs

- Simulated allele-specific alternative splicing count data (per-allele exon inclusion and skipping counts with effective isoform lengths)
- Outlier-injected replicate with PSI drawn from Uniform[0,1]
- PAIRADISE simulation and statistical-model scripts from github.com/Xinglab/PAIRADISE scripts directory
- R environment with nloptr, doParallel, foreach packages

## Outputs

- simulation_outlier_power_comparison.csv (detection power for PAIRADISE versus Fisher's pooled test across replicate counts)
- Power-versus-sample-size curves for both methods
- Significant ASAS event calls at FDR <= 10% (Benjamini-Hochberg adjusted) per simulated dataset

## How to apply

Clone github.com/Xinglab/PAIRADISE and locate the simulation and statistical-model scripts in the scripts directory. Generate simulated allele-specific alternative splicing count data — per-allele exon inclusion counts (IJC_REF/IJC_ALT) and skipping counts (SJC_REF/SJC_ALT) with effective isoform lengths (incLen/skpLen) — following the paper's simulation settings, injecting one outlier replicate whose PSI value is drawn from Uniform[0,1]. Run the PAIRADISE paired statistical model (R with nloptr optimization, parallelized via doParallel/foreach) on each simulated dataset and call significant events at the default FDR <= 10% threshold using Benjamini-Hochberg adjustment. For the same datasets, run Fisher's exact test on reads pooled across replicates as the comparison strategy. Repeat the simulation across increasing numbers of replicates to trace the power-versus-sample-size curve for both methods, then tabulate and plot detection power to verify that PAIRADISE substantially outperforms Fisher's pooled test under the outlier scenario and recovers power as sample size grows; save results to simulation_outlier_power_comparison.csv.

## Related tools

- **PAIRADISE** (Replicate-aware statistical model under evaluation; run on each simulated dataset and significant events called at FDR <= 10%) — https://github.com/Xinglab/PAIRADISE
- **Fisher's exact test (pooled reads)** (Baseline comparison method run on reads pooled across replicates for the same simulated datasets)
- **nloptr** (R optimization package used by the PAIRADISE statistical model fitting)
- **R** (Language in which the PAIRADISE statistical model and power simulation are executed)
- **Python** (Used for generating simulated count data and tabulating/plotting power results)

## Examples

```
git clone github.com/Xinglab/PAIRADISE/pairadise; R CMD INSTALL -l /user/R/lib src/pairadise_model/
```

## Evaluation signals

- PAIRADISE substantially outperforms Fisher's pooled test in detection power under the injected-outlier scenario (outlier replicate PSI ~ Uniform[0,1])
- PAIRADISE's power recovers rapidly as the number of replicates increases, while the pooled strategy lags
- Significance calls use Benjamini-Hochberg adjustment at the default FDR <= 10% threshold for both methods
- Results are saved to simulation_outlier_power_comparison.csv with power values traceable across increasing replicate numbers
- The comparison runs on identical simulated datasets for both methods, differing only in how replicates are combined

## Limitations

- Only one outlier replicate configuration is tested (a single replicate PSI drawn from Uniform[0,1]); other outlier magnitudes or multiple outliers are not covered
- Fisher's pooled test is a naive baseline; the power gap may not generalize to more sophisticated pooling alternatives
- Results depend on the paper's specific simulation settings for allele-specific inclusion/skipping counts and effective isoform lengths

## Evidence
