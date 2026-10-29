---
name: batch-effect-simulation
description: Use when benchmarking CITE-seq clustering methods under controlled regimes
  (clustering signal, dropout, batch effect) where simulated paired mRNA/ADT data
  with ground-truth cell-type labels are needed for reproducible comparison.
when_to_use_negative:
- Real CITE-seq data without ground-truth cell-type labels — AMI/NMI/ARI cannot be
  computed against simulated labels, so the supervised simulation benchmark design
  does not apply
- Single-modality (RNA-only) clustering benchmarks — the simulation pipeline is built
  around paired mRNA/ADT CITE-seq data with two jointly learned modalities
- Evaluating method properties other than clustering accuracy (e.g., joint embedding
  quality or cross-modal generation), which this label-based clustering benchmark
  does not measure
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3170
tools:
- name: SymSim
  role: Simulates the paired mRNA/ADT count datasets with clustering-signal, dropout,
    and batch-effect experiments; 10 datasets generated per experiment
  repo: https://github.com/YosefLab/SymSim
- name: scMDC
  role: 'Method under test: run on each simulated dataset with simulation-study parameters
    (pretrain_epochs=400, gamma=0.1, tol=0.001), with K supplied by the user'
  repo: https://github.com/xianglin226/scMDC
- name: Scanpy
  role: 'Preprocessing/normalization of the simulated datasets: filtering zero-count
    genes/ADTs, size-factor normalization, log-transform, and separate scaling of
    mRNA and ADT'
- name: scikit-learn
  role: Computes AMI, NMI, and ARI between each method's cluster assignments and the
    simulated ground-truth labels for every replicate
- name: TotalVI
  role: Next-best competing method in the simulation benchmark (as reported in the
    source card finding)
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-022-35031-9
    title: Clustering of single-cell multi-omics data with a multimodal deep learning
      method
  source_content_hash: sha256:1c88cf170491c3e258313c6796e11c2e0db9970d873ab7ee41df3ed6cd4b8128
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
  - TotalVI
  - Tscan
  - scDCC
  - scMDC
  - scikit-learn
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9629629629629629
n_claims: 27
---

# batch-effect-simulation

## Summary

Simulate paired mRNA/ADT CITE-seq datasets with SymSim under controlled clustering-signal, dropout-rate, and batch-effect regimes to benchmark scMDC against competing multimodal clustering methods using ground-truth labels and AMI/NMI/ARI metrics.

## When to use

Use when benchmarking CITE-seq clustering methods under controlled regimes (clustering signal, dropout, batch effect) where simulated paired mRNA/ADT data with ground-truth cell-type labels are needed for reproducible comparison.

## When NOT to use

- Real CITE-seq data without ground-truth cell-type labels — AMI/NMI/ARI cannot be computed against simulated labels, so the supervised simulation benchmark design does not apply
- Single-modality (RNA-only) clustering benchmarks — the simulation pipeline is built around paired mRNA/ADT CITE-seq data with two jointly learned modalities
- Evaluating method properties other than clustering accuracy (e.g., joint embedding quality or cross-modal generation), which this label-based clustering benchmark does not measure

## Inputs

- SymSim-simulated paired mRNA/ADT CITE-seq count matrices in h5 format (X1=RNA, X2=ADT, Y=ground-truth cell-type labels)
- Known number of clusters K per simulated dataset
- Simulated experimental settings: clustering-signal strength, dropout rates, and batch effects (10 datasets per experiment)

## Outputs

- Per-replicate AMI, NMI, and ARI scores for each method (scMDC vs. Tscan, Seurat V4 WNN, SC3, scDCC, BREM-SC, CiteFuse, Specter) via scikit-learn
- Mean clustering scores per method per experimental setting, formatted as the Fig. 4 simulation panels and Supplementary Table 3 layout, written to simulation_clustering_metrics.csv

## How to apply

