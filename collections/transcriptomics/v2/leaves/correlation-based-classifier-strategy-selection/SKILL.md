---
name: correlation-based-classifier-strategy-selection
description: Use when training a supervised cell-type annotator (NeuCA) on scRNA-seq
  data with known labels and you must decide whether high inter-cell-type correlations
  require a hierarchical neural-network strategy instead of a plain feed-forward classifier.
when_to_use_negative:
- The new sample may contain cell types or unknown/rare populations absent from the
  reference/training data — NeuCA is designed for settings where all cell types in
  the new sample already exist in the training data
- No training data with known labels exist for the tissue of interest — unsupervised
  methods (e.g., Seurat, SC3) are then the only viable solution
- Closely correlated cell types are not the concern and a simple correlated-procedure
  or feed-forward classifier suffices when all off-diagonal correlations are below
  tau = 0.95
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: NeuCA
  role: 'Implements the correlation-based strategy selection: computes cell-type mean
    profiles, Pearson correlation matrix, and switches between feed-forward and hierarchical
    neural networks at tau = 0.95'
  repo: https://bioconductor.org/packages/NeuCA
- name: keras
  role: Neural-network interface used by NeuCA to build either the feed-forward or
    hierarchical classifier
- name: limma
  role: Provides linear model framework and marker testing used by the marker-guided
    hierarchical network branch
- name: SingleCellExperiment
  role: Required input data class holding normalized counts and labels for train/test
    sets
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41598-021-04473-4
    title: A neural network-based method for exhaustive cell label assignment using
      single cell RNA-seq data
  source_content_hash: sha256:515ea8b24bb5a39962584d96675b3c52598e731150f76bf5899b5ad2ace595bd
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - NeuCA
  - R
  - SC3
  - Seurat
  - SingleCellExperiment
  - keras
  - limma
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# correlation-based-classifier-strategy-selection

## Summary

NeuCA's automatic decision rule for choosing between a feed-forward neural network and a marker-guided, cell lineage-based hierarchical neural network in supervised scRNA-seq cell annotation. It computes averaged expression profiles per cell type, builds their Pearson correlation matrix, and applies a tau = 0.95 threshold so that closely correlated cell types are classified with hierarchical structure information.

## When to use

Use when training a supervised cell-type annotator (NeuCA) on scRNA-seq data with known labels and you must decide whether high inter-cell-type correlations require a hierarchical neural-network strategy instead of a plain feed-forward classifier.

## When NOT to use

- The new sample may contain cell types or unknown/rare populations absent from the reference/training data — NeuCA is designed for settings where all cell types in the new sample already exist in the training data
- No training data with known labels exist for the tissue of interest — unsupervised methods (e.g., Seurat, SC3) are then the only viable solution
- Closely correlated cell types are not the concern and a simple correlated-procedure or feed-forward classifier suffices when all off-diagonal correlations are below tau = 0.95

## Inputs

- Training SingleCellExperiment object with normcounts assay and known cell-type labels in colData (e.g., Baron_scRNA)
- Testing SingleCellExperiment object with normcounts assay (labels optional, e.g., Seg_scRNA)
- model.size argument: 'big', 'medium', or 'small'

## Outputs

- Selected classification strategy (feed-forward neural network vs marker-guided hierarchical neural network)
- Vector of predicted cell-type labels, one per cell in the testing dataset

## How to apply

From the training data, compute the averaged (mean) expression profile matrix for all cell types, then compute the Pearson correlation matrix P = (p_ij) across the cell-type profiles. Apply the correlation threshold tau = 0.95: if all off-diagonal elements of P are smaller than tau, adopt the feed-forward neural network; otherwise adopt the cell lineage-based hierarchical neural network, which uses cell type tree/lineage structure and feature selection to improve classification of closely correlated cell types (e.g., alpha/beta/delta/gamma pancreatic endocrine cells in the Baron dataset). NeuCA performs this detection and selection automatically during NeuCA(train, test), so the curator/practitioner mainly verifies which strategy was selected and that model.size ("big", "medium", "small") matches the desired number of nodes and layers. Rationale: correlation-based methods alone struggle when pairwise correlations of several cell types are high simultaneously, and exploiting hierarchical structure improves annotation accuracy in those settings.

## Related tools

- **NeuCA** (Implements the correlation-based strategy selection: computes cell-type mean profiles, Pearson correlation matrix, and switches between feed-forward and hierarchical neural networks at tau = 0.95) — https://bioconductor.org/packages/NeuCA
- **keras** (Neural-network interface used by NeuCA to build either the feed-forward or hierarchical classifier)
- **limma** (Provides linear model framework and marker testing used by the marker-guided hierarchical network branch)
- **SingleCellExperiment** (Required input data class holding normalized counts and labels for train/test sets)

## Examples

```
predicted.label = NeuCA(train = Baron_sce, test = Seg_sce, model.size = "big", verbose = FALSE)
```

## Evaluation signals

- Confirm a Pearson correlation matrix across cell types was computed from averaged expression profiles and that tau = 0.95 was applied to off-diagonal elements
- Check which strategy was chosen: all off-diagonal correlations < 0.95 must yield the feed-forward network; any correlation >= 0.95 must yield the hierarchical network
- On a test set with true labels, evaluate a confusion matrix of predicted.label vs true labels; near-diagonal agreement (e.g., Baron/Seg example with only a few misassignments) indicates correct application
- Verify train/test SingleCellExperiment inputs have non-duplicated feature_symbol row names and normcounts assays
- In cross-condition benchmarks (e.g., ASD train->control and control->train), accuracy and ARI should remain stable and competitive across both directions

## Limitations

- NeuCA is designed for situations where all cell types in the new sample already exist in the reference/training data; it does not handle novel cell types
- The paper does not address deciding whether unknown/novel cell types exist in the data; consulting biological/clinical experts is recommended for novel-condition tissues
- The cross-condition success shown on ASD snRNA-seq data may not hold in other scenarios
- No version numbers are given for keras, TensorFlow, or R in the software description, complicating reproducibility of the trained networks

## Evidence

- [abstract] NeuCA can utilize the hierarchical structure information of the cell types to improve the annotation accuracy, which is especially helpful when data contain closely correlated cell types: "NeuCA can utilize the hierarchical structure information of the cell types to improve the annotation accuracy, which is especially helpful when data contain closely correlated cell types"
- [methods] Correlation-based strategy selection with tau = 0.95 threshold: [evidence span withheld — anchor status: not_found]
- [methods] NeuCA computes averaged expression profile matrix for all cell types in the training data: "NeuCA first computes the averaged expression profile matrix for all the cell types in the training data"
- [readme] NeuCA automatically detects highly-correlated cell types and chooses the neural-network model: "NeuCA can detect whether highly-correlated cell types exist in the training dataset, and automatically determine if a general neural-network model will be adopted or a marker-guided hierarchical"
- [intro] High-correlation failure mode motivating the hierarchical strategy: "The methods in this category may have difficulty accurately assigning labels under these high correlation settings"
