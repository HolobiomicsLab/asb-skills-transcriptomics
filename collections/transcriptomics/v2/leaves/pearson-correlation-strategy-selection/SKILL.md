---
name: pearson-correlation-strategy-selection
description: 'Decision support for choosing NeuCA''s annotation route when a labelled
  training scRNA-seq matrix is available: compute cross-cell-type correlations and
  check whether any off-diagonal element reaches tau = 0.95.'
when_to_use_negative:
- Cell types in the new sample include unknown/novel or malignant populations not
  present in the training reference — NeuCA is designed for settings where all cell
  types already exist in the training data; identify such cells first (e.g. inferCNV
  or copyKAT) before applying this gate
- Fewer than two cell types in the training labels — the off-diagonal correlation
  comparison over P is undefined
- Only the test matrix is available with no labelled training data — supervised gating
  (and NeuCA itself) requires known labels to compute the averaged profiles
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: NeuCA
  role: 'Implements the gating: automatically computes the cell-type correlation matrix
    and decides between the general feed-forward network and the marker-guided hierarchical
    neural network'
  repo: https://bioconductor.org/packages/NeuCA
- name: R
  role: Runtime environment; NeuCA is an R/Bioconductor package used to compute the
    averaged profiles and Pearson correlation matrix
- name: SingleCellExperiment
  role: Bioconductor data container required for NeuCA's train/test inputs (normcounts
    assay plus colData labels)
provenance:
  source_task_ids:
  - task_002
  - task_001
  source_papers:
  - doi: 10.1038/s41598-021-04473-4
    title: A neural network-based method for exhaustive cell label assignment using
      single cell RNA-seq data
  source_content_hash: sha256:5a5d92fd1d1423d4796c1bbdf46d14bddb25177ee19005b36e1e15f7b867cc11
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - NeuCA
  - R
  - SingleCellExperiment
  - TensorFlow
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# pearson-correlation-strategy-selection

## Summary

NeuCA's data-driven gate that computes a Pearson correlation matrix across cell types from averaged gene expression profiles of the labelled training scRNA-seq data and selects the annotation route: a single feed-forward neural network when all off-diagonal correlations are below tau = 0.95, otherwise a marker-guided hierarchical (cell lineage tree) network.

## When to use

Decision support for choosing NeuCA's annotation route when a labelled training scRNA-seq matrix is available: compute cross-cell-type correlations and check whether any off-diagonal element reaches tau = 0.95.

## When NOT to use

- Cell types in the new sample include unknown/novel or malignant populations not present in the training reference — NeuCA is designed for settings where all cell types already exist in the training data; identify such cells first (e.g. inferCNV or copyKAT) before applying this gate
- Fewer than two cell types in the training labels — the off-diagonal correlation comparison over P is undefined
- Only the test matrix is available with no labelled training data — supervised gating (and NeuCA itself) requires known labels to compute the averaged profiles

## Inputs

- Labelled training scRNA-seq expression matrix X0 (e.g. Zheng_8ct_PBMC from DuoClustering2018) as a SingleCellExperiment object with normcounts assay and cell-type labels in colData
- Testing scRNA-seq expression matrix X1 (labels optional) as a SingleCellExperiment object
- Overlapped gene set between training and testing matrices

## Outputs

- G x K averaged expression profile matrix A (mean expression per cell type)
- Symmetric K x K Pearson correlation matrix P with unit diagonal (e.g. cell_type_correlation_matrix.csv)
- Route decision: 'feed-forward' if all off-diagonal p_i,j < 0.95, otherwise 'hierarchical'

## How to apply

In R with NeuCA and SingleCellExperiment, first preprocess both training matrix X0 and testing matrix X1 by normalizing the expression values and subsetting both to their overlapped genes. Compute the averaged expression profile for each of the K cell types by taking the mean expression of cells within each cell type, yielding a G x K profile matrix A. Compute the Pearson correlation matrix P = (p_i,j) across the K cell-type columns of A. Compare all off-diagonal elements of P against the threshold tau = 0.95: if none reach 0.95, NeuCA trains a single feed-forward neural network classifier; if highly correlated cell types exist, it instead builds a cell lineage tree by complete-linkage hierarchical clustering and trains bifurcation-specific binary networks. Record the K x K matrix (e.g. cell_type_correlation_matrix.csv) together with the route decision so the gating rule is auditable.

## Related tools

- **NeuCA** (Implements the gating: automatically computes the cell-type correlation matrix and decides between the general feed-forward network and the marker-guided hierarchical neural network) — https://bioconductor.org/packages/NeuCA
- **R** (Runtime environment; NeuCA is an R/Bioconductor package used to compute the averaged profiles and Pearson correlation matrix)
- **SingleCellExperiment** (Bioconductor data container required for NeuCA's train/test inputs (normcounts assay plus colData labels))

## Examples

```
predicted.label = NeuCA(train = Baron_sce, test = Seg_sce, model.size = "big", verbose = FALSE)
```

## Evaluation signals

- The correlation matrix P is a symmetric K x K matrix with unit diagonal (validity invariant of the Pearson computation)
- The recorded route decision matches the rule: feed-forward network used only when every off-diagonal element of P is strictly below tau = 0.95
- Averaged profile matrix A has G rows (genes in the overlapped gene set) and exactly K columns, one per training-defined cell type
- Downstream consistency: after the chosen branch runs, every test cell receives exactly one of the K training-defined labels (no unassigned or intermediate labels)

## Limitations

- NeuCA's gating assumes the new sample's cell types all exist in the training reference; it does not decide whether unknown/novel cell types are present — consult biological/clinical experts for that
- Cross-condition prediction may not hold in scenarios other than the ASD data application
- No version numbers are given for keras, TensorFlow, or R in the software description

## Evidence

- [intro] NeuCA first obtains the mean gene expression profile for each cell type and calculates the correlation matrix across cell types: "NeuCA first obtains the mean gene expression profile for each cell type and calculates the correlation matrix across cell types"
- [methods] Correlation-based strategy selection threshold: [evidence span withheld — anchor status: not_found]
- [readme] README auto-detection of correlated cell types: "NeuCA can detect whether highly-correlated cell types exist in the training dataset, and automatically determine if a general neural-network model will be adopted"
- [abstract] Hierarchical route rationale for closely correlated cell types: "NeuCA can utilize the hierarchical structure information of the cell types to improve the annotation accuracy, which is especially helpful when data contain closely correlated cell types"
