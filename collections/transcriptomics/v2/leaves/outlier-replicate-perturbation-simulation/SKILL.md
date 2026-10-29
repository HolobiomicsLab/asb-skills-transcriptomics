---
name: outlier-replicate-perturbation-simulation
description: Use when evaluating whether a replicate-aware count model (e.g., PAIRADISE
  paired model) is more robust than a naive pooled-read baseline under outlier contamination,
  or when tracing power-versus-number-of-replicates curves for differential splicing
  detection.
when_to_use_negative:
- When the goal is to run the PAIRADISE pipeline on real RNA-seq FASTQ data (personalization,
  mapping, annotation, counting) — this skill is a simulation-based robustness benchmark,
  not the data-processing pipeline.
- 'When there is no replicate structure to perturb (single-sample analysis): PAIRADISE''s
  advantage comes from aggregating signals across multiple individuals sharing a heterozygous
  SNP, which is the setup being simulated.'
- When the question is detecting ASAS events themselves rather than benchmarking detection
  methods under outlier contamination.
edam_operation: http://edamontology.org/operation_0244
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3300
tools:
- name: PAIRADISE
  role: Paired replicate statistical model under evaluation; run on each simulated
    dataset with nloptr optimization to call significant events at FDR <= 10%
  repo: github.com/Xinglab/PAIRADISE
- name: nloptr
  role: R optimization backend used by the PAIRADISE paired statistical model when
    fitting simulated count data
- name: doParallel / foreach
  role: R parallelization over simulated datasets when running the PAIRADISE model
- name: Fisher's exact test (pooled reads)
  role: 'Baseline comparison method: applied to reads pooled across replicates of
    the same simulated datasets'
- name: R
  role: Implementation environment for the PAIRADISE statistical model and simulation
    scripts
- name: Python
  role: Environment for generating simulated count data and tabulating/plotting the
    power comparison
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.ajhg.2020.07.005
    title: Detecting Allele-Specific Alternative Splicing from Population-Scale RNA-Seq
      Data
  source_content_hash: sha256:13f50e76427141ff512a931a399ee77f214da8d58979ba5a69163673da8a8c4d
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/Xinglab/PAIRADISE
  tools:
  - Fisher's exact test (pooled reads)
  - PAIRADISE
  - Python
  - R
  - doParallel / foreach
  - nloptr
  edam_operation: http://edamontology.org/operation_0244
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3300
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# outlier-replicate-perturbation-simulation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A benchmarking simulation that injects an outlier replicate (one replicate PSI drawn from Uniform[0,1]) into simulated allele-specific splicing count data to show that PAIRADISE's paired replicate model is substantially more robust than a Fisher's exact test on pooled reads for differential AS detection.

## When to use

Use when evaluating whether a replicate-aware count model (e.g., PAIRADISE paired model) is more robust than a naive pooled-read baseline under outlier contamination, or when tracing power-versus-number-of-replicates curves for differential splicing detection.

## When NOT to use

- When the goal is to run the PAIRADISE pipeline on real RNA-seq FASTQ data (personalization, mapping, annotation, counting) — this skill is a simulation-based robustness benchmark, not the data-processing pipeline.
- When there is no replicate structure to perturb (single-sample analysis): PAIRADISE's advantage comes from aggregating signals across multiple individuals sharing a heterozygous SNP, which is the setup being simulated.
- When the question is detecting ASAS events themselves rather than benchmarking detection methods under outlier contamination.

## Inputs

- Simulated allele-specific alternative splicing count data (per-replicate, per-allele exon inclusion and skipping counts, i.e., IJC/SJC)
- Effective isoform lengths (incLen, skpLen) matching the PAIRADISE output schema
- Replicate-number grid for the power-versus-sample-size sweep
- PAIRADISE repository scripts (simulation and paired statistical-model scripts in the scripts directory)

## Outputs

- simulation_outlier_power_comparison.csv tabulating detection power for PAIRADISE vs. Fisher's pooled test
- Power-versus-sample-size curves for both methods under the outlier scenario
- Per-dataset significant-event tables with raw p-values and Benjamini-Hochberg-adjusted FDR values (threshold FDR <= 10%)

## How to apply

Clone github.com/Xinglab/PAIRADISE and locate the simulation and statistical-model scripts in the scripts directory. Generate simulated allele-specific alternative splicing count data — per-allele exon inclusion (IJC) and skipping (SJC) counts with effective isoform lengths (incLen, skpLen) — using the paper's simulation settings, injecting one outlier replicate whose PSI is drawn from Uniform[0,1]. Run the PAIRADISE paired statistical model in R (nloptr optimization, parallelized with doParallel/foreach) on each simulated dataset and call significant events at the default FDR <= 10% using Benjamini-Hochberg adjustment. As the comparison baseline, run Fisher's exact test on reads pooled across replicates for the same datasets. Repeat across increasing numbers of replicates, tabulate detection power for both methods, and verify PAIRADISE outperforms Fisher's pooled test by a substantial margin and recovers power as sample size increases.

## Related tools

- **PAIRADISE** (Paired replicate statistical model under evaluation; run on each simulated dataset with nloptr optimization to call significant events at FDR <= 10%) — github.com/Xinglab/PAIRADISE
- **nloptr** (R optimization backend used by the PAIRADISE paired statistical model when fitting simulated count data)
- **doParallel / foreach** (R parallelization over simulated datasets when running the PAIRADISE model)
- **Fisher's exact test (pooled reads)** (Baseline comparison method: applied to reads pooled across replicates of the same simulated datasets)
- **R** (Implementation environment for the PAIRADISE statistical model and simulation scripts)
- **Python** (Environment for generating simulated count data and tabulating/plotting the power comparison)

## Examples

```
qsub -cwd -l h_vmem=10G qsub_files/pairadise_step5_stat.qsub
```

## Evaluation signals

- PAIRADISE achieves substantially higher detection power than Fisher's pooled test across all simulated datasets containing the injected outlier replicate.
- The power gap narrows as the number of replicates increases — PAIRADISE's power recovers with sample size, reproducing the paper's power-versus-sample-size curve.
- All significance calls use Benjamini-Hochberg adjustment at the default FDR <= 10%, matching PAIRADISE's default output threshold.
- The injected outlier is constructed so exactly one replicate's PSI is drawn from Uniform[0,1], matching the paper's simulation setting.
- Results are written to simulation_outlier_power_comparison.csv with one power value per method per replicate count.

## Limitations

- Simulation-based result: the outlier is modeled as a replicate PSI drawn from Uniform[0,1]; real outlier structure may differ.
- The comparison is specific to the outlier scenario and increasing-replicate sweep described in the task card; other perturbation regimes (e.g., correlated noise across replicates) are not covered.
- The underlying PAIRADISE model shares similarities with the rMATS paired model, whose unreliable correlation-parameter estimation can lead to inflated p-values — a caveat when extending conclusions to other paired models.
- Results reflect PAIRADISE's conservative default; running with the sigma_i1 = sigma_i2 user option would increase the number of significant events (e.g., 116 to 133 in GM12878).

## Evidence
