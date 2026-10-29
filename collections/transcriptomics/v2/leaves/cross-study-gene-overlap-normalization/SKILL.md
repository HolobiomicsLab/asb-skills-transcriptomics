---
name: cross-study-gene-overlap-normalization
description: Use when a classifier trained on one scRNA-seq study must predict labels
  on an independent dataset (different platform/study), since the two count matrices
  typically contain different gene sets that must first be normalized and intersected.
when_to_use_negative:
- When training and testing data come from the same study with identical gene panels
  and no cross-platform gene-set mismatch exists
- When no labeled reference/training data exist — unsupervised approaches are the
  only viable solution in that setting
- When the new sample likely contains cell types absent from the reference; NeuCA
  assumes all cell types in the new sample already exist in the training data
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: NeuCA
  role: Downstream supervised classifier whose neural-network input requires train
    and test matrices reduced to the overlapped gene set
  repo: https://bioconductor.org/packages/NeuCA
- name: SingleCellExperiment
  role: Bioconductor container holding the normalized ('normcounts') and gene-overlap-subsetted
    matrices plus cell labels in colData
- name: R
  role: Language in which normalization, gene-set intersection, and SCE construction
    are performed
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41598-021-04473-4
    title: A neural network-based method for exhaustive cell label assignment using
      single cell RNA-seq data
  source_content_hash: sha256:731db039a97772b075597ff5da26f41b0e47a3b389e439a9a653389e93ed6bd9
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - NeuCA
  - R
  - SingleCellExperiment
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# cross-study-gene-overlap-normalization

## Summary

A preprocessing skill for supervised cross-study scRNA-seq label transfer (e.g., NeuCA on Baron/Muraro/Seg/Xin pancreas data) in which each train/test pair's expression matrices are normalized and both objects subset to the genes shared between studies, so the classifier and the queries live in a common feature space.

## When to use

Use when a classifier trained on one scRNA-seq study must predict labels on an independent dataset (different platform/study), since the two count matrices typically contain different gene sets that must first be normalized and intersected.

## When NOT to use

- When training and testing data come from the same study with identical gene panels and no cross-platform gene-set mismatch exists
- When no labeled reference/training data exist — unsupervised approaches are the only viable solution in that setting
- When the new sample likely contains cell types absent from the reference; NeuCA assumes all cell types in the new sample already exist in the training data

## Inputs

- Raw or filtered scRNA-seq count matrices from two or more studies (e.g., Baron GSE84133, Muraro GSE85241, Seg E-MTAB-5061, Xin GSE81608)
- Published true cell-type labels for the training (and optionally testing) data
- Gene symbol identifiers present as rownames of both matrices

## Outputs

- Normalized train/test expression matrices subset to the shared gene set (SingleCellExperiment objects with a 'normcounts' assay)
- Feature-annotated SCE objects (rowData$feature_symbol set, duplicated genes removed) ready for NeuCA(train = ..., test = ...)

## How to apply

For each train-test pair, first normalize the expression matrices (the NeuCA workflow expects data supplied as the 'normcounts' assay of a SingleCellExperiment object). Then subset both training and testing data to the overlapped genes between the two studies, so feature dimensionality matches for the neural-network classifier. Remove features with duplicated gene names before building the SingleCellExperiment (rowData$feature_symbol <- rownames). NeuCA then computes per-cell-type mean expression profiles and their Pearson correlation matrix, applying the tau = 0.95 threshold to choose between a feed-forward neural network and a cell-lineage-based hierarchical neural network; this model choice is only valid if both matrices were reduced to the same gene universe. Cross-study benchmarks such as Baron->Seg train on one dataset and test on the remaining studies, so this overlap step must be repeated per pair.

## Related tools

- **NeuCA** (Downstream supervised classifier whose neural-network input requires train and test matrices reduced to the overlapped gene set) — https://bioconductor.org/packages/NeuCA
- **SingleCellExperiment** (Bioconductor container holding the normalized ('normcounts') and gene-overlap-subsetted matrices plus cell labels in colData)
- **R** (Language in which normalization, gene-set intersection, and SCE construction are performed)

## Examples

```
Baron_sce <- SingleCellExperiment(assays = list(normcounts = as.matrix(Baron_counts)), colData = Baron_anno); rowData(Baron_sce)$feature_symbol <- rownames(Baron_sce); Baron_sce <- Baron_sce[!duplicated(rownames(Baron_sce)), ]
```

## Evaluation signals

- Training and testing matrices have identical rownames (gene symbols) after subsetting, with no duplicated feature symbols remaining
- Each SCE object carries a 'normcounts' assay and cell labels in colData before NeuCA is called
- Downstream NeuCA prediction on the held-out study returns a label vector of length equal to the number of test cells
- On the pancreas benchmark, after this preprocessing NeuCA attains the highest accuracy in 9 of 12 train-test scenarios and >90% accuracy on Baron->Seg

## Limitations

- Overlapping genes across distant platforms can greatly reduce the feature space, which may affect classifier accuracy
- NeuCA (and hence this preprocessing) is restricted to settings where all cell types in the new sample already exist in the reference/training data
- No software version numbers for keras/TensorFlow/R are given in the paper, so reproducing the exact normalization environment requires care

## Evidence

- [methods] Preprocessing: normalize expression matrix and subset training and testing data by overlapped genes: "After the preprocessing steps, such as normalizing the expression matrix and taking subsets of"
- [readme] SCE construction and duplicate-gene removal: "rowData(Baron_sce)$feature_symbol <- rownames(Baron_sce)
# remove features with duplicated names
Baron_sce <- Baron_sce[!duplicated(rownames(Baron_sce)), ]"
- [readme] assay and label organization expected by NeuCA: "assays = list(normcounts = as.matrix(Baron_counts)),
    colData = Baron_anno"
- [intro] supervised-vs-unsupervised scope statement: "when training data are available, the improved accuracy, speed, and reproducibility of supervised methods easily outperform unsupervised ones"
