---
name: runtime-scalability-benchmarking
description: Use when you need to empirically characterize the computational scalability
  of a clustering or embedding method — e.g., verifying a claimed linear-time behavior
  or comparing runtimes of competing methods like scMDC and Specter as cell number
  grows.
when_to_use_negative:
- You need clustering accuracy or ARI comparisons — this skill measures runtime only,
  not performance quality
- Only CPU hardware is available — scMDC experiments in this study all require an
  Nvidia Tesla P100 (16G) GPU
- The input is a small real dataset with a fixed cell count and no scalability question
  is being asked
edam_operation: http://edamontology.org/operation_0334
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2229
tools:
- name: SymSim
  role: Generates simulated single-cell RNA-seq/multi-omics datasets at controlled
    cell counts with signal and dropout for the scalability series
  repo: https://github.com/YosefLab/SymSim
- name: scMDC
  role: The method being benchmarked; run on each simulated dataset to record training
    wall-clock time
  repo: https://github.com/xianglin226/scMDC
- name: h5py
  role: Exports simulated mRNA/ADT matrices to H5 format with the X1 (RNA) / X2 (ADT)
    structure scMDC requires
- name: PyTorch
  role: Deep learning backend in which scMDC training time is measured on GPU
- name: Scanpy
  role: Preprocessing and normalization of mRNA and ADT data before running the benchmarked
    method
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-022-35031-9
    title: Clustering of single-cell multi-omics data with a multimodal deep learning
      method
  source_content_hash: sha256:313e62edf9e5cd15b7ecba46adfed0a0cd60ebd86df67d0829cfa4bf7ec220d2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YosefLab/SymSim
  tools:
  - PyTorch
  - Scanpy
  - SymSim
  - h5py
  - scMDC
  edam_operation: http://edamontology.org/operation_0334
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# runtime-scalability-benchmarking

## Summary

Benchmarks how a method's wall-clock running time scales with cell number by running it on simulated single-cell multi-omics datasets spanning a log-spaced range of cell counts (1,000 to 100,000 cells) and fitting runtime versus cell count, as done for scMDC on an Nvidia Tesla P100 (16G) GPU.

## When to use

Use when you need to empirically characterize the computational scalability of a clustering or embedding method — e.g., verifying a claimed linear-time behavior or comparing runtimes of competing methods like scMDC and Specter as cell number grows.

## When NOT to use

- You need clustering accuracy or ARI comparisons — this skill measures runtime only, not performance quality
- Only CPU hardware is available — scMDC experiments in this study all require an Nvidia Tesla P100 (16G) GPU
- The input is a small real dataset with a fixed cell count and no scalability question is being asked

## Inputs

- Simulated CITE-seq datasets (mRNA + ADT) from SymSim at log-spaced cell counts from 1,000 to 100,000 cells
- H5-format input files with scMDC-required structure: X1 (RNA) and X2 (ADT)
- Running script from the 'script' folder of the method's repository

## Outputs

- Table of cell count versus wall-clock runtime (runtime_scalability.csv)
- Fitted linear model of runtime as a function of cell count
- Empirical runtime reference points (e.g., ~7 min at 10K cells, ~1 hour at 100K cells on Tesla P100 16G)

## How to apply

Generate simulated CITE-seq (mRNA + ADT) datasets with SymSim at log-spaced cell counts from 1,000 to 100,000 cells, including signal and dropout as in the paper's simulation experiments. Export each dataset as H5 with the structure required by the target method (for scMDC: X1 for RNA and X2 for ADT, per the 'dataset' folder readme). Run the method on each dataset using the script from the 'script' folder on an Nvidia Tesla P100 (16G) GPU, recording wall-clock training time for each cell count; the paper's reference points are ~7 minutes for 10,000 cells and ~1 hour for 100,000 cells. Tabulate cell count versus runtime and fit a linear model of runtime as a function of cell count to confirm the scalability relationship, saving the result as runtime_scalability.csv.

## Related tools

- **SymSim** (Generates simulated single-cell RNA-seq/multi-omics datasets at controlled cell counts with signal and dropout for the scalability series) — https://github.com/YosefLab/SymSim
- **scMDC** (The method being benchmarked; run on each simulated dataset to record training wall-clock time) — https://github.com/xianglin226/scMDC
- **h5py** (Exports simulated mRNA/ADT matrices to H5 format with the X1 (RNA) / X2 (ADT) structure scMDC requires)
- **PyTorch** (Deep learning backend in which scMDC training time is measured on GPU)
- **Scanpy** (Preprocessing and normalization of mRNA and ADT data before running the benchmarked method)

## Examples

```
python run_scmdc.py --dataset sim_100000.h5 --gpu 0  # record wall-clock; repeat across 1k-100k log-spaced sims and tabulate as runtime_scalability.csv
```

## Evaluation signals

- Runtime grows approximately linearly with cell number (linear model of runtime vs. cell count fits the tabulated data)
- Measured runtime at 10,000 cells is consistent with the reported ~7 minutes on a Tesla P100 (16G) GPU
- Measured runtime at 100,000 cells is consistent with the reported ~1 hour on a Tesla P100 (16G) GPU
- Each H5 input file loads with the expected structure (X1 for RNA, X2 for ADT) before the run
- runtime_scalability.csv contains a runtime entry for every simulated cell count in the 1,000–100,000 log-spaced series

## Limitations

- All experiments of scMDC in this study were conducted on an Nvidia Tesla P100 (16G) GPU, so absolute runtimes are hardware-dependent and may not reproduce on other GPUs or CPU-only systems
- scMDC requires the number of clusters (K) to be specified by the user; no automatic estimation is provided, so K must be fixed consistently across the scalability series
- The benchmark uses SymSim-simulated CITE-seq data; runtime on real datasets with different gene/ADT dimensions may deviate from the simulated scaling curve

## Evidence

- [abstract] Linear runtime finding with reference hardware: "The linear scalability of running time makes scMDC a promising method for analyzing large multimodal datasets"
- [discussion] Benchmark hardware: "All experiments of scMDC in this study are conducted on Nvidia Tesla P100 (16 G) GPU."
- [methods] Input format requirement: "1) Prepare the input data in h5 format. (See readme in 'dataset' folder)"
- [discussion] Competing method's runtime context: "Van et al.24 proposed a landmark-based spectral clustering (LSC) method, Spector, for clustering single-cell data with linear-time scalability."
- [discussion] Simulation design: "We perform three experiments to test the clustering performance of scMDC and generate 10 datasets in each experiment."