Generate 10 simulated paired mRNA/ADT count datasets per experimental condition (clustering signal, dropout, batch effect) with SymSim, exporting each in h5 format with X1=RNA, X2=ADT, and Y=ground-truth cell-type labels. Preprocess each dataset per the scMDC pipeline: filter zero-count genes and ADTs, size-factor normalize, log-transform, and scale mRNA and ADT separately with Scanpy. Run scMDC on each dataset with the simulation-study parameters (pretrain_epochs=400, gamma=0.1, tol=0.001, sigma noise on RNA/ADT), supplying the known number of clusters K since scMDC does not estimate it automatically. Run competing methods (Tscan, Seurat V4 WNN, SC3, scDCC, BREM-SC, CiteFuse, Specter) on the same simulated datasets with their recommended settings. Compute AMI, NMI, and ARI against the simulated ground-truth labels with scikit-learn for every replicate, aggregate mean scores per method per setting across the 10 replicates, and format them into the Fig. 4 simulation panels and Supplementary Table 3 layout (simulation_clustering_metrics.csv). Validation: the reproduced table should show scMDC attaining the highest mean AMI/NMI/ARI over all competing methods across signal, dropout, and batch-effect experiments.

## Related tools

- **SymSim** (Simulates the paired mRNA/ADT count datasets with clustering-signal, dropout, and batch-effect experiments; 10 datasets generated per experiment) — https://github.com/YosefLab/SymSim
- **scMDC** (Method under test: run on each simulated dataset with simulation-study parameters (pretrain_epochs=400, gamma=0.1, tol=0.001), with K supplied by the user) — https://github.com/xianglin226/scMDC
- **Scanpy** (Preprocessing/normalization of the simulated datasets: filtering zero-count genes/ADTs, size-factor normalization, log-transform, and separate scaling of mRNA and ADT)
- **scikit-learn** (Computes AMI, NMI, and ARI between each method's cluster assignments and the simulated ground-truth labels for every replicate)
- **TotalVI** (Next-best competing method in the simulation benchmark (as reported in the source card finding))

## Examples

```
devtools::install_github("YosefLab/SymSim"); library(SymSim)  # simulate 10 paired mRNA/ADT datasets per setting, export h5 with X1=RNA, X2=ADT, Y=labels
```

## Evaluation signals

- Reproduced simulation table shows scMDC attaining the highest mean AMI/NMI/ARI over all competing methods across the signal, dropout, and batch-effect experiments
- Each experimental setting contains exactly 10 simulated datasets, matching the article's design
- Every h5 export carries the three required fields: X1 (RNA), X2 (ADT), and Y (ground-truth labels)
- Per-replicate metric computation succeeds for all competing methods, with TotalVI emerging as the next-best method behind scMDC
- Mean scores aggregated over the 10 replicates per method per setting reproduce the Fig. 4 simulation panels and Supplementary Table 3 layout

## Limitations

- All scMDC experiments require GPU hardware (Nvidia Tesla P100, 16 GB), which may limit reproducibility on CPU-only systems
- The number of clusters K must be specified by the user after pretraining; scMDC provides no automatic estimation
- SymSim is fundamentally an R package for simulating scRNA-seq data; its extension to paired mRNA/ADT CITE-seq simulation (with ADT channels and batch effects) follows the scMDC study's setup and may require adaptation for other modalities

## Evidence

- [discussion] Simulation of multi-omics data with SymSim including signal, dropout, and batch-effect experiments: "We perform three experiments to test the clustering performance of scMDC and generate 10 datasets in each experiment."
- [methods] h5 input preparation: "1) Prepare the input data in h5 format. (See readme in 'dataset' folder)"
- [discussion] Preprocessing pipeline: "The raw CITE-seq data is preprocessed and normalized by the Python package SCANPY48."
- [discussion] K must be supplied: "After pretraining, the users need to specify the number of clusters (K)."
- [readme] SymSim tool role: [evidence span withheld — anchor status: external_doc]
- [abstract] Simulation-based superiority finding: "Extensive simulation and real-data experiments reveal that scMDC outperforms existing single-cell single-modal and multimodal clustering methods"
