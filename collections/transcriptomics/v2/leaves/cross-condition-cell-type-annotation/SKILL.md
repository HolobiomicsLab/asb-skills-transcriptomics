---
name: cross-condition-cell-type-annotation
description: Use when a labeled scRNA-seq reference from one biological condition
  (e.g., ASD patients) must annotate cells in another condition (e.g., controls),
  and accuracy/ARI of competing supervised and unsupervised annotators must be benchmarked.
when_to_use_negative:
- The new sample may contain cell types absent from the reference data — NeuCA assumes
  all cell types in the new sample already exist in the training data and does not
  detect unknown types
- Deciding whether unknown or novel cell types exist in the data (e.g., tissues from
  novel conditions) — the article recommends consulting biological/clinical experts
  instead
- No labeled training data exist for the tissue of interest — unsupervised clustering
  is then the only viable option
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2269
tools:
- name: NeuCA
  role: Primary supervised neural-network annotator trained on one condition to predict
    labels in the other; selects feed-forward vs. hierarchical network via Pearson
    correlation of cell-type profiles at tau=0.95
  repo: https://bioconductor.org/packages/NeuCA/
- name: scmap
  role: Baseline supervised annotators (scmap-cluster, scmap-cell) run in both cross-condition
    directions; scmap-cell is expected second-best at ~0.7 accurately-assigned rate
- name: CHETAH
  role: Baseline supervised hierarchical annotator evaluated in both directions
- name: Seurat
  role: Baseline unsupervised clustering annotator evaluated in both directions
- name: SC3
  role: Baseline unsupervised clustering annotator evaluated in both directions
- name: R
  role: Implementation environment; NeuCA is an R/Bioconductor package depending on
    keras, limma, SingleCellExperiment, and e1071
  repo: https://bioconductor.org/packages/NeuCA/
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41598-021-04473-4
    title: A neural network-based method for exhaustive cell label assignment using
      single cell RNA-seq data
  source_content_hash: sha256:63074680c8ea0e71f8bf4d6d84b3d12313386e5687badc588a8f99ee172fb0b7
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CHETAH
  - NeuCA
  - R
  - SC3
  - Seurat
  - scmap
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# cross-condition-cell-type-annotation

## Summary

Benchmark protocol for supervised cross-condition cell-type annotation on ASD snRNA-seq data, in which NeuCA (feed-forward or hierarchical neural network selected at a Pearson correlation threshold tau=0.95) is trained on one condition and used to predict labels in the other, against scmap-cluster, scmap-cell, CHETAH, Seurat, and SC3 baselines.

## When to use

Use when a labeled scRNA-seq reference from one biological condition (e.g., ASD patients) must annotate cells in another condition (e.g., controls), and accuracy/ARI of competing supervised and unsupervised annotators must be benchmarked.

## When NOT to use

- The new sample may contain cell types absent from the reference data — NeuCA assumes all cell types in the new sample already exist in the training data and does not detect unknown types
- Deciding whether unknown or novel cell types exist in the data (e.g., tissues from novel conditions) — the article recommends consulting biological/clinical experts instead
- No labeled training data exist for the tissue of interest — unsupervised clustering is then the only viable option

## Inputs

- ASD snRNA-seq expression matrices with published cell labels (15 ASD patients)
- Control-condition snRNA-seq expression matrices with published cell labels
- SingleCellExperiment objects (normcounts assay, colData labels, rowData feature_symbol) for training and testing sets

## Outputs

- Predicted cell-type label vector for each testing cell (one per method, per train/test direction)
- Cross-condition benchmark table (asd_crosscondition_benchmark.csv) with accuracy and ARI per method and direction
- Confusion matrix / Sankey diagram of predicted vs. true labels for evaluation

## How to apply

Load the ASD snRNA-seq expression matrices with published cell labels for the 15 ASD patients and the control samples, then preprocess by normalizing each expression matrix and subsetting training and testing data to their overlapped genes. Represent both datasets as SingleCellExperiment objects (normcounts assay, colData cell labels, feature symbols in rowData with duplicated features removed) as required by NeuCA. Train NeuCA on one condition — it computes the averaged expression profile per cell type, a Pearson correlation matrix across cell types, and applies a tau=0.95 threshold to select between a feed-forward neural network and a marker-guided hierarchical network (model.size can be 'big', 'medium', or 'small') — then predict labels on the other condition; repeat with the training and testing roles swapped. Run scmap-cluster, scmap-cell, CHETAH, Seurat, and SC3 on the same preprocessed data in both directions as baselines. Compute annotation accuracy and adjusted Rand index (ARI) against published true labels for every method-direction pair and compile the cross-condition benchmark table (asd_crosscondition_benchmark.csv); the expected result is that NeuCA achieves the highest accuracy and ARI in both directions, with scmap-cell the second-best supervised method at around 0.7 accurately-assigned rate.

## Related tools

- **NeuCA** (Primary supervised neural-network annotator trained on one condition to predict labels in the other; selects feed-forward vs. hierarchical network via Pearson correlation of cell-type profiles at tau=0.95) — https://bioconductor.org/packages/NeuCA/
- **scmap** (Baseline supervised annotators (scmap-cluster, scmap-cell) run in both cross-condition directions; scmap-cell is expected second-best at ~0.7 accurately-assigned rate)
- **CHETAH** (Baseline supervised hierarchical annotator evaluated in both directions)
- **Seurat** (Baseline unsupervised clustering annotator evaluated in both directions)
- **SC3** (Baseline unsupervised clustering annotator evaluated in both directions)
- **R** (Implementation environment; NeuCA is an R/Bioconductor package depending on keras, limma, SingleCellExperiment, and e1071) — https://bioconductor.org/packages/NeuCA/

## Examples

```
predicted.label = NeuCA(train = asd_sce, test = control_sce, model.size = "big", verbose = FALSE)
```

## Evaluation signals

- NeuCA achieves the highest accuracy AND adjusted Rand index (ARI) among all six methods in both directions (train-on-ASD/test-on-control and train-on-control/test-on-ASD)
- scmap-cell is the second-best supervised method with accurately assigned rate around 0.7
- Training and testing matrices share the same overlapped gene set after preprocessing, and inputs are valid SingleCellExperiment objects with non-duplicated feature symbols
- Per-method confusion matrix against published true labels confirms the compiled benchmark table values
- Predicted label vector length equals the number of cells in the testing dataset

## Limitations

- NeuCA is restricted to settings where all cell types in the new sample already exist in the reference/training data
- Cross-condition prediction success was shown on the ASD data application and may not hold in other scenarios
- CHETAH produces an unignorable amount of unassigned/intermediate cell labels even when no unknown types are expected
- No version numbers are given for keras, TensorFlow, or R in the software description
- Unknown/rare or malignant cells should be identified beforehand (e.g., with inferCNV or copyKAT) since NeuCA assigns exhaustive labels

## Evidence

- [abstract] Cross-condition setting: "the applications on eight real datasets show that NeuCA has stable performance for intra- and inter-study annotation, as well as cross-condition annotation"
- [methods] Correlation threshold strategy: [evidence span withheld — anchor status: not_found]
- [methods] Preprocessing step: [evidence span withheld — anchor status: not_found]
- [intro] Workflow: correlation matrix: "NeuCA first obtains the mean gene expression profile for each cell type and calculates the correlation matrix across cell types"
- [readme] README invocation: "predicted.label = NeuCA(train = Baron_sce, test = Seg_sce, model.size = "big", verbose = FALSE)"
- [intro] Supervised superiority: "when training data are available, the improved accuracy, speed, and reproducibility of supervised methods easily outperform unsupervised ones"
