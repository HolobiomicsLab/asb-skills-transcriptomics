---
name: supervised-cell-label-benchmarking
description: Use when training scRNA-seq data with known cell labels are available
  and you must quantify whether a supervised annotation method (e.g. NeuCA) outperforms
  existing supervised or unsupervised alternatives under varying train/test splits
  or conditions.
when_to_use_negative:
- Novel/unknown cell types may exist in the testing data — NeuCA assumes all cell
  types in the new sample already exist in the reference/training data
- No training data with known labels exist for the tissue of interest — unsupervised
  methods are then the only viable solution
- The goal is novelty detection of rare cells rather than exhaustive label assignment;
  identify rare/unknown cells first (e.g. inferCNV or copyKAT for malignant cells)
  before applying NeuCA
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: NeuCA
  role: Primary supervised neural-network classifier being benchmarked; also the R/Bioconductor
    package used to train and predict labels (keras API, ReLU, RMSprop, mini-batch
    training)
  repo: https://bioconductor.org/packages/NeuCA
- name: Seurat
  role: Unsupervised clustering baseline evaluated via ARI on the same simulation
    splits
- name: SC3
  role: Unsupervised clustering baseline evaluated via ARI on the same simulation
    splits
- name: scmap
  role: Supervised annotation baseline (scmap-cluster and scmap-cell) in the cross-condition
    ASD benchmark
- name: CHETAH
  role: Supervised hierarchical annotation baseline; known to produce unassigned/intermediate
    labels
- name: SingleCellExperiment
  role: Bioconductor data container required for NeuCA training and test inputs
- name: keras
  role: Neural-network API used by NeuCA for fast high-level network implementation
provenance:
  source_task_ids:
  - task_004
  - task_008
  source_papers:
  - doi: 10.1038/s41598-021-04473-4
    title: A neural network-based method for exhaustive cell label assignment using
      single cell RNA-seq data
  source_content_hash: sha256:92b9cce6786c0d5ce984565bec441fed8f0020ae202f3d7c8a0249262322a882
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CHETAH
  - NeuCA
  - R
  - SC3
  - Seurat
  - SingleCellExperiment
  - keras
  - scmap
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# supervised-cell-label-benchmarking

## Summary

Benchmark supervised cell label assignment (NeuCA and baselines such as scmap-cluster, scmap-cell, CHETAH) against unsupervised clustering (Seurat, SC3) on scRNA-seq data with known labels, using accurately assigned rate and adjusted Rand index (ARI) as metrics across training-size, testing-size, and cross-condition settings.

## When to use

Use when training scRNA-seq data with known cell labels are available and you must quantify whether a supervised annotation method (e.g. NeuCA) outperforms existing supervised or unsupervised alternatives under varying train/test splits or conditions.

## When NOT to use

- Novel/unknown cell types may exist in the testing data — NeuCA assumes all cell types in the new sample already exist in the reference/training data
- No training data with known labels exist for the tissue of interest — unsupervised methods are then the only viable solution
- The goal is novelty detection of rare cells rather than exhaustive label assignment; identify rare/unknown cells first (e.g. inferCNV or copyKAT for malignant cells) before applying NeuCA

## Inputs

- Normalized scRNA-seq expression matrices (train and test) with published true cell labels for the training set, as SingleCellExperiment objects with normcounts and colData annotation
- Real 10X PBMC data downloaded from the 10X Genomics website for simulation settings
- Zheng_8ct_PBMC from the DuoClustering2018 R/Bioconductor package
- ASD snRNA-seq expression matrices with labels for 15 ASD patients and control samples
- GEO datasets for inter-study tests: GSE130430 (NK cells), GSE103544 (monocytes), GSE81608 (Xin/Baron/Muraro)

## Outputs

- Predicted cell label vector (one label per test cell) from each method
- Confusion matrix of predicted vs. true labels where ground truth is available
- Benchmark table of accurately assigned rate (supervised methods) and ARI (Seurat, SC3) per method, setting, and split, e.g. pbmc_simulation_benchmark_results.csv or asd_crosscondition_benchmark.csv

## How to apply

