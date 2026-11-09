---
name: cross-classifier-rejection-comparison
description: 'Use when evaluating which classifier variant better rejects cells from
  unseen cell populations: you have a labeled hierarchical dataset, can hold out one
  population (e.g. Group 3 / four AMB nodes), and need per-variant rejection rates.'
when_to_use_negative:
- Your goal is accurate classification of known populations rather than rejection
  of unseen ones — the linear SVM had accuracy equal to or higher than the one-class
  SVM on all datasets, and the one-class SVM degrades more on small populations
- The datasets are not yet aligned — scHPL is not a batch correction tool, so alignment
  (scVI or scArches) must be done before matching cell populations
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scHPL
  role: Trains the hierarchical classification tree with linear SVM or one-class SVM
    nodes and predicts/rejects labels on held-out cells (learn, train, predict, evaluate
    modules)
  repo: https://github.com/lcmmichielsen/scHPL
- name: scArches
  role: Recommended alignment of datasets before matching cell populations, since
    scHPL requires pre-aligned data
  repo: https://github.com/theislab/scarches
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-021-23196-8
    title: Hierarchical progressive learning of cell identities in single-cell data
  source_content_hash: sha256:c6a4cb1a0819a7c92447f19e5a8d67a4be70165a0818251f8b6967bc9e666534
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lcmmichielsen/scHPL
  tools:
  - scArches
  - scHPL
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# cross-classifier-rejection-comparison

## Summary

A benchmarking procedure that trains the same hierarchical classification tree with two scHPL classifier variants (linear SVM vs one-class SVM) and compares their ability to reject cells from unseen populations, e.g. the simulated dataset with 'Group 3' held out.

## When to use

Use when evaluating which classifier variant better rejects cells from unseen cell populations: you have a labeled hierarchical dataset, can hold out one population (e.g. Group 3 / four AMB nodes), and need per-variant rejection rates.

## When NOT to use

- Your goal is accurate classification of known populations rather than rejection of unseen ones — the linear SVM had accuracy equal to or higher than the one-class SVM on all datasets, and the one-class SVM degrades more on small populations
- The datasets are not yet aligned — scHPL is not a batch correction tool, so alignment (scVI or scArches) must be done before matching cell populations

## Inputs

- Simulated single-cell dataset with cell-population annotations (downloadable from the scHPL Zenodo record, doi:10.5281/zenodo.4557712)
- Hierarchical tree of cell populations constructed by scHPL's learn function on the training populations
- Held-out unseen population test set (e.g. Group 3: four AMB nodes at tree layers 2 and 3)

## Outputs

- Per-classifier rejection rates for the held-out population (fraction of cells rejected as unseen)
- Comparison table of rejection rates (expected: 100% one-class SVM vs 38.9% linear SVM on Group 3)
- Fig. 3E-style heatmap of per-node rejection fractions covering all four held-out AMB nodes at layers 2 and 3

## How to apply

Load the simulated single-cell dataset with annotations and remove the 'Group 3' population (four AMB nodes at tree layers 2 and 3), keeping it aside as the unseen-population test set. Construct the hierarchical classification tree on the remaining populations with scHPL's learn function; align datasets beforehand (e.g. with scVI or scArches) since scHPL is not a batch correction tool. Train the hierarchical classifier twice with scHPL's train function — once with linear SVM nodes and once with one-class SVM nodes at each node of the tree. Apply scHPL's predict function to classify the held-out Group 3 cells under both trees and record the fraction of cells rejected as unseen, compiling rejection rates with scHPL's evaluate module and rendering per-node rejection fractions as a Fig. 3E-style heatmap.

## Related tools

- **scHPL** (Trains the hierarchical classification tree with linear SVM or one-class SVM nodes and predicts/rejects labels on held-out cells (learn, train, predict, evaluate modules)) — https://github.com/lcmmichielsen/scHPL
- **scArches** (Recommended alignment of datasets before matching cell populations, since scHPL requires pre-aligned data) — https://github.com/theislab/scarches

## Examples

```
from scHPL import train, predict; classifier_linear = train(tree, data_train, classifier='linear_svm'); classifier_oc = train(tree, data_train, classifier='oneclass_svm'); predictions = predict(classifier_oc, data_group3)
```

## Evaluation signals

- One-class SVM rejects 100% of held-out Group 3 cells; linear SVM rejects 38.9% — computed rates must match these reported values
- The per-node rejection heatmap covers all four held-out AMB nodes at tree layers 2 and 3
- Both classifier variants are trained on an identical tree structure (same learn output) so rejection rates are directly comparable
- Predicted fractions are true fractions of cells rejected, verified via scHPL's evaluate module

## Limitations

- The one-class SVM's performance decreases more than the linear SVM's on populations with a small number of cells
- scHPL is not a batch correction tool; datasets must be aligned before running it, and adding a new dataset requires rerunning the alignment and consequently scHPL
- The hierarchical tree assumption can be inadequate in situations such as dedifferentiation (e.g. beta to alpha cell conversions in type 2 diabetes)

## Evidence

- [methods] For each node in the tree either a linear SVM, kNN, or one-class SVM is trained.: "For each node in the tree either a linear SVM, kNN, or one-class SVM is trained."
- [readme] one-class SVM which enables the detection of unknown populations: "For each node in the tree either a linear SVM, kNN, or one-class SVM, which enables the detection of unknown populations, is trained."
- [intro] we adopt an advanced rejection procedure including a one-class classifier to be able to discover new cell (sub)populations: "(iii) we adopt an advanced rejection procedure including a one-class classifier to be able to discover new cell (sub)populations"
- [discussion] for the one-class SVM this effect was more apparent on small populations: "For both classifiers, we saw a decrease in performance on populations with a small number of cells, but for the one-class SVM this effect was more apparent."
- [discussion] we require datasets to be aligned before running scHPL: "Since batch effects are inevitable when combining datasets, we require datasets to be aligned before running scHPL"
