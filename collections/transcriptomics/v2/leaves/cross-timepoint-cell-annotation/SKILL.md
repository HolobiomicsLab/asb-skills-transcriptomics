---
name: cross-timepoint-cell-annotation
description: Use when you have labeled scRNA-seq data at an initial time point (t0)
  and unlabeled expression data at a later time point (t1) from the same study, and
  you need to assign cell-type labels to the t1 cells (e.g., pre- vs post-treatment
  samples).
when_to_use_negative:
- The initial time point's data is dramatically different from the later time point's
  data — the neural network would fail to borrow useful information and could not
  be trained well.
- You only have a single time point (cross-sectional data) with no t0/t1 split — standard
  supervised annotation tools or unsupervised clustering apply instead.
- You want to detect cell types absent from the t0 label set as part of annotation
  — that requires the separate DetectNovelCells pipeline (corr.cutoff=0.4, p.cutoff=0.01),
  since the classifier only assigns known t0 labels.
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: CASi
  role: provides CreateCASiList to assemble t0/t1 counts plus t0 labels and CrossTimeAnnotation
    to train the classifier and transfer labels
  repo: github.com/yizhuo-wang/CASi
- name: keras
  role: front-end used to define and train the feed-forward neural network classifier
    (256/128/64 units, 0.4 dropout, softmax output)
- name: TensorFlow
  role: backend required for the Keras model; must be set up before using CASi
- name: Seurat
  role: used for UMAP visualization of annotated t1 cells (DrawUMAP=TRUE)
- name: scmap
  role: competing supervised annotation method benchmarked on the identical t0/t1
    splits for accuracy and ARI
- name: CHETAH
  role: competing supervised annotation method benchmarked on the identical t0/t1
    splits for accuracy and ARI
- name: scPred
  role: competing supervised annotation method benchmarked in the PBMC simulation
    scenarios
provenance:
  source_task_ids:
  - task_001
  - task_002
  - task_007
  - task_006
  - task_004
  source_papers:
  - doi: 10.1038/s41598-024-58566-x
    title: 'CASi: A framework for cross-timepoint analysis of single-cell RNA sequencing
      data'
  source_content_hash: sha256:c0fb766848bd3593f4993c6afef7a422ddcdba3bf038abc1abfa706bdbe081d1
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CAMLU
  - CASi
  - CHETAH
  - Seurat
  - TensorFlow
  - ggplot2
  - keras
  - scPred
  - scmap
  - singleR
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# cross-timepoint-cell-annotation

## Summary

CASi's cross-timepoint cell annotation trains a Keras/TensorFlow neural network on labeled cells from an initial time point (t0) and transfers those labels to scRNA-seq cells from later time points (t1), borrowing information from earlier data in a stepwise fashion.

## When to use

Use when you have labeled scRNA-seq data at an initial time point (t0) and unlabeled expression data at a later time point (t1) from the same study, and you need to assign cell-type labels to the t1 cells (e.g., pre- vs post-treatment samples).

## When NOT to use

- The initial time point's data is dramatically different from the later time point's data — the neural network would fail to borrow useful information and could not be trained well.
- You only have a single time point (cross-sectional data) with no t0/t1 split — standard supervised annotation tools or unsupervised clustering apply instead.
- You want to detect cell types absent from the t0 label set as part of annotation — that requires the separate DetectNovelCells pipeline (corr.cutoff=0.4, p.cutoff=0.01), since the classifier only assigns known t0 labels.

## Inputs

- gene expression matrix (counts) of the initial time point t0
- gene expression matrix (counts) of the following time point t1
- cell labels of the initial time point t0 (supervised labels)
- pbmc_demo dataset shipped with the CASi package (for demonstration)

## Outputs

- per-cell predicted labels for t1 cells (e.g., written to ann_predictions.csv)
- annotated CASi list object with t1 cell labels
- serialized Keras model
- optional UMAP plots colored by cell labels (DrawUMAP=TRUE)
- benchmark table of accuracy and ARI versus scmap/CHETAH/scPred (e.g., annotation_benchmark_results.csv)

## How to apply

Build a CASi list with CreateCASiList(t0_count, t1_count, t0_label), restricting features to the default 2000 top highly variable genes; both time-point matrices are min-max normalized. Run CrossTimeAnnotation, which trains a feed-forward Keras network (TensorFlow backend) with three hidden layers of 256, 128, and 64 ReLU units, each with a 0.4 dropout layer to prevent overfitting, ending in a softmax output over the known t0 labels. Compile with MSE loss and the Adam optimizer, train on t0 cells in mini-batches with early stopping on validation loss, then predict t1 labels. Benchmark the transferred labels against competing supervised methods (scmap, CHETAH, scPred, and optionally singleR/CAMLU) on the identical t0/t1 splits using per-cell accuracy and adjusted Rand index (ARI).

## Related tools

- **CASi** (provides CreateCASiList to assemble t0/t1 counts plus t0 labels and CrossTimeAnnotation to train the classifier and transfer labels) — github.com/yizhuo-wang/CASi
- **keras** (front-end used to define and train the feed-forward neural network classifier (256/128/64 units, 0.4 dropout, softmax output))
- **TensorFlow** (backend required for the Keras model; must be set up before using CASi)
- **Seurat** (used for UMAP visualization of annotated t1 cells (DrawUMAP=TRUE))
- **scmap** (competing supervised annotation method benchmarked on the identical t0/t1 splits for accuracy and ARI)
- **CHETAH** (competing supervised annotation method benchmarked on the identical t0/t1 splits for accuracy and ARI)
- **scPred** (competing supervised annotation method benchmarked in the PBMC simulation scenarios)

## Examples

```
library(CASi); casi_list <- CreateCASiList(t0_count, t1_count, t0_label); annotated <- CrossTimeAnnotation(casi_list)
```

## Evaluation signals

- CrossTimeAnnotation is only runnable after CreateCASiList successfully combines t0_count, t1_count, and t0_label with the default nfeature=2000 highly variable genes
- In simulation scenarios 1 and 2, annotation accuracy and ARI should exceed scmap, CHETAH, and scPred over the 200 Monte Carlo repetitions
- In the MCL real-data application, CASi's accuracy and ARI when mapping pre-treatment labels onto post-treatment data should be the highest among compared methods
- Per-cell prediction output must contain a predicted label for every t1 cell, with labels drawn from the t0 label set
- With DrawUMAP=TRUE, the UMAP of t1 cells should show label-consistent clustering matching the reported favorable performance

## Limitations

- If the initial timepoint's data differs dramatically from later timepoint data, the neural network fails to borrow useful information and cannot be well trained
- The current annotation step pools data from all samples together for training and applying the network, which may be inappropriate when cell populations differ across conditions
- Requirements: TensorFlow must be set up and the keras package available before using CASi

## Evidence

- [intro] Our framework implements a neural network classifier to assign cell labels in a supervised way, and it can achieve high accuracy in the cross-timepoint setting.: "Our framework implements a neural network classifier to assign cell labels in a supervised way, and it can achieve high accuracy in the cross-timepoint setting."
- [methods] CrossTimeAnnotation parameters: "Below are the input parameters of the CrossTimeAnnotation function"
- [readme] CASi package description: "CASi is a comprehensive R package for analyzing multi-timepoint scRNA-seq data, which provides users with: (1) cross-time points cell annotation"
- [readme] README inputs: "Gene expression matrix of the initial data point/t0."
- [discussion] discussion limitation: "If the initial timepoint’s data is dramatically different from later timepoint’s data, the neural network would fail to borrow useful information and could not be very well trained"
- [readme] README setup requirement: [evidence span withheld — anchor status: external_doc]
