---
name: dataset-subsampling-for-scalability-testing
description: Use when you must quantify how runtime and memory of a training method
  scale with dataset size, by training on successive cell-count subsamples and comparing
  against competing tools under a fixed protocol.
when_to_use_negative:
- When the goal is predictive performance (antigen specificity, clustering quality)
  rather than runtime/memory scalability — use mvTCR's evaluation notebooks instead
- When comparing methods on a single fixed dataset size with no subsampling — speedup-vs-size
  curves cannot be estimated
- On MAC with M1 chip or CPU-only machines, since GPU- and CUDA-support is heavily
  encouraged/required for mvTCR training
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: mvTCR
  role: Method under benchmark; trained on each subsample for 20 epochs to measure
    wall-clock time and memory
  repo: https://github.com/SchubertLab/mvTCR
- name: tessa
  role: Baseline method run on the same subsamples under the same protocol for runtime
    comparison
- name: CoNGA
  role: Baseline clonotype neighbor-graph method run on the same subsamples for runtime
    comparison
- name: uTILity
  role: Source of the tumor-infiltrating lymphocyte dataset with paired TCR sequencing
    used for subsampling
  repo: https://github.com/ncborcherding/utility
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-024-49806-9
    title: Multi-modal generative modeling for joint analysis of single-cell T cell
      receptor and gene expression data
  source_content_hash: sha256:7aab42398a4d2b1a2ccb4cacc21fac5194cb222446d15e76c5d0fefb213ea993
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SchubertLab/mvTCR
  tools:
  - CoNGA
  - mvTCR
  - tessa
  - uTILity
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# dataset-subsampling-for-scalability-testing

## Summary

Benchmarks a single training run of mvTCR against tessa and CoNGA on cell-count subsamples of a TIL dataset up to 100,000 cells, recording wall-clock time and peak memory to demonstrate scalability of atlas-scale multi-modal TCR-transcriptome integration.

## When to use

Use when you must quantify how runtime and memory of a training method scale with dataset size, by training on successive cell-count subsamples and comparing against competing tools under a fixed protocol.

## When NOT to use

- When the goal is predictive performance (antigen specificity, clustering quality) rather than runtime/memory scalability — use mvTCR's evaluation notebooks instead
- When comparing methods on a single fixed dataset size with no subsampling — speedup-vs-size curves cannot be estimated
- On MAC with M1 chip or CPU-only machines, since GPU- and CUDA-support is heavily encouraged/required for mvTCR training

## Inputs

- TIL single-cell dataset with paired gene expression and TCR sequencing (e.g. the utility TIL collection)
- Cell-count subsamples of the dataset (e.g. up to 100,000 cells)
- GPU- and CUDA-enabled compute environment with size-scaled RAM allocation

## Outputs

- timing_benchmark.csv with wall-clock time, peak memory, and speedup factors per subsample size per tool
- Reported speedup factors (e.g. mvTCR vs CoNGA and vs tessa) at each dataset size

## How to apply

Set up an isolated environment (conda create --name mvTCR python=3.10 -y, pip install mvtcr, then PyTorch with a fitting CUDA version) on a GPU- and CUDA-enabled machine. Load the TIL dataset and construct cell-count subsamples up to 100,000 cells, scaling RAM allocation with size — 8 GB sufficed for >60k cells in the paper's preprocessed 10x dataset. Train each tool on every subsample for a fixed protocol (20 epochs for mvTCR), recording single-run wall-clock time and peak memory for mvTCR, tessa, and CoNGA alike. Compute per-size speedup factors of mvTCR relative to tessa and CoNGA and write all timings, speedups, and memory limits to timing_benchmark.csv. Rationale: identical subsamples and protocol isolate runtime as the only varying factor, making the speedup comparison fair.

## Related tools

- **mvTCR** (Method under benchmark; trained on each subsample for 20 epochs to measure wall-clock time and memory) — https://github.com/SchubertLab/mvTCR
- **tessa** (Baseline method run on the same subsamples under the same protocol for runtime comparison)
- **CoNGA** (Baseline clonotype neighbor-graph method run on the same subsamples for runtime comparison)
- **uTILity** (Source of the tumor-infiltrating lymphocyte dataset with paired TCR sequencing used for subsampling) — https://github.com/ncborcherding/utility

## Examples

```
conda create --name mvTCR python=3.10 -y && conda activate mvTCR && pip install mvtcr
```

## Evaluation signals

- timing_benchmark.csv exists and contains wall-clock time, peak memory, and computed speedup factors for every subsample size × tool combination
- Speedup factors are internally consistent: tool runtime at each size divided by mvTCR runtime at that size matches the reported factors (e.g. ~135× vs CoNGA at 100k cells, ~2,353× vs tessa at 30k cells)
- All three tools ran on identical subsamples under the same fixed protocol (e.g. 20 epochs for mvTCR), so the comparison is apples-to-apples
- Peak memory stays within the allocated budget per subsample size (reference: 8 GB RAM sufficed for >60k cells)

## Limitations

- Benchmark timings depend on hardware and CUDA versions; results on different compute clusters may diverge, as noted in the reproducibility repo
- mvTCR applied mainly to fully paired data (gene expression with α- and β-CDR3); partially missing data is insufficiently addressed
- Baseline tools (tessa, CoNGA) are far slower, so large subsamples may be infeasible for them; the 2,353× tessa comparison is at only 30,000 cells

## Evidence

- [results] mvTCR is up to 135 times faster than CoNGA (545 s vs 73,516 s at 100,000 cells): [evidence span withheld — anchor status: not_found]
- [readme] 8 GB RAM reference: "As a reference, 8GBs of RAM were sufficient to train mvTCR on our preprocessed 10x dataset (>60k cells)."
- [methods] environment setup command: "conda create --name mvTCR python=3.10 -y && conda activate mvTCR && pip install mvtcr"
- [readme] GPU requirement: "To train mvTCR a machine with GPU support is required."
