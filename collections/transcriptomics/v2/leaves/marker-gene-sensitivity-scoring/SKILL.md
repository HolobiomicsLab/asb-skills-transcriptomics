---
name: marker-gene-sensitivity-scoring
description: Use when NeuCA's hierarchical branch is triggered — off-diagonal Pearson
  correlations among cell-type mean profiles exceed tau = 0.95 — and test cells must
  be pre-assigned from highly sensitive markers before tree traversal.
when_to_use_negative:
- All off-diagonal elements of the cell-type correlation matrix P are below tau =
  0.95 — NeuCA switches to a single feed-forward network and the marker-guided hierarchical
  branch is not used
- The test sample may contain novel or unknown cell types not present in the training
  reference — NeuCA assumes every test cell belongs to one of the K training-defined
  types
- No labeled training data exist for the tissue — supervised pre-assignment cannot
  be performed and unsupervised methods are required
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: limma
  role: Linear-model framework used to test and rank bifurcation-specific marker features
    at each node of the lineage tree
- name: NeuCA
  role: Host R/Bioconductor package that computes marker sensitivity scores and runs
    the marker-guided hierarchical neural-network annotation
  repo: https://bioconductor.org/packages/NeuCA/
- name: SingleCellExperiment
  role: Required Bioconductor container for the training and testing scRNA-seq matrices
    fed into the marker-scoring pipeline
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41598-021-04473-4
    title: A neural network-based method for exhaustive cell label assignment using
      single cell RNA-seq data
  source_content_hash: sha256:8b8403574c31f1111bcfa378ff8b3fd3ea14d71cfb810fa5c5c7f43da3aae7fc
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - NeuCA
  - R
  - SingleCellExperiment
  - TensorFlow
  - limma
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# marker-gene-sensitivity-scoring

## Summary

Scores cell-type-specific marker genes by sensitivity (Sen_gk > 0.95, i.e. expressed in >95% of cells of that type) from the averaged expression profile, and uses markers to pre-assign test cells before NeuCA's lineage-tree bifurcation classification, improving accuracy for closely correlated cell types.

## When to use

Use when NeuCA's hierarchical branch is triggered — off-diagonal Pearson correlations among cell-type mean profiles exceed tau = 0.95 — and test cells must be pre-assigned from highly sensitive markers before tree traversal.

## When NOT to use

- All off-diagonal elements of the cell-type correlation matrix P are below tau = 0.95 — NeuCA switches to a single feed-forward network and the marker-guided hierarchical branch is not used
- The test sample may contain novel or unknown cell types not present in the training reference — NeuCA assumes every test cell belongs to one of the K training-defined types
- No labeled training data exist for the tissue — supervised pre-assignment cannot be performed and unsupervised methods are required

## Inputs

- Training scRNA-seq data as a SingleCellExperiment object (normcounts assay, colData with known cell labels, rowData$feature_symbol, duplicated gene rows removed)
- Testing scRNA-seq data as a SingleCellExperiment object (same format; true labels not required)
- Averaged expression profile matrix A (G x K) computed from the training data

## Outputs

- Set of highly sensitive markers per cell type (Sen_gk > 0.95)
- Pre-assigned labels for test cells expressing > nu2 = 3 markers of a qualifying type
- Leaf labels for remaining cells traversed through the bifurcation tree
- A predicted.label vector (one label per test cell), e.g. written to cell_labels.csv

## How to apply

From the normalized, gene-overlapped training matrix, compute the averaged G x K expression profile matrix A (one mean profile per cell type) and derive the per-type sensitivity score Sen_gk, keeping genes with Sen_gk > 0.95 as highly sensitive markers. If a cell type has at least nu1 = 10 such markers, pre-assign any test cell expressing more than nu2 = 3 of them to that type. The remaining cells traverse the cell lineage tree built by complete-linkage hierarchical clustering on the columns of A; at each bifurcation, the top M = 1000 differentially expressed features are selected with limma's linear-model framework, and a binary feed-forward network trained with binary cross-entropy assigns the leaf label. This marker-guided feature selection exploits hierarchical cell-type structure so that closely correlated types are separated by bifurcation-specific markers rather than a single softmax output.

## Related tools

- **limma** (Linear-model framework used to test and rank bifurcation-specific marker features at each node of the lineage tree)
- **NeuCA** (Host R/Bioconductor package that computes marker sensitivity scores and runs the marker-guided hierarchical neural-network annotation) — https://bioconductor.org/packages/NeuCA/
- **SingleCellExperiment** (Required Bioconductor container for the training and testing scRNA-seq matrices fed into the marker-scoring pipeline)

## Examples

```
predicted.label = NeuCA(train = Baron_sce, test = Seg_sce, model.size = "big", verbose = FALSE)
```

## Evaluation signals

- Every selected marker has sensitivity Sen_gk > 0.95 within its cell type
- Only cell types with at least nu1 = 10 highly sensitive markers receive pre-assigned cells, and those cells express more than nu2 = 3 of the markers
- Every test cell receives exactly one of the K training-defined labels — no unassigned or intermediate labels
- Confusion matrix against known test labels is strongly diagonal (README example: 328/331 alpha cells correct, with Sankey diagram for visualization)
- Length of predicted.label equals the number of cells in the testing SingleCellExperiment object

## Limitations

- NeuCA is restricted to settings where all cell types in the new sample already exist in the reference/training data; unknown cell types are not detected
- Marker-based decisions are prone to marker misspecification if marker-gene knowledge is imperfect
- The single tau = 0.95 correlation threshold governs branch selection; cross-condition prediction may not hold in scenarios other than the demonstrated ASD application

## Evidence

- [methods] NeuCA first computes the averaged expression profile matrix for all the cell types in the training data: "NeuCA first computes the averaged expression profile matrix for all the cell types in the training data"
- [readme] Feature selection in hierarchical structure: "Feature selection is performed in hierarchical structure to further improve classification accuracy."
- [readme] limma role for markers: "limma, for linear model framework and testing markers"
- [abstract] Hierarchical information helps with correlated types: "NeuCA can utilize the hierarchical structure information of the cell types to improve the annotation accuracy, which is especially helpful when data contain closely correlated cell types"
- [intro] Marker misspecification risk: "Methods in this category require known knowledge of cell type-specific marker genes, and they thereby could be prone to marker misspecification"
