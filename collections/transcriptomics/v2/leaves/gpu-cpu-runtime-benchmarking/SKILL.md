---
name: gpu-cpu-runtime-benchmarking
description: 'Use when evaluating whether an imputation pipeline scales to large scRNA-seq
  datasets: time identical LATE runs on GPU and CPU builds of TensorFlow and compare
  against other imputation methods'' runtimes.'
when_to_use_negative:
- The goal is imputation accuracy (e.g., mean squared error against a masked ground
  truth) rather than runtime scalability — use the LATE imputation/analysis workflow
  instead
- The dataset is a small PBMC subset (e.g., example.msk90.hd5) where GPU-vs-CPU differences
  are unlikely to be meaningful; timing is intended for large (1M+ cell) data
- The environment only supports TensorFlow 2.x, since LATE is not compatible with
  TF2 and timing results would be unobtainable
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: LATE
  role: Autoencoder-based imputation method whose GPU and CPU wall-clock runtimes
    are benchmarked
  repo: https://github.com/audreyqyfu/LATE
- name: TensorFlow (1.0-1.4)
  role: Backend for LATE; installed as tensorflow-gpu in the GPU environment and CPU-only
    TensorFlow in the CPU environment to produce the timing comparison
- name: DCA
  role: Comparison imputation method timed on the same data/hardware for runtime reference
- name: scVI
  role: Comparison imputation method timed on the same data/hardware for runtime reference
- name: Python 3.5+
  role: Language in which the LATE implementation and benchmarking scripts are written
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1007/s40484-019-0192-7
    title: Imputation of single‐cell gene expression with an autoencoder neural network
  source_content_hash: sha256:bc0ecef30c138c912982912998b92a8d70ee83169d749ee5867f6a50be51014a
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/audreyqyfu/LATE
  tools:
  - DCA
  - LATE
  - Python 3.5+
  - Python+
  - TensorFlow (1.0-1.4)
  - numpy
  - pandas
  - scVI
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# gpu-cpu-runtime-benchmarking

## Summary

Benchmark wall-clock runtime of LATE imputation on GPU versus CPU environments using the 10x Genomics mouse brain scRNA-seq data, to demonstrate scalability (over 1 million cells in a few hours on a GPU) and quantify the GPU speedup relative to CPU.

## When to use

Use when evaluating whether an imputation pipeline scales to large scRNA-seq datasets: time identical LATE runs on GPU and CPU builds of TensorFlow and compare against other imputation methods' runtimes.

## When NOT to use

- The goal is imputation accuracy (e.g., mean squared error against a masked ground truth) rather than runtime scalability — use the LATE imputation/analysis workflow instead
- The dataset is a small PBMC subset (e.g., example.msk90.hd5) where GPU-vs-CPU differences are unlikely to be meaningful; timing is intended for large (1M+ cell) data
- The environment only supports TensorFlow 2.x, since LATE is not compatible with TF2 and timing results would be unobtainable

## Inputs

- 10x Genomics mouse brain scRNA-seq h5 file (example_mouse_brain_10x.h5; genes in rows, cells in columns; orientation must be explicitly specified)
- GPU environment: virtualenv with TensorFlow 1.x (1.0-1.4 range; e.g., 1.15) and tensorflow-gpu
- CPU-only environment: same virtualenv with CPU-only TensorFlow
- Optionally, the same input matrix for comparison imputation methods (DCA, scVI, SAVER, scImpute)

## Outputs

- Wall-clock runtime (hours) of the LATE GPU run
- Wall-clock runtime (hours) of the LATE CPU run
- Runtime table for LATE and comparison methods (e.g., runtime_benchmark.tsv), including recorded failures such as memory errors

## How to apply

Create a virtual environment with TensorFlow 1.x (e.g., tensorflow==1.15) plus tensorflow-gpu, since LATE is not compatible with TensorFlow 2, and install numpy, pandas, tables, and related modules. Run LATE on the mouse brain h5 file via example_late_10x.py, explicitly specifying the input orientation (genes in rows, cells in columns) and noting the input is log10(count+1)-transformed before imputation; record wall-clock training time. Repeat the identical run in a second environment with CPU-only TensorFlow installed and record its time. Optionally time comparison methods (DCA, scVI, SAVER, scImpute) on the same data and hardware, recording failures such as memory errors. Aggregate all runtimes into a single table (e.g., runtime_benchmark.tsv) and accept the benchmark if the GPU run completes in a few hours' order of magnitude and both GPU/CPU timings are recorded.

## Related tools

- **LATE** (Autoencoder-based imputation method whose GPU and CPU wall-clock runtimes are benchmarked) — https://github.com/audreyqyfu/LATE
- **TensorFlow (1.0-1.4)** (Backend for LATE; installed as tensorflow-gpu in the GPU environment and CPU-only TensorFlow in the CPU environment to produce the timing comparison)
- **DCA** (Comparison imputation method timed on the same data/hardware for runtime reference)
- **scVI** (Comparison imputation method timed on the same data/hardware for runtime reference)
- **Python 3.5+** (Language in which the LATE implementation and benchmarking scripts are written)

## Examples

```
python example_late_10x.py -mode='late' -infile='../data/example_mouse_brain_10x.h5'
```

## Evaluation signals

- The LATE GPU run completes successfully in only a few hours' order of magnitude, consistent with the reported scalability that LATE can process over 1 million cells in just a few hours on a GPU
- Wall-clock time is recorded for BOTH the GPU and CPU runs so the GPU/CPU speedup can be computed (reported as roughly seven-fold, e.g., under 11 h GPU vs 53.2 h CPU on 1.3M-cell data)
- The identical command and input file are used in both environments so the GPU-vs-CPU comparison is apples-to-apples
- Runtime results for comparison methods (or their failure modes, e.g., memory errors) are captured in the aggregate table
- Output folder step2/ with imputation.step2.hd5 is produced, confirming the timed run actually finished imputation

## Limitations

- LATE is not compatible with TensorFlow 2; a virtual environment with TensorFlow 1.x is required, so timing results depend on legacy software
- Hyperparameters were manually tuned and systematic hyperparameter tuning (as in DCA) was not explored, which may affect runtime comparability
- The mouse brain h5 used in the example is a small subset of 10,000 genes and 10,000 cells; full 1M+ cell timing requires the complete 10x dataset
- Comparison methods may fail outright (e.g., memory errors) on large data, so their runtime rows may be qualitative rather than numeric

## Evidence
