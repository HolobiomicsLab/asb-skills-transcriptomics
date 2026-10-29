---
name: linear-scaling-regression-analysis
description: 'Use when a method''s scalability claim (e.g., ''linear running time'')
  must be verified empirically: you can generate or obtain datasets at multiple sizes
  and record wall-clock runtime per size for regression.'
when_to_use_negative:
- The goal is clustering accuracy rather than computational scalability — use the
  paper's ARI/NMI benchmarking experiments instead
- Only a single dataset size is available, so no runtime-versus-size regression can
  be fitted
- The method must be benchmarked on CPU-only systems — the article notes all scMDC
  experiments require a Nvidia Tesla P100 GPU, so runtime comparisons would not be
  reproducible
tools:
- name: SymSim
  role: R package used to generate simulated single-cell multi-omics datasets at each
    cell count for the scalability series
- name: scMDC
  role: Method under test whose running time is recorded at each simulated dataset
    size
- name: PyTorch
  role: Deep learning framework underlying scMDC's GPU training timed in the benchmark
- name: h5py
  role: Used to export each simulated dataset in H5 format with the X1 (RNA) / X2
    (ADT) structure required by scMDC
- name: Scanpy
  role: Preprocessing/normalization of mRNA and ADT data before running scMDC on each
    simulated dataset
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-022-35031-9
    title: Clustering of single-cell multi-omics data with a multimodal deep learning
      method
  source_content_hash: sha256:23d73ab5dba78b8a62892a0cc28878678d4676ca2dd2804b0c3a2b0485a47aed
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PyTorch
  - Scanpy
  - SymSim
  - h5py
  - scMDC
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# linear-scaling-regression-analysis

## Summary

Benchmark empirical wall-clock runtime of a method (here, scMDC) across a log-spaced series of simulated dataset sizes and fit a linear model of runtime versus cell count to verify linear scalability, as scMDC does from 1,000 to 100,000 CITE-seq cells on a Tesla P100 GPU.

## When to use

Use when a method's scalability claim (e.g., 'linear running time') must be verified empirically: you can generate or obtain datasets at multiple sizes and record wall-clock runtime per size for regression.

## When NOT to use

- The goal is clustering accuracy rather than computational scalability — use the paper's ARI/NMI benchmarking experiments instead
- Only a single dataset size is available, so no runtime-versus-size regression can be fitted
- The method must be benchmarked on CPU-only systems — the article notes all scMDC experiments require a Nvidia Tesla P100 GPU, so runtime comparisons would not be reproducible

## Inputs

- Simulated CITE-seq datasets (mRNA + ADT) at log-spaced cell counts from 1,000 to 100,000 cells, generated with SymSim with signal and dropout
- scMDC running script from the 'script' folder
- Nvidia Tesla P100 (16G) GPU hardware

## Outputs

- Tabulated cell count versus wall-clock runtime
- Fitted linear model of runtime versus cell count confirming linear scalability
- runtime_scalability.csv

## How to apply

Generate a log-spaced series of simulated CITE-seq datasets (mRNA + ADT) spanning 1,000 to 100,000 cells using SymSim, including signal and dropout as in the paper's simulation experiments, and export each in H5 format with the scMDC-required X1 (RNA) / X2 (ADT) structure. Run scMDC on each dataset with the running script from the 'script' folder on an Nvidia Tesla P100 (16G) GPU, recording wall-clock training time per cell count; reference points are ~7 minutes at 10,000 cells and ~1 hour at 100,000 cells. Tabulate cell count versus runtime, then fit a linear model of runtime as a function of cell count: an approximately linear fit (constant per-cell time) confirms the scalability claim. Save the results as runtime_scalability.csv.

## Related tools

- **SymSim** (R package used to generate simulated single-cell multi-omics datasets at each cell count for the scalability series)
- **scMDC** (Method under test whose running time is recorded at each simulated dataset size)
- **PyTorch** (Deep learning framework underlying scMDC's GPU training timed in the benchmark)
- **h5py** (Used to export each simulated dataset in H5 format with the X1 (RNA) / X2 (ADT) structure required by scMDC)
- **Scanpy** (Preprocessing/normalization of mRNA and ADT data before running scMDC on each simulated dataset)

## Examples

```
python scripts/run_scMDC.py --input sim_100000_cells.h5 --gpu tesla_p100; # record wall-clock time, then fit lm(runtime ~ cells) and save runtime_scalability.csv
```

## Evaluation signals

- Fitted runtime-versus-cell-count model shows approximately linear growth (no super-linear term)
- Measured runtime is consistent with ~7 minutes at 10,000 cells and ~1 hour at 100,000 cells on a Tesla P100 (16G) GPU
- Per-cell runtime is roughly constant across the size series (time per 10,000 cells does not inflate with dataset size)
- runtime_scalability.csv contains one runtime row per simulated dataset size across the full 1,000–100,000 cell range
- Per-cell runtime is roughly constant across the size series (time per 10,000 cells does not inflate with dataset size)

## Limitations

- Runtimes are hardware-specific: all scMDC experiments in the study were run on an Nvidia Tesla P100 (16G) GPU, so timings do not transfer to CPU-only systems
- scMDC requires the number of clusters (K) to be specified by the user after pretraining; it is not estimated automatically
- Scalability is demonstrated on simulated CITE-seq data; runtime on real data with different modality dimensions may differ

## Evidence

- [abstract] linear scalability reference runtimes: "The linear scalability of running time makes scMDC a promising method for analyzing large multimodal datasets"
- [discussion] GPU requirement: "All experiments of scMDC in this study are conducted on Nvidia Tesla P100 (16 G) GPU."
- [discussion] K must be user-specified: "After pretraining, the users need to specify the number of clusters (K)."
- [discussion] SymSim simulation design: "We perform three experiments to test the clustering performance of scMDC and generate 10 datasets in each experiment."
- [methods] H5 input preparation: "1) Prepare the input data in h5 format. (See readme in 'dataset' folder)"
