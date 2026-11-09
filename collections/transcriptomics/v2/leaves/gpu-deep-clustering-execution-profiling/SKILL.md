---
name: gpu-deep-clustering-execution-profiling
description: Use when you must characterize how a deep single-cell clustering method's
  runtime scales with cell count, or benchmark empirical runtimes (e.g., scMDC at
  ~7 min/10K cells, ~1 hr/100K cells) on GPU hardware.
when_to_use_negative:
- Benchmarking clustering accuracy or cell-type resolution rather than running time
  — use the paper's clustering-performance simulation experiments instead
- The available hardware is CPU-only — all scMDC experiments in this study require
  an Nvidia Tesla P100 (16 G) GPU, so runtime measurements would not be comparable
- The input is a real experimental dataset (e.g., GSE100866 CBMN or GSE128639 BMNC)
  rather than a SymSim simulation at controlled cell counts — cell number is then
  confounded and the scalability curve cannot be cleanly fitted
edam_operation: http://edamontology.org/operation_3840
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_3170
tools:
- name: SymSim
  role: R package used to simulate CITE-seq (mRNA + ADT) datasets at controlled cell
    counts from 1,000 to 100,000 cells with signal and dropout
  repo: https://github.com/YosefLab/SymSim
- name: scMDC
  role: Multimodal deep learning clustering method whose wall-clock training time
    is profiled across cell counts using its 'script' folder running script
  repo: https://github.com/xianglin226/scMDC
- name: PyTorch
  role: Deep learning framework underlying scMDC training on the Nvidia Tesla P100
    (16G) GPU
- name: Scanpy
  role: Python package for preprocessing and normalization of the simulated mRNA and
    ADT data before scMDC execution
- name: h5py
  role: Used to export the simulated datasets in H5 format with the scMDC-required
    X1 (RNA) / X2 (ADT) structure
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-022-35031-9
    title: Clustering of single-cell multi-omics data with a multimodal deep learning
      method
  source_content_hash: sha256:7d6e52463d973151536466dc2b839fd720c1b7ba1a00d7f2f88df8bf7d99a6b2
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
  edam_operation: http://edamontology.org/operation_3840
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# gpu-deep-clustering-execution-profiling

## Summary

Profile the wall-clock running time of the scMDC deep clustering model across log-spaced simulated CITE-seq datasets (1,000 to 100,000 cells) on an Nvidia Tesla P100 (16G) GPU to demonstrate linear runtime scalability with cell number.

## When to use

Use when you must characterize how a deep single-cell clustering method's runtime scales with cell count, or benchmark empirical runtimes (e.g., scMDC at ~7 min/10K cells, ~1 hr/100K cells) on GPU hardware.

## When NOT to use

- Benchmarking clustering accuracy or cell-type resolution rather than running time — use the paper's clustering-performance simulation experiments instead
- The available hardware is CPU-only — all scMDC experiments in this study require an Nvidia Tesla P100 (16 G) GPU, so runtime measurements would not be comparable
- The input is a real experimental dataset (e.g., GSE100866 CBMN or GSE128639 BMNC) rather than a SymSim simulation at controlled cell counts — cell number is then confounded and the scalability curve cannot be cleanly fitted

## Inputs

- Simulated CITE-seq (mRNA + ADT) datasets generated with SymSim at log-spaced cell counts from 1,000 to 100,000 cells, including signal and dropout
- Each dataset exported in H5 format with scMDC-required structure (X1 = RNA matrix, X2 = ADT matrix)
- scMDC running script from the 'script' folder (scMDC v1.0.1)
- Nvidia Tesla P100 (16 Gb) GPU environment with PyTorch

## Outputs

- runtime_scalability.csv — table of cell count versus measured wall-clock training time
- Fitted linear model of runtime as a function of cell count verifying linear scalability

## How to apply

Generate simulated CITE-seq (mRNA + ADT) datasets with SymSim at a log-spaced series of cell counts spanning 1,000 to 100,000 cells, including signal and dropout as in the paper's simulation experiments. Export each dataset as H5 with the scMDC-required structure (X1 = RNA, X2 = ADT), then run scMDC using the running script from the 'script' folder on an Nvidia Tesla P100 (16G) GPU, recording wall-clock training time at each cell count. Tabulate cell count versus runtime, fit a linear model of runtime versus cell count, and save the result as runtime_scalability.csv. The linear fit is the rationale: the paper claims linear scalability makes scMDC suitable for large multimodal datasets, so approximately linear growth of the measured runtimes is the acceptance criterion.

## Related tools

- **SymSim** (R package used to simulate CITE-seq (mRNA + ADT) datasets at controlled cell counts from 1,000 to 100,000 cells with signal and dropout) — https://github.com/YosefLab/SymSim
- **scMDC** (Multimodal deep learning clustering method whose wall-clock training time is profiled across cell counts using its 'script' folder running script) — https://github.com/xianglin226/scMDC
- **PyTorch** (Deep learning framework underlying scMDC training on the Nvidia Tesla P100 (16G) GPU)
- **Scanpy** (Python package for preprocessing and normalization of the simulated mRNA and ADT data before scMDC execution)
- **h5py** (Used to export the simulated datasets in H5 format with the scMDC-required X1 (RNA) / X2 (ADT) structure)

## Examples

```
python run_scmdc.py --data sim_10000cells.h5 --gpu 0  # time each of the log-spaced SymSim datasets (1K–100K cells) and log wall-clock to runtime_scalability.csv
```

## Evaluation signals

- Measured runtime at 10,000 cells is approximately 7 minutes on an Nvidia Tesla P100 (16 G) GPU
- Measured runtime at 100,000 cells is approximately 1 hour on an Nvidia Tesla P100 (16 G) GPU
- A linear model fitted to cell count versus runtime shows approximately linear growth (good linear fit across the 1,000–100,000 cell range)
- runtime_scalability.csv exists and contains a runtime row for every log-spaced simulated cell count in the series

## Limitations

- All experiments of scMDC in this study are conducted on Nvidia Tesla P100 (16 G) GPU; runtimes measured on other hardware are not directly comparable
- The number of clusters (K) must be specified by the user; no automatic estimation is provided
- Reference points (~7 min at 10K cells, ~1 hr at 100K cells) apply to scMDC v1.0.1 on this GPU; absolute times will differ elsewhere even if linearity holds

## Evidence

- [abstract] Linear scalability of running time finding: "The linear scalability of running time makes scMDC a promising method for analyzing large multimodal datasets"
- [discussion] GPU hardware requirement: "All experiments of scMDC in this study are conducted on Nvidia Tesla P100 (16 G) GPU."
- [discussion] Simulation with SymSim: "We perform three experiments to test the clustering performance of scMDC and generate 10 datasets in each experiment."
- [methods] H5 input preparation: "1) Prepare the input data in h5 format. (See readme in 'dataset' folder)"
- [discussion] Pretraining protocol affecting runtime: "We pretrain the autoencoders for 400 epochs before entering the clustering stage."
