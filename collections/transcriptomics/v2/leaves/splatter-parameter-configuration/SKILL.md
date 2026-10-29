---
name: splatter-parameter-configuration
description: Use when you need controlled, simulated two-batch scRNA-seq count matrices
  to benchmark batch-effect correction tools, and you must reproduce a specific effect
  design (large batch shift, elevated dropout, unequal batch sizes, small batch effects,
  or multi-cell-type structure).
when_to_use_negative:
- The goal is real-data integration, not controlled simulation — Splatter output cannot
  substitute for real pancreatic (Muraro CEL-Seq2 / Segerstolpe Smart-Seq2) or 10X
  PBMC datasets used in the paper's real-data experiments
- You need to simulate more than two batches per dataset, since the benchmark design
  and BATMAN's implementation are restricted to two-dataset (pairwise) integration
- A simulator with a different generative model (e.g. zero-inflated or Gaussian) is
  required; Splatter here specifically uses the gamma-Poisson model
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3524
- http://edamontology.org/topic_3170
tools:
- name: Splatter
  role: Simulator used to generate two-batch scRNA-seq count data under the gamma-Poisson
    model with per-scenario parameter settings (batch.facLoc/batch.facScale, dropout,
    batch sizes)
- name: BATMAN
  role: Integration method whose simulation configuration and scenario scripts define
    the parameter values; its repository hosts the benchmark setup being reproduced
  repo: https://github.com/mandricigor/batman
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.isci.2020.101185
    title: 'BATMAN: Fast and Accurate Integration of Single-Cell RNA-Seq Datasets
      via Minimum-Weight Matching'
  source_content_hash: sha256:13621820ad4cfa73837e0321fd98b8bafd986c9571f57dd0e13efc6140bb9523
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/mandricigor/batman
  repo_url: https://github.com/mandricigor/batman
  tools:
  - BATMAN
  - Splatter
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3524
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# splatter-parameter-configuration

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Configures the Splatter simulator to generate paired-batch scRNA-seq count data under a gamma-Poisson model for benchmarking batch-integration methods, replicating the BATMAN paper's five scenarios (LB, LB-DR, LB-UB, SB, LB-CT) with 1,000 genes per dataset.

## When to use

Use when you need controlled, simulated two-batch scRNA-seq count matrices to benchmark batch-effect correction tools, and you must reproduce a specific effect design (large batch shift, elevated dropout, unequal batch sizes, small batch effects, or multi-cell-type structure).

## When NOT to use

- The goal is real-data integration, not controlled simulation — Splatter output cannot substitute for real pancreatic (Muraro CEL-Seq2 / Segerstolpe Smart-Seq2) or 10X PBMC datasets used in the paper's real-data experiments
- You need to simulate more than two batches per dataset, since the benchmark design and BATMAN's implementation are restricted to two-dataset (pairwise) integration
- A simulator with a different generative model (e.g. zero-inflated or Gaussian) is required; Splatter here specifically uses the gamma-Poisson model

## Inputs

- Splatter simulation parameter set (scenario configuration: batch.facLoc, batch.facScale, dropout rate, batch cell counts, gene count)
- Scenario scripts from the BATMAN repository (github.com/mandricigor/batman)

## Outputs

- One simulated two-batch count-matrix file per scenario (CSV/TSV), each with 1,000 genes
- 100 simulated dataset pairs per scenario (LB, LB-DR, LB-UB, SB, LB-CT)

## How to apply

Load the simulation configuration and scenario scripts from the BATMAN repository (github.com/mandricigor/batman), then call Splatter to simulate two-batch count data under the gamma-Poisson (negative-binomial) model with 1,000 genes. Set per-scenario Splatter parameters: large batch effects use batch.facLoc=0.5 and batch.facScale=0.5, small batch effects use 0.001 for both, the LB-DR scenario raises the dropout rate, and the unequal-batch-size scenario (LB-UB) generates batches of 200 and 1,000 cells; the LB-CT variant embeds two cell types. Generate 100 simulated dataset pairs per scenario, as in the BATMAN benchmark, and save each scenario as a separate count-matrix file (CSV/TSV) with both batches for downstream integration testing.

## Related tools

- **Splatter** (Simulator used to generate two-batch scRNA-seq count data under the gamma-Poisson model with per-scenario parameter settings (batch.facLoc/batch.facScale, dropout, batch sizes))
- **BATMAN** (Integration method whose simulation configuration and scenario scripts define the parameter values; its repository hosts the benchmark setup being reproduced) — https://github.com/mandricigor/batman

## Examples

```
library(splatter); params <- newSplatParams(nGenes=1000); params <- setParams(params, batch.facLoc=0.5, batch.facScale=0.5); sim <- splatSimulate(params, group.cells=c(1000,1000), method='groups')
```

## Evaluation signals

- Each scenario output file exists and contains exactly two batches
- Each dataset contains 1,000 genes and each scenario is replicated 100 times, matching the paper's setup
- Designed condition is qualitatively present: e.g. elevated dropout in LB-DR, unequal batch sizes of 200 and 1,000 cells in LB-UB, visible batch shift in LB but not in SB
- Large-batch scenarios (LB, LB-DR, LB-UB, LB-CT) use batch.facLoc/batch.facScale = 0.5 while the SB scenario uses 0.001 — verify parameter values in the saved configuration

## Limitations

- Splatter simulations follow a gamma-Poisson generative model and may not capture all real batch-effect structure, which is why the paper also benchmarks on real datasets (Muraro, Segerstolpe, 10X PBMC 8k, 10X Pan T cells)
- Simulated data support only two-batch designs, consistent with BATMAN's restriction to two datasets
- No changelog found for the BATMAN repository, so scenario scripts may have changed since the paper

## Evidence

- [results] large batch effects use batch.facLoc/batch.facScale of 0.5, small batch effects use 0.001, and unequal-size batches contain 200 and 1,000 cells.: [evidence span withheld — anchor status: not_found]
- [results] The simulations comprise five scenarios (LB, LB-DR, LB-UB, SB, LB-CT) with 1,000 genes each: [evidence span withheld — anchor status: not_found]