Preprocess each train/test pair by normalizing the expression matrices and subsetting both to their overlapped genes, then format them as SingleCellExperiment objects with normcounts and colData labels. Train NeuCA — which computes averaged cell-type expression profiles, builds the Pearson correlation matrix across cell types, and picks the feed-forward network vs. the marker-guided hierarchical network at the tau=0.95 correlation threshold — and run baselines (scmap-cluster, scmap-cell, CHETAH, Seurat, SC3) on identical splits. For sensitivity analyses on the 10X PBMC data, vary training proportion p in {10%, 20%, 50%, 80%} with n=4000 test cells, and testing size n in {800, 1600, 4000}; build three simulation settings (all cell types, five T cell subtypes only, and major types with the five T cell subtypes merged into one 'T cells' type). For cross-condition benchmarking (ASD snRNA-seq, 15 patients vs. controls), train on ASD to predict controls and vice versa. Evaluate with accurately assigned rate (NeuCA, scmap, CHETAH) and ARI (Seurat, SC3) against published true labels, writing metrics to a results CSV; confirm expected trends such as Seurat's ARI rising from ~0.45 to 0.5 and SC3's from ~0.60 to 0.65 as testing size grows from 800 to 4000 while NeuCA stays the highest performer.

## Related tools

- **NeuCA** (Primary supervised neural-network classifier being benchmarked; also the R/Bioconductor package used to train and predict labels (keras API, ReLU, RMSprop, mini-batch training)) — https://bioconductor.org/packages/NeuCA
- **Seurat** (Unsupervised clustering baseline evaluated via ARI on the same simulation splits)
- **SC3** (Unsupervised clustering baseline evaluated via ARI on the same simulation splits)
- **scmap** (Supervised annotation baseline (scmap-cluster and scmap-cell) in the cross-condition ASD benchmark)
- **CHETAH** (Supervised hierarchical annotation baseline; known to produce unassigned/intermediate labels)
- **SingleCellExperiment** (Bioconductor data container required for NeuCA training and test inputs)
- **keras** (Neural-network API used by NeuCA for fast high-level network implementation)

## Examples

```
predicted.label = NeuCA(train = Baron_sce, test = Seg_sce, model.size = "big", verbose = FALSE)
```

## Evaluation signals

- As training proportion increases from 10% to 80%, NeuCA's accurately assigned rate rises from about 0.86 to 0.90
- As testing size grows from 800 to 4000 cells, Seurat's ARI rises from ~0.45 to 0.5 and SC3's from ~0.60 to 0.65, while NeuCA remains the highest performer in ARI and accuracy
- In the ASD cross-condition benchmark, NeuCA achieves the highest accuracy and ARI in both train-test directions, and scmap-cell is the second-best supervised method with accurately assigned rate around 0.7
- Confusion matrix diagonals should dominate the off-diagonals (e.g. 328/331 alpha cells correctly assigned in the README example); predicted.label vector length equals number of test cells
- All metrics for every method, direction, and split are written to the benchmark CSV (e.g. pbmc_simulation_benchmark_results.csv)

## Limitations

- NeuCA is designed for settings where all cell types in the new sample already exist in the reference/training data; it does not detect unknown or novel types
- Cross-condition prediction demonstrated on ASD data may not hold in other scenarios; expert consultation is recommended for novel-condition tissues
- When unknown cell types are not expected, existing methods (e.g. CHETAH) still label a considerable number of cells as unassigned, which affects fair comparison
- No version numbers are given for keras, TensorFlow, or R in the software description, complicating exact reproduction

## Evidence

- [methods] Correlation-based strategy selection (threshold tau = 0.95): [evidence span withheld — anchor status: not_found]
- [abstract] NeuCA outperforms existing methods: "NeuCA can achieve more accurate cell annotation results compared with existing methods"
- [readme] README training and prediction call: "predicted.label = NeuCA(train = Baron_sce, test = Seg_sce, model.size = "big", verbose = FALSE)"
- [intro] Supervised outperforms unsupervised when training data available: "when training data are available, the improved accuracy, speed, and reproducibility of supervised methods easily outperform unsupervised ones"
- [discussion] Cross-condition scope caveat: "Although our application to ASD data shows that NeuCA can be applied to cross-condition predictions, this may not hold in other scenarios."
