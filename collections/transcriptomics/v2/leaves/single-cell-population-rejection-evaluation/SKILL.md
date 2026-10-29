---
name: single-cell-population-rejection-evaluation
description: 'Use when you need to test whether a hierarchical single-cell classifier
  detects truly novel populations: hold out a known population (e.g. ''Group 3'' with
  four AMB nodes at tree layers 2 and 3) and measure the fraction of its cells rejected
  as unseen under alternative node classifiers.'
when_to_use_negative:
- 'Datasets are not aligned: scHPL is not a batch correction tool, so running this
  rejection evaluation without prior alignment (e.g. scVI or scArches) is inappropriate'
- The goal is classification accuracy on populations present in training rather than
  rejection of unseen populations — for accuracy comparisons use the classifiers'
  standard performance metrics instead
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scHPL
  role: 'Hierarchical progressive learning method: learn builds the classification
    tree, train fits linear SVM or one-class SVM at each node, predict labels new
    cells with rejection, evaluate compiles rejection rates'
  repo: https://github.com/lcmmichielsen/scHPL
- name: scArches
  role: Recommended alignment of datasets before running scHPL, since scHPL is not
    a batch correction tool
  repo: https://github.com/theislab/scarches
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-021-23196-8
    title: Hierarchical progressive learning of cell identities in single-cell data
  source_content_hash: sha256:0377f08ff8b7ea0dfe9e4dc0a1311b79487f1697c57fb8739cfb58cd6930f6cc
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
yes_rate: 0.625
n_claims: 8
---

# single-cell-population-rejection-evaluation

## Summary

Benchmark how well classifier variants within scHPL (linear SVM vs one-class SVM) reject cells from unseen cell populations held out from training, quantifying rejection rates and rendering per-node rejection fractions as a Fig. 3E-style heatmap.

## When to use

Use when you need to test whether a hierarchical single-cell classifier detects truly novel populations: hold out a known population (e.g. 'Group 3' with four AMB nodes at tree layers 2 and 3) and measure the fraction of its cells rejected as unseen under alternative node classifiers.

## When NOT to use

- Datasets are not aligned: scHPL is not a batch correction tool, so running this rejection evaluation without prior alignment (e.g. scVI or scArches) is inappropriate
- The goal is classification accuracy on populations present in training rather than rejection of unseen populations — for accuracy comparisons use the classifiers' standard performance metrics instead

## Inputs

- Simulated single-cell dataset with cell-population annotations (downloadable from scHPL Zenodo, doi 10.5281/zenodo.4557712)
- Population annotation labels identifying the held-out group (e.g. 'Group 3') and tree node membership (AMB nodes at layers 2 and 3)

## Outputs

- Rejection-rate table per classifier (one-class SVM vs linear SVM) for the held-out population
- Fig. 3E-style heatmap of per-node rejection fractions across the held-out AMB nodes

## How to apply

Load the simulated single-cell dataset with annotations and remove the 'Group 3' population (four AMB nodes at tree layers 2 and 3) from training, keeping it as the unseen-population test set. Build the hierarchical classification tree on the remaining populations with scHPL's learn function, after aligning datasets (e.g. with scVI or scArches) since scHPL is not a batch correction tool. Train the tree twice with scHPL's train function — once with linear SVM nodes and once with one-class SVM nodes — then apply scHPL's predict function to the held-out Group 3 cells and record the rejected fraction per classifier. Use scHPL's evaluate module to compile rejection rates into a table and render per-node rejection fractions as a Fig. 3E-style heatmap covering all four held-out AMB nodes.

## Related tools

- **scHPL** (Hierarchical progressive learning method: learn builds the classification tree, train fits linear SVM or one-class SVM at each node, predict labels new cells with rejection, evaluate compiles rejection rates) — https://github.com/lcmmichielsen/scHPL
- **scArches** (Recommended alignment of datasets before running scHPL, since scHPL is not a batch correction tool) — https://github.com/theislab/scarches

## Examples

```
import scHPL; tree = scHPL.learn(data_aligned, labels_train); tree = scHPL.train(tree, classifier='one-class SVM'); predicted = scHPL.predict(tree, X_group3); scHPL.evaluate.rejection_rates(predicted)
```

## Evaluation signals

- One-class SVM rejects 100% of held-out Group 3 cells; linear SVM rejects 38.9%
- Heatmap covers all four held-out AMB nodes at tree layers 2 and 3
- Rejection rates compiled with scHPL's evaluate module match the reported values for Group 3
- Both trained trees contain the same node structure (learned on the same training set with Group 3 removed), differing only in node classifier type

## Limitations

- One-class SVM performance decreases more on small populations; linear SVM accuracy was equal to or higher than one-class SVM on all datasets
- Datasets must be aligned before running scHPL since it is not a batch correction tool; adding a new dataset requires rerunning the alignment and consequently scHPL
- The hierarchical-relationship assumption has inherent limitations, e.g. dedifferentiation like beta-to-alpha cell conversions in type 2 diabetes

## Evidence

- [other] one-class SVM detects unseen populations better: [evidence span withheld — anchor status: not_found]
- [methods] scHPL trains different node classifiers: "For each node in the tree either a linear SVM, kNN, or one-class SVM is trained."
- [discussion] alignment requirement: "Since batch effects are inevitable when combining datasets, we require datasets to be aligned before running scHPL"
- [readme] README installation and usage: "The easiest way to install scHPL is through the following command::

    pip install scHPL"
- [readme] one-class SVM purpose: "or one-class SVM, which enables the detection of unknown populations, is trained"
