---
name: neural-network-classifier-training
description: Use when you have paired scRNA-seq count matrices from two timepoints
  (t0/t1), labels for the t0 cells, and need supervised cross-timepoint cell-type
  annotation of t1 cells.
when_to_use_negative:
- The initial timepoint's data is dramatically different from later timepoint's data
  — the neural network would fail to borrow useful information and could not be very
  well trained.
- The data is cross-sectional from a single time point with no t0/t1 pairing — use
  a standard supervised annotation method instead.
- You expect novel cell types that are transcriptomically similar to known cell types
  — annotation alone may misassign them; pair with DetectNovelCells and treat results
  cautiously.
edam_operation: http://edamontology.org/operation_0253
tools:
- name: CASi
  role: Provides CreateCASiList to build the t0/t1 input object and CrossTimeAnnotation
    to train and apply the neural network classifier for cross-timepoint cell labels
  repo: github.com/yizhuo-wang/CASi
- name: keras
  role: R Keras/TensorFlow dependency required by CrossTimeAnnotation to train and
    run the artificial neural network
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-024-58566-x
    title: 'CASi: A framework for cross-timepoint analysis of single-cell RNA sequencing
      data'
  source_content_hash: sha256:0d6d3d5b21e82c96be22fb2f70e91712dcbf5ec9990121caa052b0e4c0f52f6a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CASi
  - CHETAH
  - keras
  - scPred
  - scmap
  edam_operation: http://edamontology.org/operation_0253
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# neural-network-classifier-training

## Summary

Trains a supervised Keras/TensorFlow artificial neural network (via CASi's CrossTimeAnnotation) to assign cell labels to a later timepoint (t1) by borrowing information from annotated cells at an initial timepoint (t0) in multi-timepoint scRNA-seq experiments.

## When to use

Use when you have paired scRNA-seq count matrices from two timepoints (t0/t1), labels for the t0 cells, and need supervised cross-timepoint cell-type annotation of t1 cells.

## When NOT to use

- The initial timepoint's data is dramatically different from later timepoint's data — the neural network would fail to borrow useful information and could not be very well trained.
- The data is cross-sectional from a single time point with no t0/t1 pairing — use a standard supervised annotation method instead.
- You expect novel cell types that are transcriptomically similar to known cell types — annotation alone may misassign them; pair with DetectNovelCells and treat results cautiously.

## Inputs

- Gene expression matrix (counts) of the initial timepoint/t0
- Gene expression matrix (counts) of the following timepoint/t1
- Cell labels of the initial timepoint/t0 (optional but needed for supervised annotation)
- CASiList object built via CreateCASiList(t0_counts, t1_counts, t0_labels)

## Outputs

- Predicted cell-type labels for t1 cells
- Per-cell annotation accuracy per method per scenario (aggregated, e.g., annotation_benchmark_results.csv)
- Adjusted Rand index (ARI) comparisons across scenarios (as in supplementary Figure 11)

## How to apply

Assemble the t0 and t1 gene expression matrices together with t0 cell labels using CASi's CreateCASiList, which produces the CASiList input object. Then call CrossTimeAnnotation, which trains a Keras/TensorFlow neural network classifier (TensorFlow must be installed per tensorflow.rstudio.com instructions and the 'keras' package loaded) to assign labels to t1 cells in a supervised way, borrowing information from the earlier timepoint. The rationale is that cross-timepoint samples from the same subject are correlated, so t0 labels are informative for annotating t1; benchmark this against supervised competitors (scmap, CHETAH, scPred) on identical t0/t1 splits, reporting per-cell annotation accuracy and adjusted Rand index (ARI). Repeating the simulation across 200 Monte Carlo repetitions and aggregating results (e.g., to annotation_benchmark_results.csv) verifies robustness. If new cell types may have emerged at t1, follow up with DetectNovelCells, since ignoring novel cell types leads to inaccurate annotation.

## Related tools

- **CASi** (Provides CreateCASiList to build the t0/t1 input object and CrossTimeAnnotation to train and apply the neural network classifier for cross-timepoint cell labels) — github.com/yizhuo-wang/CASi
- **keras** (R Keras/TensorFlow dependency required by CrossTimeAnnotation to train and run the artificial neural network)

## Examples

```
require(keras); require(CASi); cas <- CreateCASiList(t0_counts, t1_counts, t0_labels); annot <- CrossTimeAnnotation(cas)
```

## Evaluation signals

- In benchmark scenarios 1 and 2 (shifted compositions, disappearing cell type), CASi should achieve the highest accuracy and ARI versus scmap, CHETAH, and scPred over 200 Monte Carlo repetitions.
- In scenario 3 (novel emerging cell type), accuracy and ARI should rise significantly only when the novel-cell detection pipeline is combined with annotation — if novel cells are ignored, annotation degrades.
- Trained model must converge under a working TensorFlow setup (keras package available) — installation failures indicate environment misconfiguration.
- Output table (annotation_benchmark_results.csv) must contain per-replicate, per-scenario, per-method accuracy and ARI values rather than a single averaged score.
- ARI comparisons should reproduce the ranking pattern reported in supplementary Figure 11.

## Limitations

- If the initial timepoint's data is dramatically different from later timepoint's data, the neural network fails to borrow useful information and cannot be trained well.
- The annotation step pools the data from all samples together for training, which may be inappropriate when cell populations differ across conditions.
- No code or software availability statement / version numbers for all tools are provided in the article text; ARI results for the three scenarios are only in supplementary Figure 11.

## Evidence

- [intro] Our framework implements a neural network classifier to assign cell labels in a supervised way, and it can achieve high accuracy in the cross-timepoint setting.: "Our framework implements a neural network classifier to assign cell labels in a supervised way, and it can achieve high accuracy in the cross-timepoint setting."
- [methods] Below are the input parameters of the CrossTimeAnnotation function: "Below are the input parameters of the CrossTimeAnnotation function"
- [readme] Before using CASi, please make sure to set up TensorFlow from https://tensorflow.rstudio.com/install/, and that the dependent package "keras" is available to use.: [evidence span withheld — anchor status: external_doc]
- [methods] The demo pbmc data, "pbmc_demo", has been lazy-loaded with the package.: "The demo pbmc data, "pbmc_demo", has been lazy-loaded with the package."
- [discussion] If the initial timepoint’s data is dramatically different from later timepoint’s data, the neural network would fail to borrow useful information and could not be very well trained: "If the initial timepoint’s data is dramatically different from later timepoint’s data, the neural network would fail to borrow useful information and could not be very well trained"
