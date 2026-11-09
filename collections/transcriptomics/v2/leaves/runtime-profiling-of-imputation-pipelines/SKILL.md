---
name: runtime-profiling-of-imputation-pipelines
description: Use when imputation methods must be benchmarked for scalability on tens
  of thousands of cells (e.g., 62,893-cell, 91.9%-dropout droplet data) and a wall-clock
  runtime ordering is required alongside accuracy scores.
when_to_use_negative:
- The dataset is small or low-dropout, where runtime differences among imputation
  methods are not a meaningful selection criterion
- Only one imputation method is used, so no cross-method runtime comparison or ordering
  can be established
- The analysis goal is imputation accuracy alone (t-SNE, pseudotime, clustering) without
  computational feasibility constraints
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: Bfimpute
  role: Imputation method being profiled; run in labeled mode with defaults D=32,
    totalepoch=300, burnin=200, ncores=5 while timing the run
  repo: github.com/maiziezhoulab/Bfimpute
- name: scImpute
  role: Comparison imputation method run in labeled mode; its wall-clock time is recorded
    for the runtime ordering
- name: DrImpute
  role: Comparison imputation method; fails on the Vladoiu-scale dataset and is recorded
    as a failure rather than a runtime
- name: SAVER
  role: Comparison imputation method run in labeled mode; expected to be the slowest
    successful method in the runtime ordering
- name: MAGIC
  role: Comparison imputation method; expected to be the fastest, defining the lower
    bound of the runtime ordering
- name: VIPER
  role: Comparison imputation method; fails on the Vladoiu-scale dataset and is recorded
    as a failure rather than a runtime
- name: PBLR
  role: Comparison imputation method run in labeled mode; placed between MAGIC and
    Bfimpute in the runtime ordering
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.crmeth.2021.100133
    title: A Bayesian factorization method to recover single-cell RNA sequencing data
  source_content_hash: sha256:721b7f268c1ad408007329ccd141317dc4381d9682cb75a206d93cf161a2c714
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/maiziezhoulab/Bfimpute
  tools:
  - Bfimpute
  - DrImpute
  - MAGIC
  - PBLR
  - SAVER
  - VIPER
  - scImpute
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# Runtime Profiling of Imputation Pipelines

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Wall-clock profiling of competing scRNA-seq imputation methods (Bfimpute, scImpute, DrImpute, SAVER, MAGIC, VIPER, PBLR) on a large, high-dropout dataset to establish computational feasibility and a reproducible runtime ordering alongside accuracy metrics.

## When to use

Use when imputation methods must be benchmarked for scalability on tens of thousands of cells (e.g., 62,893-cell, 91.9%-dropout droplet data) and a wall-clock runtime ordering is required alongside accuracy scores.

## When NOT to use

- The dataset is small or low-dropout, where runtime differences among imputation methods are not a meaningful selection criterion
- Only one imputation method is used, so no cross-method runtime comparison or ordering can be established
- The analysis goal is imputation accuracy alone (t-SNE, pseudotime, clustering) without computational feasibility constraints

## Inputs

- Raw 10x scRNA-seq count matrix (genes x cells) from the Vladoiu mouse cerebellum dataset (GSE118068; 62,893 cells, 91.9% dropout)
- Cell type label vector/data frame for labeled-mode imputation
- Run logs or timers for Bfimpute, scImpute, DrImpute, SAVER, MAGIC, VIPER, PBLR

## Outputs

- Wall-clock runtime table with one row per imputation method
- Combined score table (TSCAN POS/Kendall, four clustering metrics) plus runtimes saved as vladoiu_imputation_benchmark_results.csv
- Recorded method failures (DrImpute and VIPER expected to fail on this dataset)

## How to apply

Run each imputation method on the same raw count matrix under its default settings, recording wall-clock time per method. For Bfimpute, supply known cell labels via ccluster='labeled' with label, and defaults D=32, totalepoch=300, burnin=200, sn_max=10, sn_init=1, threshold=0.5, ncores=5; run scImpute, DrImpute, SAVER, and PBLR in labeled mode, and MAGIC and VIPER with defaults. Record not only timings but also which methods fail outright (DrImpute and VIPER fail on the Vladoiu-scale input). Verify the reproduced runtime ordering MAGIC < PBLR < Bfimpute < scImpute < SAVER, then assemble timings together with TSCAN pseudotime (POS, Kendall) and clustering scores into vladoiu_imputation_benchmark_results.csv matching Figure S6.

## Related tools

- **Bfimpute** (Imputation method being profiled; run in labeled mode with defaults D=32, totalepoch=300, burnin=200, ncores=5 while timing the run) — github.com/maiziezhoulab/Bfimpute
- **scImpute** (Comparison imputation method run in labeled mode; its wall-clock time is recorded for the runtime ordering)
- **DrImpute** (Comparison imputation method; fails on the Vladoiu-scale dataset and is recorded as a failure rather than a runtime)
- **SAVER** (Comparison imputation method run in labeled mode; expected to be the slowest successful method in the runtime ordering)
- **MAGIC** (Comparison imputation method; expected to be the fastest, defining the lower bound of the runtime ordering)
- **VIPER** (Comparison imputation method; fails on the Vladoiu-scale dataset and is recorded as a failure rather than a runtime)
- **PBLR** (Comparison imputation method run in labeled mode; placed between MAGIC and Bfimpute in the runtime ordering)

## Examples

```
library(Bfimpute); t0 <- Sys.time(); counts_imputed <- Bfimpute(counts, ccluster = "labeled", label = cell_type, ncores = 5); print(Sys.time() - t0)
```

## Evaluation signals

- Reproduced runtime ordering matches the reported MAGIC < PBLR < Bfimpute < scImpute < SAVER
- DrImpute and VIPER fail on the 62,893-cell input rather than completing, consistent with Figure S6
- Wall-clock time is recorded for every method that completes, with no missing timing entries for successful runs
- Combined score and runtime table reproduces Figure S6 when assembled into vladoiu_imputation_benchmark_results.csv
- Bfimpute runs with the documented defaults (D=32, totalepoch=300, burnin=200, ncores=5), so timing reflects the recommended configuration

## Limitations

- There is no consensus about which imputation method is the best; runtime efficiency does not imply accuracy across all criteria, protocols, and downstream analyses
- It is premature to conclude that Bfimpute will achieve better performance across all datasets, so runtime orderings may not transfer to other inputs
- Larger D, totalepoch, and burnin increase accuracy but increase runtime, so profiled timings depend on the chosen hyperparameters
- Depending on the application, users may need to constrain parameters, which alters both runtime and false-positive behavior

## Evidence
