---
name: repeated-measure-runtime-aggregation
description: Use when you need to quantify computational time requirements of a scRNA-seq
  trajectory-inference workflow (e.g., Escort) across dataset sizes such as 1000 vs
  3000 cells, and report reproducible per-step runtime statistics.
when_to_use_negative:
- When the goal is accuracy or trajectory-quality evaluation rather than runtime measurement
  — Escort's scoring system (negative infinity to two, scores > 0 recommended) addresses
  suitability, not speed.
- When the trajectory method does not use two-dimensional representations, since Escort
  only considers trajectory inference approaches using two-dimensional representations.
- When timing end-to-end pipelines with variable numbers of embeddings — this skill
  times Steps 2-3 for a single embedding, so multi-embedding sweeps need per-embedding
  aggregation instead.
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_2229
tools:
- name: Escort
  role: Framework whose Step 1 (trajectory suitability testing) and Steps 2-3 (embedding
    evaluation and trajectory fitting) are timed in this benchmark
  repo: https://github.com/xiaorudong/Escort
- name: R
  role: Implementation environment for Scaffold simulation and Escort timing runs
- name: Scaffold
  role: R package v0.2.0 used with estimateScaffoldParameters() to simulate 1000-
    and 3000-cell scRNA-seq datasets with 20,000 genes from the human pancreas reference
- name: Slingshot
  role: Default trajectory method used during Escort Steps 2-3 when timing per-embedding
    runtime
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bib/bbae216
    title: Data-driven selection of analysis decisions in single-cell RNA-seq trajectory
      inference
  source_content_hash: sha256:b0dc0798d0439d8d9cc1cbec57e1c2b04d0168bdf3ce8d9835761fc6debab22e
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/xiaorudong/Escort
  repo_url: https://github.com/xiaorudong/Escort
  tools:
  - Escort
  - R
  - Scaffold
  - Slingshot
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# repeated-measure-runtime-aggregation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark the computational cost of Escort's trajectory-inference workflow by timing Step 1 (trajectory suitability testing) independently of any embedding and Steps 2-3 (embedding evaluation plus trajectory fitting) per single embedding, repeating each scenario five times and reporting mean and SD runtime in minutes.

## When to use

Use when you need to quantify computational time requirements of a scRNA-seq trajectory-inference workflow (e.g., Escort) across dataset sizes such as 1000 vs 3000 cells, and report reproducible per-step runtime statistics.

## When NOT to use

- When the goal is accuracy or trajectory-quality evaluation rather than runtime measurement — Escort's scoring system (negative infinity to two, scores > 0 recommended) addresses suitability, not speed.
- When the trajectory method does not use two-dimensional representations, since Escort only considers trajectory inference approaches using two-dimensional representations.
- When timing end-to-end pipelines with variable numbers of embeddings — this skill times Steps 2-3 for a single embedding, so multi-embedding sweeps need per-embedding aggregation instead.

## Inputs

- Simulated scRNA-seq datasets (Scaffold R package v0.2.0, estimateScaffoldParameters() on human pancreas reference) of 1000 and 3000 cells with 20,000 genes each
- Escort R package installation (github.com/xiaorudong/Escort) with Slingshot as default trajectory method
- Laptop-class hardware configuration (e.g., 16 GB RAM) for consistent timing

## Outputs

- escort_computation_time.csv reporting mean and SD runtime (min) for Escort Step 1 and for Steps 2-3 per embedding, per scenario, across five repeats

## How to apply

First simulate scRNA-seq datasets with the Scaffold R package v0.2.0, configuring parameters with estimateScaffoldParameters() on the human pancreas reference to generate 1000-cell and 3000-cell datasets with 20,000 genes each. Time Escort Step 1 (trajectory suitability testing) on each simulated dataset, keeping this timing independent of any specific embedding. Time Steps 2 and 3 combined (embedding evaluation and trajectory fitting with Slingshot as the default trajectory method) separately, recording the per-embedding runtime for one embedding per dataset. Repeat each scenario five times, then compute both the mean runtime and the standard deviation in minutes for each cell count and write the results to a CSV (e.g., escort_computation_time.csv).

## Related tools

- **Escort** (Framework whose Step 1 (trajectory suitability testing) and Steps 2-3 (embedding evaluation and trajectory fitting) are timed in this benchmark) — https://github.com/xiaorudong/Escort
- **R** (Implementation environment for Scaffold simulation and Escort timing runs)
- **Scaffold** (R package v0.2.0 used with estimateScaffoldParameters() to simulate 1000- and 3000-cell scRNA-seq datasets with 20,000 genes from the human pancreas reference)
- **Slingshot** (Default trajectory method used during Escort Steps 2-3 when timing per-embedding runtime)

## Examples

```
library(Escort); # time Step 1 on simulated data, then Steps 2-3 per embedding with Slingshot; timings in min over 5 repeats
```

## Evaluation signals

- Output CSV contains mean and SD runtime (min) for both Step 1 and Steps 2-3, for each cell-count scenario, computed over five repeats.
- Step 1 timings were recorded independently of any embedding, while Steps 2-3 timings correspond to a single embedding per dataset.
- Reported runtimes fall within the reference ranges: Step 1 ~5 min at 1000 cells and ~18 min at 3000 cells; Steps 2-3 ~0.4-1.5 min per single embedding for 1000-3000 cells.
- Simulation parameters match the benchmark setup: Scaffold v0.2.0, estimateScaffoldParameters() on the human pancreas reference, 20,000 genes per dataset.

## Limitations

- Benchmarked on laptop-class hardware (16 GB RAM); absolute timings will differ on other machines.
- Escort is limited to trajectory inference approaches using two-dimensional representations.
- Escort is not guaranteed to identify a recommended embedding among the evaluated choices, which affects how many per-embedding timings may be needed in practice.

## Evidence

- [results] Escort's Step 1 took an average of 5 minutes on 1000-cell datasets and 18 minutes on 3000-cell datasets, while embedding evaluation in Steps 2 and 3 took between 0.4 and 1.5 minutes per single embedding: [evidence span withheld — anchor status: not_found]
