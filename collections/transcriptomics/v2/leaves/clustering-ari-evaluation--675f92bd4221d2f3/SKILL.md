---
name: clustering-ari-evaluation
description: Use when comparing clustering-based cell type identification (Seurat,
  SC3) against a supervised annotator on scRNA-seq data with known true labels, and
  you need ARI as the comparable metric across training/test size conditions.
when_to_use_negative:
- No true cell labels are available for the testing data — ARI and accurately assigned
  rate cannot be computed against ground truth
- The testing sample may contain unknown/novel cell types absent from the training
  reference — NeuCA is restricted to settings where all test cell types already exist
  in the training data
- Input is already a clustering result/label vector with no underlying expression
  matrix — Seurat and SC3 cannot be re-run on the same splits
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: NeuCA
  role: Supervised neural-network classifier trained on each training split (tau=0.95
    correlation threshold selects feed-forward vs hierarchical network); evaluated
    by accurately assigned rate
  repo: https://bioconductor.org/packages/NeuCA
- name: Seurat
  role: Unsupervised clustering baseline run on the same simulation splits; evaluated
    by ARI
- name: SC3
  role: Unsupervised clustering baseline run on the same simulation splits; evaluated
    by ARI
- name: R
  role: Implementation environment for the simulation, training, and metric computation
    workflow
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41598-021-04473-4
    title: A neural network-based method for exhaustive cell label assignment using
      single cell RNA-seq data
  source_content_hash: sha256:5e1f8bd23e7cfd301f8151351c9647176502c44c464ac5392ae26927c2acedbf
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Harmony
  - LIGER
  - NeuCA
  - R
  - SC3
  - Scanorama
  - Seurat
  - scETM
  - scVAE-GM
  - scVI
  - scVI-LD
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# clustering-ari-evaluation

## Summary

Benchmark sensitivity-analysis protocol that scores unsupervised clustering baselines (Seurat, SC3) by Adjusted Rand Index against the supervised NeuCA annotator's accurately-assigned rate, on 10X PBMC simulation splits varying training proportion (10/20/50/80%) and test size (800/1600/4000 cells).

## When to use

Use when comparing clustering-based cell type identification (Seurat, SC3) against a supervised annotator on scRNA-seq data with known true labels, and you need ARI as the comparable metric across training/test size conditions.

## When NOT to use

- No true cell labels are available for the testing data — ARI and accurately assigned rate cannot be computed against ground truth
- The testing sample may contain unknown/novel cell types absent from the training reference — NeuCA is restricted to settings where all test cell types already exist in the training data
- Input is already a clustering result/label vector with no underlying expression matrix — Seurat and SC3 cannot be re-run on the same splits

## Inputs

- Normalized scRNA-seq expression matrix from 10X PBMC data (subset to overlapped genes between training and testing sets)
- Known true cell labels for test cells (needed to compute ARI and accurately assigned rate)
- SingleCellExperiment objects for training and testing sets (NeuCA input format)

## Outputs

- ARI values for Seurat and SC3 per simulation setting and test size
- Accurately assigned rate for NeuCA per training proportion and test size
- pbmc_simulation_benchmark_results.csv containing all computed metrics

## How to apply

Download real PBMC data from the 10X Genomics website, normalize the expression matrix, and subset training and testing data to overlapped genes to build simulation datasets. Randomly draw p% of cells as the training set with p in {10%, 20%, 50%, 80%} and draw n=4000 test cells from the remainder; also vary the testing set size n in {800, 1600, 4000}. Train the NeuCA classifier on each split — it computes the averaged expression profile per cell type and a Pearson correlation matrix across types, choosing a feed-forward network vs. a cell lineage-based hierarchical network at threshold tau=0.95 (ReLU activation, RMSprop optimizer, mini-batch training) — and record its accurately assigned rate. Run Seurat and SC3 baselines on the same splits and compute their ARI, including the five-T-cell-subtype setting and the setting with the five T cell types merged into one 'T cells' type. Write all metrics to pbmc_simulation_benchmark_results.csv and check the reported trends: Seurat ARI rising from ~0.45 to ~0.5 and SC3 from ~0.60 to ~0.65 as test size grows from 800 to 4000, while NeuCA's accurately assigned rate rises from ~0.86 to ~0.90 with training proportion and stays highest.

## Related tools

- **NeuCA** (Supervised neural-network classifier trained on each training split (tau=0.95 correlation threshold selects feed-forward vs hierarchical network); evaluated by accurately assigned rate) — https://bioconductor.org/packages/NeuCA
- **Seurat** (Unsupervised clustering baseline run on the same simulation splits; evaluated by ARI)
- **SC3** (Unsupervised clustering baseline run on the same simulation splits; evaluated by ARI)
- **R** (Implementation environment for the simulation, training, and metric computation workflow)

## Examples

```
predicted.label = NeuCA(train = Baron_sce, test = Seg_sce, model.size = "big", verbose = FALSE)
```

## Evaluation signals

- As testing size grows from 800 to 4000 cells, Seurat's ARI rises from ~0.45 to ~0.5 and SC3's from ~0.60 to ~0.65
- NeuCA's accurately assigned rate rises from ~0.86 to ~0.90 as the training proportion increases from 10% to 80%
- NeuCA shows higher accuracy and ARI than all compared existing methods in every tested setting (including 5 T cell subtypes and merged T cells settings)
- All metrics for every p/n combination are present in pbmc_simulation_benchmark_results.csv

## Limitations

- NeuCA is designed for situations where all the cell types in the new sample already exist in the reference/training data; novel or unknown types fall outside its scope
- Existing label assignment methods (including those in this comparison) may label a considerable number of cells as unassigned even when unknown cell types are not expected
- No version numbers are given for keras, TensorFlow, or R, complicating exact reproduction of the neural-network training environment

## Evidence

- [methods] methods: "Real PBMC data from the 10X Genomics website (https://support.10xgenomics.com/)31 were downloaded and processed to generate our simulation datasets"
- [readme] readme: "NeuCA can detect whether highly-correlated cell types exist in the training dataset, and automatically determine if a general neural-network model will be adopted"
- [abstract] abstract: "the applications on eight real datasets show that NeuCA has stable performance for intra- and inter-study annotation, as well as cross-condition annotation"
