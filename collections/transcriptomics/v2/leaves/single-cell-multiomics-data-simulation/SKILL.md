---
name: single-cell-multiomics-data-simulation
description: Use when benchmarking CITE-seq clustering methods (scMDC vs. TotalVI,
  Seurat V4 WNN, SC3, scDCC, BREM-SC, CiteFuse, Specter) requires controlled mRNA+ADT
  data with ground-truth labels, or for linear-runtime scalability tests.
when_to_use_negative:
- The data are real, already-annotated CITE-seq datasets (e.g. GSE128639 BMNC, 10X
  PBMC, GSE150599 SLN) — use the real-data benchmarking workflow rather than simulation.
- Only a single modality (scRNA-seq alone) needs simulation; the paired mRNA/ADT h5
  structure (X1/X2/Y) and multimodal preprocessing are not required.
- Only CPU hardware is available and validated results are mandatory — the study's
  simulated experiments were all run on an Nvidia Tesla P100 (16 G) GPU and CPU reproduction
  is unvalidated.
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0091
tools:
- name: SymSim
  role: R package used to simulate the CITE-seq count datasets for the three clustering-performance
    experiments (signal, dropout, batch effect) and the 1,000–100,000-cell scalability
    series
  repo: https://github.com/YosefLab/SymSim
- name: scMDC
  role: Consumer of the simulated h5 datasets (X1/X2, known K) whose cluster assignments
    and GPU wall-clock training times are measured
  repo: https://github.com/xianglin226/scMDC
- name: Scanpy
  role: 'Preprocessing and normalization of each simulated dataset: filter zero-count
    genes/ADTs, size-factor normalization, log-transform, and separate mRNA/ADT scaling'
- name: scikit-learn
  role: Computing AMI, NMI, and ARI between each method's cluster assignments and
    the simulated ground-truth labels
provenance:
  source_task_ids:
  - task_003
  - task_008
  source_papers:
  - doi: 10.1038/s41467-022-35031-9
    title: Clustering of single-cell multi-omics data with a multimodal deep learning
      method
  source_content_hash: sha256:68ef45af86319fa1a23dd0f0e02196d904941d52f1bec2e5f34102ed207aa247
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YosefLab/SymSim
  tools:
  - BREM-SC
  - CiteFuse
  - SC3
  - Scanpy
  - Seurat
  - Seurat V4
  - Specter
  - SymSim
  - Tscan
  - h5py
  - scDCC
  - scMDC
  - scikit-learn
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 16
---

# single-cell-multiomics-data-simulation

## Summary

Generates synthetic paired mRNA/ADT CITE-seq datasets with SymSim across clustering-signal, dropout-rate, and batch-effect regimes (plus 1,000–100,000-cell scalability series), exporting h5 files (X1=RNA, X2=ADT, Y=ground-truth labels) that serve as controlled benchmarks for evaluating scMDC-style multimodal clustering performance and runtime scaling.

## When to use

Use when benchmarking CITE-seq clustering methods (scMDC vs. TotalVI, Seurat V4 WNN, SC3, scDCC, BREM-SC, CiteFuse, Specter) requires controlled mRNA+ADT data with ground-truth labels, or for linear-runtime scalability tests.

## When NOT to use

- The data are real, already-annotated CITE-seq datasets (e.g. GSE128639 BMNC, 10X PBMC, GSE150599 SLN) — use the real-data benchmarking workflow rather than simulation.
- Only a single modality (scRNA-seq alone) needs simulation; the paired mRNA/ADT h5 structure (X1/X2/Y) and multimodal preprocessing are not required.
- Only CPU hardware is available and validated results are mandatory — the study's simulated experiments were all run on an Nvidia Tesla P100 (16 G) GPU and CPU reproduction is unvalidated.

## Inputs

- SymSim simulation parameter settings per experiment (clustering signal strength, dropout rate, batch effect; 10 replicates each)
- Log-spaced series of target cell counts from 1,000 to 100,000 cells for the runtime scalability experiment
- Known number of clusters K from the simulated ground truth (must be supplied by the user; no automatic estimation)
- R environment with devtools and the Bioconductor packages listed in SymSim's DESCRIPTION; Python environment with scMDC, PyTorch, Scanpy, h5py

## Outputs

