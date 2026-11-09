---
name: memory-usage-profiling-of-single-cell-models
description: Use when comparing the scalability of scRNA-seq/TCR integration models
  across dataset sizes, or when planning GPU/RAM allocation for training on atlas-scale
  (>60k cells) single-cell multimodal data.
when_to_use_negative:
- When the goal is accuracy or embedding-quality benchmarking (e.g., antigen specificity
  prediction) rather than runtime/memory scalability
- When input data is unpaired or partially missing TCR data — mvTCR was mainly applied
  to paired αβ-CDR3 plus gene expression data
- When no GPU/CUDA-enabled machine is available, since mvTCR training requires GPU
  support
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: mvTCR
  role: Model under test; trained 20 epochs per subsample while recording wall-clock
    time and peak memory
  repo: https://github.com/SchubertLab/mvTCR
- name: tessa
  role: Baseline Bayesian method timed on the same subsamples to compute mvTCR speedup
    factors
- name: CoNGA
  role: Baseline clonotype neighbor-graph method timed on the same subsamples to compute
    mvTCR speedup factors
- name: mvTCR_reproducibility
  role: Repository with training/evaluation notebooks and settings to reproduce the
    runtime benchmarking experiments
  repo: https://github.com/SchubertLab/mvTCR_reproducibility
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-024-49806-9
    title: Multi-modal generative modeling for joint analysis of single-cell T cell
      receptor and gene expression data
  source_content_hash: sha256:788cf9b305f8a50c7c1504ce7f0082944abf316af76217558230e59693ca5880
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SchubertLab/mvTCR
  tools:
  - CoNGA
  - mvTCR
  - mvTCR_reproducibility
  - tessa
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# Memory-usage profiling of single-cell models

## Summary

Benchmarks how RAM requirements of single-cell deep learning models (e.g., mvTCR, tessa, CoNGA) scale with cell count, using subsampled TIL datasets and recording wall-clock time and peak memory per run.

## When to use

Use when comparing the scalability of scRNA-seq/TCR integration models across dataset sizes, or when planning GPU/RAM allocation for training on atlas-scale (>60k cells) single-cell multimodal data.

## When NOT to use

- When the goal is accuracy or embedding-quality benchmarking (e.g., antigen specificity prediction) rather than runtime/memory scalability
- When input data is unpaired or partially missing TCR data — mvTCR was mainly applied to paired αβ-CDR3 plus gene expression data
- When no GPU/CUDA-enabled machine is available, since mvTCR training requires GPU support

## Inputs

- TIL single-cell dataset with paired TCR and gene expression data
- Cell-count subsamples of the dataset at varying sizes (up to 100,000 cells)
- Conda environment with mvTCR (Python 3.10, pip install mvtcr) on a GPU/CUDA machine
- tessa and CoNGA installations for baseline runs

## Outputs

- timing_benchmark.csv with wall-clock time, peak memory, memory limits, and speedup factors per subsample size per tool
- Speedup factors of mvTCR relative to tessa and CoNGA at each subsample size

## How to apply

Set up the model environment in an isolated conda environment (mvTCR: Python 3.10, pip install mvtcr, on a GPU- and CUDA-enabled machine). Load the TIL dataset and construct cell-count subsamples at increasing sizes up to 100,000 cells, scaling the memory allocation with dataset size — 8 GB RAM sufficed for >60k cells in the mvTCR 10x reference. Train mvTCR on each subsample for a fixed protocol of 20 epochs, recording single-run wall-clock time and peak memory, then repeat under the same protocol for tessa and CoNGA. Compute speedup factors of mvTCR relative to each baseline per subsample size and write all timings, speedups, and memory limits to timing_benchmark.csv. Rationale: the article reports mvTCR as up to 135x faster than CoNGA (545 s vs 73,516 s at 100,000 cells) and up to 2,353-fold faster than tessa (166 s vs 389,578 s at 30,000 cells), so profiling memory and runtime versus cell count demonstrates scalability to atlas-scale integration.

## Related tools

- **mvTCR** (Model under test; trained 20 epochs per subsample while recording wall-clock time and peak memory) — https://github.com/SchubertLab/mvTCR
- **tessa** (Baseline Bayesian method timed on the same subsamples to compute mvTCR speedup factors)
- **CoNGA** (Baseline clonotype neighbor-graph method timed on the same subsamples to compute mvTCR speedup factors)
- **mvTCR_reproducibility** (Repository with training/evaluation notebooks and settings to reproduce the runtime benchmarking experiments) — https://github.com/SchubertLab/mvTCR_reproducibility

## Examples

```
conda create --name mvTCR python=3.10 -y && conda activate mvTCR && pip install mvtcr
```

## Evaluation signals

- timing_benchmark.csv exists and contains timings, speedups, and memory limits for every subsample size and all three tools
- mvTCR speedup versus CoNGA reaches on the order of 135x at 100,000 cells and versus tessa on the order of 2,353x at 30,000 cells
- mvTCR training on >60k cells completes within approximately 8 GB of RAM, matching the paper's reference memory footprint
- Each run uses a fixed protocol of 20 epochs so timings are comparable across tools and dataset sizes
- Runtime increases monotonically with subsample size for each tool

## Limitations

- Absolute runtimes differ across compute clusters, hardware components, and CUDA versions — reproducibility models are provided on Zenodo since retrained models may diverge
- 8 GB RAM sufficed for >60k cells in the paper, but required memory scales with dataset size, so larger datasets need proportionally more memory
- Hyperparameter optimization in the manuscript required up to 48 GPU-hours on machines with 32 GB or 512 GB memory; a quick benchmark on subsampled notebooks takes ~15 minutes but is not atlas-scale
- Only CDR3 sequence is used for TCR representation; results may not generalize to models incorporating VDJ-gene encodings or pre-trained TCR embeddings

## Evidence

- [methods] 8GBs of RAM were sufficient to train mvTCR on our preprocessed 10x dataset (>60k cells): "8GBs of RAM were sufficient to train mvTCR on our preprocessed 10x dataset (>60k cells)"
- [results] mvTCR is up to 135 times faster than CoNGA and up to 2,353-fold faster than tessa: [evidence span withheld — anchor status: not_found]
- [methods] conda create --name mvTCR python=3.10 -y && conda activate mvTCR && pip install mvtcr: "conda create --name mvTCR python=3.10 -y && conda activate mvTCR && pip install mvtcr"
- [readme] Based on different runtimes on the compute clusters, different hardware components, and CUDA-versions, the resulting models might diver: [evidence span withheld — anchor status: external_doc]
