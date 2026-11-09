---
name: pbmc-simulation-dataset-design
description: Use when benchmarking cell label assignment methods on PBMC scRNA-seq
  under controlled training-size and testing-size conditions, needing simulation splits
  and metrics like ARI.
when_to_use_negative:
- The new sample may contain cell types absent from the training/reference data —
  NeuCA is designed for settings where all cell types in the new sample already exist
  in the reference
- The goal is annotating a real dataset rather than benchmarking annotation methods
  under controlled training/testing-size conditions
- Unknown or novel cell types are expected in the data; identifying rare/unknown cells
  should be done as a first step prior to applying NeuCA
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: NeuCA
  role: Supervised neural-network classifier trained on each simulation split to predict
    test cell labels; strategy (feed-forward vs hierarchical) auto-selected at tau=0.95
  repo: https://bioconductor.org/packages/NeuCA
- name: Seurat
  role: Unsupervised clustering baseline run on identical simulation splits; performance
    scored by ARI
- name: SC3
  role: Unsupervised clustering baseline run on identical simulation splits; performance
    scored by ARI
- name: R
  role: Implementation environment for the simulation pipeline, baselines, and metric
    computation (NeuCA is an R/Bioconductor package)
  repo: https://bioconductor.org/packages/NeuCA
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41598-021-04473-4
    title: A neural network-based method for exhaustive cell label assignment using
      single cell RNA-seq data
  source_content_hash: sha256:cfce6c32ad0397c1cb8cb3d2b3bd279b4ded5df1150c40b29340331d117117cc
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - NeuCA
  - R
  - SC3
  - Seurat
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# pbmc-simulation-dataset-design

## Summary

Design PBMC-based simulation benchmarks from real 10X Genomics scRNA-seq data to assess how training proportion (10–80%) and testing size (800/1600/4000 cells) affect supervised annotation (NeuCA) versus baselines (Seurat, SC3), measured by accurately assigned rate and ARI.

## When to use

Use when benchmarking cell label assignment methods on PBMC scRNA-seq under controlled training-size and testing-size conditions, needing simulation splits and metrics like ARI.

## When NOT to use

- The new sample may contain cell types absent from the training/reference data — NeuCA is designed for settings where all cell types in the new sample already exist in the reference
- The goal is annotating a real dataset rather than benchmarking annotation methods under controlled training/testing-size conditions
- Unknown or novel cell types are expected in the data; identifying rare/unknown cells should be done as a first step prior to applying NeuCA

## Inputs

- Real PBMC scRNA-seq count matrix downloaded from the 10X Genomics website (https://support.10xgenomics.com/)
- Known cell type labels for training cells
- Training and testing splits subset to overlapped genes, as SingleCellExperiment objects for NeuCA

## Outputs

- pbmc_simulation_benchmark_results.csv containing accurately assigned rates (NeuCA) and ARI values (Seurat, SC3) per simulation setting
- Trained NeuCA classifiers per training split (feed-forward or hierarchical network chosen by tau=0.95 rule)

## How to apply

Download real PBMC data from the 10X Genomics website and preprocess it by normalizing the expression matrix and subsetting training and testing data to overlapped genes. Build simulation setting 1 by randomly drawing p% of cells as the training set with p in {10%, 20%, 50%, 80%} and n=4000 test cells from the remainder; construct settings 2 and 3 using five T cell subtypes only, and major cell types with the five T cell types merged into one 'T cells' type. Train the NeuCA classifier on each split, relying on its correlation-based strategy selection with threshold tau=0.95 to pick a feed-forward versus hierarchical network (ReLU activation, RMSprop optimizer, mini-batch training). Run Seurat and SC3 baselines on the identical splits, and vary the testing size n in {800, 1600, 4000} to probe testing-size sensitivity. Compute the accurately assigned rate for NeuCA and ARI for Seurat/SC3 per setting and write all metrics to pbmc_simulation_benchmark_results.csv.

## Related tools

- **NeuCA** (Supervised neural-network classifier trained on each simulation split to predict test cell labels; strategy (feed-forward vs hierarchical) auto-selected at tau=0.95) — https://bioconductor.org/packages/NeuCA
- **Seurat** (Unsupervised clustering baseline run on identical simulation splits; performance scored by ARI)
- **SC3** (Unsupervised clustering baseline run on identical simulation splits; performance scored by ARI)
- **R** (Implementation environment for the simulation pipeline, baselines, and metric computation (NeuCA is an R/Bioconductor package)) — https://bioconductor.org/packages/NeuCA

## Examples

```
predicted.label = NeuCA(train = Baron_sce, test = Seg_sce, model.size = "big", verbose = FALSE)
```

## Evaluation signals

- NeuCA's accurately assigned rate rises from about 0.86 to 0.90 as the training proportion increases from 10% to 80%
- As testing size grows from 800 to 4000 cells, Seurat's ARI rises from about 0.45 to 0.5 and SC3's from about 0.60 to 0.65
- NeuCA maintains higher accuracy and ARI than Seurat and SC3 across all training proportions and testing sizes
- Training/test matrices are subset to overlapped genes and normalized before model fitting
- All per-setting metrics are present in pbmc_simulation_benchmark_results.csv for every combination of p in {10,20,50,80}% and n in {800,1600,4000}

## Limitations

- Cross-condition prediction may not hold in scenarios other than the ASD data application
- No version numbers given for keras, TensorFlow, or R in the software description
- Deciding whether unknown/novel cell types exist in the data is not addressed by the method; expert consultation is recommended

## Evidence

- [methods] Real PBMC data from the 10X Genomics website: "Real PBMC data from the 10X Genomics website (https://support.10xgenomics.com/)31 were downloaded and processed to generate our simulation datasets"
- [methods] Correlation-based strategy selection threshold: [evidence span withheld — anchor status: not_found]
- [abstract] NeuCA outperforms baselines in benchmark: "NeuCA can achieve more accurate cell annotation results compared with existing methods"
- [readme] NeuCA input format requirement: [evidence span withheld — anchor status: external_doc]
- [discussion] NeuCA scope restriction: "NeuCA is designed to fit the situations where cell types are well-studied, i.e., the situations where all the cell types in the new sample already exist in the reference/training data."
