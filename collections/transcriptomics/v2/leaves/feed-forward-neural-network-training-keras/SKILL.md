---
name: feed-forward-neural-network-training-keras
description: Use when training data with known cell-type labels exist and the Pearson
  correlation matrix over averaged K cell-type profiles has all off-diagonal values
  below tau = 0.95 (no highly correlated types).
when_to_use_negative:
- Training cell types are highly correlated (any off-diagonal Pearson correlation
  >= tau = 0.95) — use NeuCA's hierarchical cell-lineage branch instead
- Novel/unknown cell types are expected in the test sample — NeuCA only assigns the
  K labels present in the reference/training data
- No labeled training data exist for the tissue — supervised methods do not apply;
  unsupervised clustering is required
edam_operation: http://edamontology.org/operation_2995
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3170
tools:
- name: NeuCA
  role: R/Bioconductor package implementing the feed-forward classifier, correlation-gated
    strategy selection, and exhaustive label assignment
  repo: https://bioconductor.org/packages/NeuCA/
- name: keras
  role: R API used by NeuCA for high-level feed-forward neural network implementation
    (RMSprop, mini-batch, categorical cross-entropy)
- name: SingleCellExperiment
  role: Bioconductor class required to format training and testing scRNA-seq matrices
    as NeuCA input
- name: R
  role: Runtime environment in which NeuCA and its keras dependency execute
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41598-021-04473-4
    title: A neural network-based method for exhaustive cell label assignment using
      single cell RNA-seq data
  source_content_hash: sha256:5ecaaffc4aa72ca2d94c615b1e49e5e895846945b9bacdf145da0cf4730330b6
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - NeuCA
  - R
  - SingleCellExperiment
  - TensorFlow
  - keras
  edam_operation: http://edamontology.org/operation_2995
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# feed-forward-neural-network-training-keras

## Summary

Trains a supervised feed-forward neural network classifier in R via keras (TensorFlow backend) to assign scRNA-seq cell labels exhaustively, as implemented in NeuCA's correlation-gated feed-forward branch.

## When to use

Use when training data with known cell-type labels exist and the Pearson correlation matrix over averaged K cell-type profiles has all off-diagonal values below tau = 0.95 (no highly correlated types).

## When NOT to use

- Training cell types are highly correlated (any off-diagonal Pearson correlation >= tau = 0.95) — use NeuCA's hierarchical cell-lineage branch instead
- Novel/unknown cell types are expected in the test sample — NeuCA only assigns the K labels present in the reference/training data
- No labeled training data exist for the tissue — supervised methods do not apply; unsupervised clustering is required

## Inputs

- Training scRNA-seq count/expression matrix with known cell-type labels (SingleCellExperiment with normcounts assay and colData labels)
- Testing scRNA-seq matrix to be labeled (SingleCellExperiment; true labels optional)
- model.size choice: 'small', 'medium', or 'big'

## Outputs

- predicted.label: a vector of the same length as the number of testing cells, each entry one of the K training-defined cell type labels
- Optional confusion matrix of predicted vs. true test labels for accuracy evaluation
- Trained keras/TensorFlow classifier (e.g., the shared pre-trained human PBMC classifier)

## How to apply

Normalize the training matrix X0 and testing matrix X1, then subset both to their overlapped genes and format each as a SingleCellExperiment object with cell labels in colData and feature_symbol in rowData (removing duplicated gene names). Compute the averaged G x K expression profile per cell type and the Pearson correlation matrix P; adopt the feed-forward branch only when every off-diagonal element of P is below tau = 0.95, otherwise switch to NeuCA's hierarchical cell-lineage strategy. Select network size via model.size ('small' = 3 hidden layers/64 nodes, 'medium' = 4 layers/64,128, 'big' = 5 layers/64,128,256) built as ReLU hidden layers with a softmax output layer, trained with RMSprop on mini-batches minimizing categorical cross-entropy. Run one NeuCA(train, test, model.size, verbose) call; keras serves as the R API to TensorFlow for the network implementation. Rationale: with moderately correlated types, a single network exhaustively assigns every test cell one of the K training labels without unassigned or intermediate labels.

## Related tools

- **NeuCA** (R/Bioconductor package implementing the feed-forward classifier, correlation-gated strategy selection, and exhaustive label assignment) — https://bioconductor.org/packages/NeuCA/
- **keras** (R API used by NeuCA for high-level feed-forward neural network implementation (RMSprop, mini-batch, categorical cross-entropy))
- **SingleCellExperiment** (Bioconductor class required to format training and testing scRNA-seq matrices as NeuCA input)
- **R** (Runtime environment in which NeuCA and its keras dependency execute)

## Examples

```
predicted.label = NeuCA(train = Baron_sce, test = Seg_sce, model.size = "big", verbose = FALSE)
```

## Evaluation signals

- Every test cell receives exactly one of the K training-defined labels — no unassigned or intermediate labels in predicted.label
- length(predicted.label) equals the number of cells in the testing dataset
- Correlation gate honored: the feed-forward network was only trained after confirming all off-diagonal elements of P are below tau = 0.95
- Confusion matrix of predicted.label vs. known test labels shows most mass on the diagonal (README example: alpha 328/331, beta 109/109, gamma 132/132)
- Network architecture matches the chosen model.size setting (e.g., 'big' = 5 ReLU hidden layers with 64, 128, 256 nodes)

## Limitations

- NeuCA is designed for well-studied settings where all cell types in the new sample already exist in the reference/training data
- Cross-condition prediction worked for the ASD data application but may not hold in other scenarios
- No version numbers are given for keras, TensorFlow, or R in the software description (README session info shows R 4.0.3, keras 2.4.0, tensorflow 2.4.0 for one installation)

## Evidence

- [discussion] keras API for NN training: "NeuCA uses keras as the application programming interface (API) to enable fast high-level neural network implementation."
- [intro] supervised training/prediction paradigm: "It uses existing scRNA-seq data with known labels to train a neural network-based classifier and then predict cell labels in new data of interest"
- [abstract] rationale for hierarchical vs feed-forward choice: "NeuCA can utilize the hierarchical structure information of the cell types to improve the annotation accuracy, which is especially helpful when data contain closely correlated cell types"
- [readme] concrete one-line invocation: "predicted.label = NeuCA(train = Baron_sce, test = Seg_sce, model.size = "big", verbose = FALSE)"
- [readme] keras dependency role: "NeuCA depends on the following packages in R/Bioconductor: - **keras**, for neural-network interface in R"