- h5 simulation files with X1 (RNA counts), X2 (ADT counts), and Y (ground-truth cell-type labels)
- simulation_clustering_metrics.csv with mean AMI/NMI/ARI per method per experimental setting across 10 replicates
- runtime_scalability.csv tabulating cell count versus wall-clock runtime with a fitted linear model

## How to apply

Generate paired mRNA/ADT count matrices with SymSim (R, installed via devtools from GitHub) under three controlled regimes — clustering signal strength, dropout rate, and batch effect — producing 10 replicate datasets per experiment, and export each in h5 format with the scMDC-required structure (X1 = RNA, X2 = ADT, Y = ground-truth cell-type labels). Preprocess every simulated dataset per the scMDC pipeline with Scanpy: filter genes and ADTs with no count, size-factor normalize, log-transform, and scale mRNA and ADT separately. For scalability testing, regenerate datasets at log-spaced cell counts from 1,000 to 100,000 cells and record scMDC's wall-clock training time on an Nvidia Tesla P100 (16 G) GPU, then fit runtime vs. cell count to verify the linear relationship (~7 min at 10K, ~1 h at 100K cells). Finally, run each method with the known number of clusters K supplied and compute AMI, NMI, and ARI (scikit-learn) against the simulated labels, aggregating mean scores across the 10 replicates.

## Related tools

- **SymSim** (R package used to simulate the CITE-seq count datasets for the three clustering-performance experiments (signal, dropout, batch effect) and the 1,000–100,000-cell scalability series) — https://github.com/YosefLab/SymSim
- **scMDC** (Consumer of the simulated h5 datasets (X1/X2, known K) whose cluster assignments and GPU wall-clock training times are measured) — https://github.com/xianglin226/scMDC
- **Scanpy** (Preprocessing and normalization of each simulated dataset: filter zero-count genes/ADTs, size-factor normalization, log-transform, and separate mRNA/ADT scaling)
- **scikit-learn** (Computing AMI, NMI, and ARI between each method's cluster assignments and the simulated ground-truth labels)

## Examples

```
devtools::install_github("YosefLab/SymSim"); library(SymSim)
```

## Evaluation signals

- Every simulated dataset exports as h5 with keys X1 (RNA), X2 (ADT), Y (labels), and each of the three experiments yields 10 datasets.
- simulation_clustering_metrics.csv shows scMDC attaining the highest mean AMI/NMI/ARI over all competing methods (TotalVI next-best) across the signal, dropout, and batch-effect settings.
- runtime_scalability.csv shows runtime growing approximately linearly with cell count, with measured runtimes consistent with ~7 minutes at 10,000 cells and ~1 hour at 100,000 cells on a Tesla P100 (16 G).
- Preprocessed matrices contain no all-zero genes or ADTs, and mRNA and ADT are scaled separately per the Scanpy pipeline.
- Competing methods (Tscan, Seurat V4 WNN, SC3, scDCC, BREM-SC, CiteFuse, Specter) run on the identical simulated h5 inputs with their recommended settings.

## Limitations

- All experiments were conducted on an Nvidia Tesla P100 (16 G) GPU; CPU-only reproduction of the scalability timings may not be reproducible.
- The number of clusters K must be specified by the user after pretraining — no automatic estimation exists, so simulations must carry ground-truth K forward.
- SymSim is fundamentally an R package for scRNA-seq simulation; verify that the paired ADT channel and h5 export (X1/X2/Y) match the scMDC 'dataset' folder specification before large-scale generation.
- Only 10 replicates per experimental setting are generated, limiting statistical power for detecting small clustering-performance differences.

## Evidence

- [discussion] We perform three experiments to test the clustering performance of scMDC and generate 10 datasets in each experiment.: "We perform three experiments to test the clustering performance of scMDC and generate 10 datasets in each experiment."
- [discussion] Preprocessing rule for simulated and real data: "Specifically, the genes and ADTs with no count are filtered out."
- [discussion] GPU requirement for the simulated benchmark runs: "All experiments of scMDC in this study are conducted on Nvidia Tesla P100 (16 G) GPU."
- [abstract] Motivation for the linear runtime scalability experiment: "The linear scalability of running time makes scMDC a promising method for analyzing large multimodal datasets"
- [readme] Simulation tool identity and environment: [evidence span withheld — anchor status: external_doc]
- [methods] Required input file format for scMDC runs: "1) Prepare the input data in h5 format. (See readme in 'dataset' folder)"
- [discussion] K must be user-specified when clustering simulated data: "After pretraining, the users need to specify the number of clusters (K)."
