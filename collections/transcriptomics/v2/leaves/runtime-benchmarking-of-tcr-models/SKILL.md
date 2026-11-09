---
name: runtime-benchmarking-of-tcr-models
description: Use when comparing computational scalability of TCR-transcriptome integration
  methods (mvTCR, CoNGA, tessa) on datasets of varying size, or when deciding whether
  a model can scale to atlas-level (>60k cell) immune profiling data.
when_to_use_negative:
- The goal is to compare embedding quality (antigen specificity prediction, clustering
  ARI) rather than computational cost — use accuracy/cluster metrics instead of wall-clock
  benchmarking
- The hardware has no GPU/CUDA support — mvTCR requires a GPU machine, so timing comparisons
  would not reflect the intended benchmark setup
- Datasets lack paired TCR and transcriptome data, since CoNGA, tessa, and mvTCR all
  require both modalities
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_0091
tools:
- name: mvTCR
  role: Model under benchmark; multi-view VAE whose single-run training time and memory
    are measured on each subsample
  repo: https://github.com/SchubertLab/mvTCR
- name: tessa
  role: Baseline comparator; Bayesian TCR functional landscape model timed on the
    same subsamples
- name: CoNGA
  role: Baseline comparator; clonotype neighbor-graph analysis timed on the same subsamples
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-024-49806-9
    title: Multi-modal generative modeling for joint analysis of single-cell T cell
      receptor and gene expression data
  source_content_hash: sha256:59171c40ba5d203eed37df8acb38bb144f83a1d7fb0ad2195e075880d69d6e90
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SchubertLab/mvTCR
  tools:
  - CoNGA
  - mvTCR
  - tessa
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# runtime-benchmarking-of-tcr-models

## Summary

Benchmark single-run training wall-clock time and peak memory of mvTCR against CoNGA and tessa on cell-count subsamples of a paired TCR/scRNA-seq dataset, quantifying scalability of multi-modal TCR-transcriptome integration methods.

## When to use

Use when comparing computational scalability of TCR-transcriptome integration methods (mvTCR, CoNGA, tessa) on datasets of varying size, or when deciding whether a model can scale to atlas-level (>60k cell) immune profiling data.

## When NOT to use

- The goal is to compare embedding quality (antigen specificity prediction, clustering ARI) rather than computational cost — use accuracy/cluster metrics instead of wall-clock benchmarking
- The hardware has no GPU/CUDA support — mvTCR requires a GPU machine, so timing comparisons would not reflect the intended benchmark setup
- Datasets lack paired TCR and transcriptome data, since CoNGA, tessa, and mvTCR all require both modalities

## Inputs

- Paired TCR-seq / scRNA-seq dataset (e.g., TIL dataset) with gene expression and CDR3 sequences
- Cell-count subsamples (random subsampling up to 100,000 cells)
- GPU- and CUDA-enabled compute environment with conda

## Outputs

- timing_benchmark.csv with per-size wall-clock times, speedup factors, and memory limits for mvTCR, tessa, and CoNGA
- Peak-memory measurements per model per subsample size

## How to apply

Create a conda environment (`conda create --name mvTCR python=3.10 -y && conda activate mvTCR && pip install mvtcr`) on a GPU- and CUDA-enabled machine. Load a paired TCR/scRNA-seq dataset (e.g., the TIL dataset) and construct cell-count subsamples up to 100,000 cells by random subsampling. Train mvTCR on each subsample for a fixed protocol of 20 epochs, recording single-run wall-clock time and peak memory; run tessa and CoNGA on identical subsamples under the same protocol. Scale memory allocation with dataset size — 8 GB RAM sufficed for >60k cells in the paper's preprocessed 10x dataset. Compute speedup factors of mvTCR relative to tessa and CoNGA per subsample size and write timings, speedups, and memory limits to a CSV (timing_benchmark.csv).

## Related tools

- **mvTCR** (Model under benchmark; multi-view VAE whose single-run training time and memory are measured on each subsample) — https://github.com/SchubertLab/mvTCR
- **tessa** (Baseline comparator; Bayesian TCR functional landscape model timed on the same subsamples)
- **CoNGA** (Baseline comparator; clonotype neighbor-graph analysis timed on the same subsamples)

## Examples

```
conda create --name mvTCR python=3.10 -y && conda activate mvTCR && pip install mvtcr && conda install nb_conda_kernels -y
```

## Evaluation signals

- Reported mvTCR runtime at 100,000 cells should be far below CoNGA's (paper: 545 s vs 73,516 s, ~135x speedup)
- Reported mvTCR runtime at 30,000 cells should be far below tessa's (paper: 166 s vs 389,578 s, ~2,353-fold speedup)
- All three methods run on identical subsamples with an identical fixed training protocol (20 epochs) so timings are comparable
- timing_benchmark.csv contains rows for every subsample size with wall-clock time, speedup factors, and memory columns populated
- Peak memory stays within the allocated limit per size (reference: 8 GB RAM sufficient for >60k cells)

## Limitations

- Timings are hardware-dependent: differing runtimes, compute clusters, CUDA versions, and GPU models affect results, as noted in the reproducibility README
- Memory requirements scale with dataset size; subsampling design and RAM allocation must grow with cell count
- The benchmark measures a single training run under a fixed epoch protocol and does not reflect runtime of inference, reference mapping, or repeated training for modality-contribution tuning

## Evidence

- [results] mvTCR is up to 135 times faster than CoNGA (545 s vs 73,516 s at 100,000 cells) and up to 2,353-fold faster than tessa (166 s vs 389,578 s at 30,000 cells): [evidence span withheld — anchor status: not_found]
- [methods] conda create --name mvTCR python=3.10 -y && conda activate mvTCR && pip install mvtcr: "conda create --name mvTCR python=3.10 -y && conda activate mvTCR && pip install mvtcr"
- [readme] As a reference, 8GBs of RAM were sufficient to train mvTCR on our preprocessed 10x dataset (>60k cells).: "The required memory scales with the dataset size. As a reference, 8GBs of RAM were sufficient to train mvTCR on our preprocessed 10x dataset (>60k cells)."
- [readme] To train mvTCR a machine with GPU support is required.: "To train mvTCR a machine with GPU support is required."
