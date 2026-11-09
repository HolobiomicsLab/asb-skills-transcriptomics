---
name: min-max-gene-expression-normalization
description: 'Use when preparing t0/t1 scRNA-seq count matrices for CASi''s CrossTimeAnnotation
  neural network: features must first be restricted to the top 2000 highly variable
  genes and min-max normalized identically across timepoints so the ANN trains stably.'
when_to_use_negative:
- The initial timepoint's data is dramatically different from later timepoints' data
  — the neural network would fail to borrow useful information and could not be well
  trained, so normalization alone will not rescue the annotation
- You are pooling samples from multiple conditions for annotation — CASi's annotation
  step pools data from all samples together, which may be inappropriate when cell
  populations differ across conditions
- You only have a single timepoint with no cross-timepoint prediction goal — standard
  scRNA-seq normalization workflows are the appropriate choice instead
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: CASi
  role: R framework whose CrossTimeAnnotation pipeline performs the highly-variable-gene
    restriction and min-max normalization before training the neural network classifier
  repo: https://github.com/yizhuo-wang/CASi
- name: keras
  role: R interface used to build and train the feed-forward network that consumes
    the normalized expression matrices
- name: TensorFlow
  role: Backend required for the Keras model used in cross-timepoint cell annotation
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41598-024-58566-x
    title: 'CASi: A framework for cross-timepoint analysis of single-cell RNA sequencing
      data'
  source_content_hash: sha256:c263c70c2956eba690e52b818581cf96757e52e019e9dc355558bea9f917e3c3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/yizhuo-wang/CASi
  tools:
  - CASi
  - TensorFlow
  - keras
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# min-max-gene-expression-normalization

## Summary

Rescale single-cell gene expression values to a common [0, 1] range on a shared set of highly variable genes before feeding them into CASi's Keras neural network classifier for cross-timepoint cell annotation.

## When to use

Use when preparing t0/t1 scRNA-seq count matrices for CASi's CrossTimeAnnotation neural network: features must first be restricted to the top 2000 highly variable genes and min-max normalized identically across timepoints so the ANN trains stably.

## When NOT to use

- The initial timepoint's data is dramatically different from later timepoints' data — the neural network would fail to borrow useful information and could not be well trained, so normalization alone will not rescue the annotation
- You are pooling samples from multiple conditions for annotation — CASi's annotation step pools data from all samples together, which may be inappropriate when cell populations differ across conditions
- You only have a single timepoint with no cross-timepoint prediction goal — standard scRNA-seq normalization workflows are the appropriate choice instead

## Inputs

- Gene expression matrix of the initial data point/t0 (cells with known labels)
- Gene expression matrix of the following data point/t1
- (optional) Cell labels of the initial data point

## Outputs

- Min-max normalized expression matrices for t0 and t1 restricted to the top 2000 highly variable genes
- Normalized features suitable as input to the Keras ANN cell-annotation classifier

## How to apply

Start from the gene expression matrix of the initial time point (t0, with known cell labels) and the following time point (t1), e.g. the pbmc_demo dataset lazy-loaded with the CASi package. Restrict features to the top 2000 highly variable genes (the nfeature default), since CASi's classifier expects this reduced gene space. Apply min-max normalization to the expression values of BOTH time points so they share a comparable [0, 1] scale — the normalization must be consistent across timepoints because the network trained on t0 is used to predict t1 labels. Feed the normalized matrices into the Keras/TensorFlow feed-forward network (hidden layers of 256, 128, 64 ReLU units with 0.4 dropout, softmax output), compiled with MSE loss and the Adam optimizer, trained with mini-batches and early stopping on validation loss. The rationale is that the neural network's training is sensitive to feature scale, so unscaled raw counts would destabilize the supervised label assignment.

## Related tools

- **CASi** (R framework whose CrossTimeAnnotation pipeline performs the highly-variable-gene restriction and min-max normalization before training the neural network classifier) — https://github.com/yizhuo-wang/CASi
- **keras** (R interface used to build and train the feed-forward network that consumes the normalized expression matrices)
- **TensorFlow** (Backend required for the Keras model used in cross-timepoint cell annotation)

## Examples

```
require(keras); require(CASi); data(pbmc_demo)  # then min-max scale t0/t1 within CASi's CrossTimeAnnotation workflow
```

## Evaluation signals

- All normalized expression values for both timepoints fall within [0, 1] after min-max scaling
- Feature dimension of the normalized matrices equals the top 2000 highly variable genes (nfeature default), with the same gene set retained across t0 and t1
- The downstream Keras model compiles with MSE loss and the Adam optimizer and trains with early stopping on validation loss rather than diverging
- The trained network assigns cell labels to t1 cells (CrossTimeAnnotation-style prediction) with high cross-timepoint accuracy, consistent with the paper's reported supervised-annotation performance

## Limitations

- If the initial timepoint's data is dramatically different from later timepoint's data, the neural network would fail to borrow useful information and could not be very well trained
- The current cell type annotation step pools the data from all the samples together for training and applying the neural network, which may be inappropriate when cell populations differ across conditions
- CASi requires TensorFlow setup and the keras package to be available before use, per the README installation instructions

## Evidence

- [methods] Neural-network input prep: [evidence span withheld — anchor status: not_found]
- [intro] Neural network classifier: "Our framework implements a neural network classifier to assign cell labels in a supervised way, and it can achieve high accuracy in the cross-timepoint setting."
- [readme] TensorFlow/keras dependency: [evidence span withheld — anchor status: external_doc]
- [methods] Demo data availability: "The demo pbmc data, "pbmc_demo", has been lazy-loaded with the package."
