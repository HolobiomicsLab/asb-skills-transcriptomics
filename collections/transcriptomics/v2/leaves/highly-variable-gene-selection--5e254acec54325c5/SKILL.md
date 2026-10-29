---
name: highly-variable-gene-selection
description: Use when preparing scRNA-seq expression matrices from multiple time points
  as features for supervised cross-timepoint cell-annotation, and you must reduce
  thousands of genes to a manageable, informative subset before neural-network training.
when_to_use_negative:
- Input is not a gene-by-cell expression matrix (e.g., already a reduced feature table
  or bulk RNA-seq data), so variability-based gene filtering does not apply.
- The initial timepoint's data is dramatically different from later timepoints' data
  — the article warns the neural network 'would fail to borrow useful information
  and could not be very well trained', so HVG features learned from t0 would not transfer.
- Cell populations differ across conditions and pooling matters — the article notes
  CASi's annotation step 'pools the data from all the samples together for training
  and applying the neural network', which may be inappropriate; a condition-aware
  feature selection would be needed.
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: CASi
  role: R package whose cross-timepoint annotation pipeline applies the top-2000 HVG
    restriction (nfeature default) before training its neural network classifier
  repo: github.com/yizhuo-wang/CASi
- name: keras
  role: R interface used to build and train the feed-forward network that consumes
    the HVG-restricted, min-max normalized features
- name: TensorFlow
  role: Backend that must be set up for keras before running CASi's annotation workflow
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41598-024-58566-x
    title: 'CASi: A framework for cross-timepoint analysis of single-cell RNA sequencing
      data'
  source_content_hash: sha256:bc1d722607b462da7b065ac67d57e67a05457f626210d317e738de1af8153da2
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CASi
  - TensorFlow
  - keras
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# Highly variable gene selection

## Summary

Feature-reduction step for CASi's cross-timepoint cell annotation: restrict single-cell gene expression matrices to the top 2000 highly variable genes (nfeature default) and min-max normalize them, so the Keras neural network classifier trains on a compact, discriminative feature set instead of the full transcriptome.

## When to use

Use when preparing scRNA-seq expression matrices from multiple time points as features for supervised cross-timepoint cell-annotation, and you must reduce thousands of genes to a manageable, informative subset before neural-network training.

## When NOT to use

- Input is not a gene-by-cell expression matrix (e.g., already a reduced feature table or bulk RNA-seq data), so variability-based gene filtering does not apply.
- The initial timepoint's data is dramatically different from later timepoints' data — the article warns the neural network 'would fail to borrow useful information and could not be very well trained', so HVG features learned from t0 would not transfer.
- Cell populations differ across conditions and pooling matters — the article notes CASi's annotation step 'pools the data from all the samples together for training and applying the neural network', which may be inappropriate; a condition-aware feature selection would be needed.

## Inputs

- Gene expression matrix of the initial data point/t0 (train_count, with cell labels train_label)
- Gene expression matrix of the following data points/t1 (test_count)
- pbmc_demo demo dataset lazy-loaded with the CASi package (for the demonstration workflow)

## Outputs

- Feature-restricted t0 and t1 expression matrices containing only the top 2000 highly variable genes, min-max normalized
- Reduced feature set consumed by the CrossTimeAnnotation neural network classifier, whose per-cell predicted labels are written to ann_predictions.csv

## How to apply

After preprocessing and QC of the scRNA-seq data, split the data into the initial time point (t0 train_count/train_label) and the following time points (t1 test_count). Rank genes by expression variability across the t0 training cells and retain only the top 2000 highly variable genes, CASi's nfeature default. Min-max normalize the expression values for both time points so all retained features share a common [0,1] scale, which is required by the downstream Keras (TensorFlow backend) feed-forward network (hidden layers of 256, 128, 64 units with ReLU activation and 0.4 dropout each, softmax output, MSE loss, Adam optimizer, early stopping on validation loss). Apply the identical gene subset and normalization consistently to t0 and t1 so the network can borrow information from the labeled earlier time point to assign labels to later cells. The rationale is that highly variable genes carry the discriminative signal for cross-timepoint label transfer while limiting input dimensionality and overfitting risk.

## Related tools

- **CASi** (R package whose cross-timepoint annotation pipeline applies the top-2000 HVG restriction (nfeature default) before training its neural network classifier) — github.com/yizhuo-wang/CASi
- **keras** (R interface used to build and train the feed-forward network that consumes the HVG-restricted, min-max normalized features)
- **TensorFlow** (Backend that must be set up for keras before running CASi's annotation workflow)

## Evaluation signals

- The retained feature matrix contains exactly 2000 genes (CASi's nfeature default), with gene identities and ordering identical between the t0 and t1 matrices.
- Both timepoint matrices are min-max normalized, with expression values bounded in [0,1] and consistent scaling across train and test features.
- Downstream Keras model trains with early stopping on validation loss; converged training without runaway validation loss indicates the reduced features were informative rather than degenerate.
- Per-cell predicted labels for t1 are produced and written to ann_predictions.csv together with the serialized Keras model.
- On the pbmc_demo PBMC dataset (more than 60,000 sorted cells from eight immune cell types), predicted t1 labels should recover the known immune cell types with high accuracy.

## Limitations

- If the initial timepoint's data is dramatically different from later timepoint's data, the neural network would fail to borrow useful information and could not be very well trained — HVG features selected from t0 will not generalize.
- The annotation step pools data from all samples together for training, which may be inappropriate when cell populations differ across conditions.
- Novel cell detection may fail when novel cells are transcriptomically similar to known cell types, so the 2000-HVG feature space may not isolate emergent cell types.

## Evidence

- [methods] HVG restriction feeding the classifier: [evidence span withheld — anchor status: not_found]
- [intro] Purpose of the neural network classifier using these features: "Our framework implements a neural network classifier to assign cell labels in a supervised way, and it can achieve high accuracy in the cross-timepoint setting."
- [methods] Demo dataset used in the feature-selection workflow: "The demo pbmc data, "pbmc_demo", has been lazy-loaded with the package."
- [readme] Input data type required for the workflow: "Gene expression matrix of the initial data point/t0."
- [readme] Backend setup required to run the classifier consuming the selected features: [evidence span withheld — anchor status: external_doc]
