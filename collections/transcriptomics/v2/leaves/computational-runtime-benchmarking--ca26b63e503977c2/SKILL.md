---
name: computational-runtime-benchmarking
description: Use when you need to quantify wall-clock runtime of Escort's three-step
  trajectory-inference workflow on simulated scRNA-seq datasets of varying sizes (e.g.,
  1000 vs 3000 cells, 20,000 genes) on laptop-class hardware.
when_to_use_negative:
- The goal is to evaluate embedding quality or trajectory accuracy rather than runtime
  — use Escort's benchmark scoring system (scores from negative infinity to two, embeddings
  scoring greater than zero recommended) instead of timing.
- The trajectory inference method does not use two-dimensional representations — Escort
  presently only considers approaches that use two-dimensional embeddings, so timing
  its Steps 2–3 is not applicable.
- You need to choose among trajectory inference methods for accuracy reasons — runtime
  benchmarking of Escort does not inform method selection (e.g., via dynguidelines).
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: Escort
  role: 'Framework being benchmarked: Step 1 (trajectory suitability testing, no embedding)
    and Steps 2–3 (embedding evaluation and trajectory fitting) are timed individually'
  repo: https://github.com/xiaorudong/Escort
- name: Scaffold
  role: R package v0.2.0 used to simulate the 1000- and 3000-cell, 20,000-gene scRNA-seq
    benchmark datasets with estimateScaffoldParameters() on the human pancreas reference
- name: Slingshot
  role: Default trajectory method invoked within Escort Steps 2–3 during per-embedding
    runtime measurement
- name: R
  role: Runtime environment in which Escort, Scaffold, and Slingshot are executed
    and timed
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bib/bbae216
    title: Data-driven selection of analysis decisions in single-cell RNA-seq trajectory
      inference
  source_content_hash: sha256:3838fb6a3426ebd87cb673dceb21d2d2c187735e8af15e08c5ec2b7d9f2740e1
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# computational-runtime-benchmarking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmarks the wall-clock computational cost of the Escort trajectory-inference workflow (Step 1 suitability testing; Steps 2–3 embedding evaluation and Slingshot trajectory fitting) on simulated scRNA-seq datasets of 1000 and 3000 cells with 20,000 genes, reporting mean and SD runtime in minutes across five repeats.

## When to use

Use when you need to quantify wall-clock runtime of Escort's three-step trajectory-inference workflow on simulated scRNA-seq datasets of varying sizes (e.g., 1000 vs 3000 cells, 20,000 genes) on laptop-class hardware.

## When NOT to use

- The goal is to evaluate embedding quality or trajectory accuracy rather than runtime — use Escort's benchmark scoring system (scores from negative infinity to two, embeddings scoring greater than zero recommended) instead of timing.
- The trajectory inference method does not use two-dimensional representations — Escort presently only considers approaches that use two-dimensional embeddings, so timing its Steps 2–3 is not applicable.
- You need to choose among trajectory inference methods for accuracy reasons — runtime benchmarking of Escort does not inform method selection (e.g., via dynguidelines).

## Inputs

- Simulated scRNA-seq count matrices of 1000 and 3000 cells with 20,000 genes, generated with Scaffold v0.2.0 using estimateScaffoldParameters() on the human pancreas single-cell RNA-seq reference
- A single two-dimensional cell embedding per dataset (input to Escort Steps 2–3, with Slingshot as the default trajectory method)
- Escort R package (installed from GitHub xiaorudong/Escort) and R runtime environment

## Outputs

- escort_computation_time.csv: table of mean and SD runtime (min) for Escort Step 1 and for Steps 2–3 (per single embedding) for each cell-count scenario (1000 and 3000 cells), aggregated over five repeats

## How to apply

Simulate scRNA-seq datasets with the Scaffold R package v0.2.0, configuring parameters with estimateScaffoldParameters() on the human pancreas single-cell RNA-seq reference to generate datasets of 1000 and 3000 cells with 20,000 genes each. Time Escort Step 1 (trajectory suitability testing) separately because it does not involve a specific embedding; expected order of magnitude is ~5 min for 1000 cells and ~18 min for 3000 cells. Time Steps 2 and 3 combined (embedding evaluation plus trajectory fitting, with Slingshot as the default trajectory method) per single embedding; expected per-embedding runtime is ~0.4–1.5 min for 1000 or 3000 cells, respectively. Repeat each scenario five times and compute both the mean runtime and the standard deviation in minutes for each cell count. Write the timing results for both scenarios to escort_computation_time.csv, reporting mean and SD (min) for Step 1 and for Steps 2–3 per scenario. Benchmark on laptop-class hardware (e.g., 16 GB RAM) and state the hardware, since timings are hardware-dependent.

## Related tools

- **Escort** (Framework being benchmarked: Step 1 (trajectory suitability testing, no embedding) and Steps 2–3 (embedding evaluation and trajectory fitting) are timed individually) — https://github.com/xiaorudong/Escort
- **Scaffold** (R package v0.2.0 used to simulate the 1000- and 3000-cell, 20,000-gene scRNA-seq benchmark datasets with estimateScaffoldParameters() on the human pancreas reference)
- **Slingshot** (Default trajectory method invoked within Escort Steps 2–3 during per-embedding runtime measurement)
- **R** (Runtime environment in which Escort, Scaffold, and Slingshot are executed and timed)

## Examples

```
remotes::install_github("xiaorudong/Escort"); library(Escort)
```

## Evaluation signals

- Output file escort_computation_time.csv exists and reports mean and SD runtime (min) for Step 1 and for Steps 2–3 for both the 1000-cell and 3000-cell scenarios across five repeats.
- Step 1 runtime was recorded independently of any embedding (no embedding supplied during Step 1 timing), while Steps 2–3 runtime was recorded for exactly one embedding per run.
- Timings fall in the reported reference ranges: Step 1 ≈ 5 min (1000 cells) and ≈ 18 min (3000 cells); Steps 2–3 ≈ 0.4–1.5 min per single embedding for 1000 or 3000 cells, respectively.
- Each scenario was repeated five times before aggregating, and both mean and standard deviation (in minutes) are present in the table.
- Simulated datasets match the specification: 1000 and 3000 cells with 20,000 genes each, generated via Scaffold v0.2.0 with estimateScaffoldParameters() on the human pancreas reference; hardware (16 GB RAM laptop) documented.

## Limitations

- Timings are hardware-dependent: the benchmark was run on laptop-class hardware with 16 GB RAM, so absolute minutes will differ on other machines.
- Runtime scales with cell number (Step 1: ~5 min at 1000 cells vs ~18 min at 3000 cells), so extrapolation to other dataset sizes should not be assumed linear.
- Steps 2–3 timing is per single embedding; total workflow time grows with the number of embeddings evaluated, and Escort is not guaranteed to identify a recommended embedding among the choices evaluated.

## Evidence

- [other] Escort's Step 1 took an average of 5 minutes on 1000-cell datasets and 18 minutes on 3000-cell datasets: [evidence span withheld — anchor status: not_found]
