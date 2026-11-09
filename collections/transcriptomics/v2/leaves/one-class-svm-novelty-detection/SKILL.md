---
name: one-class-svm-novelty-detection
description: Use when a hierarchical single-cell classifier must flag cells from cell
  populations that were excluded from training (novelty/unseen-population detection),
  e.g. testing whether held-out 'Group 3' AMB nodes are rejected.
when_to_use_negative:
- scHPL is not a batch correction tool — if datasets from different batches are not
  yet aligned (e.g. with scVI or scArches), align them first rather than feeding raw
  combined data
- The goal is accurate classification of known populations rather than rejection of
  unseen ones — the linear SVM matches or exceeds one-class SVM accuracy on known
  classes, and the one-class SVM degrades more on small populations
- No labeled population annotations exist to build the hierarchical tree in the first
  place
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scHPL
  role: Trains the hierarchical classification tree with one-class SVM or linear SVM
    nodes and predicts/rejects held-out cells (learn, train, predict, evaluate modules)
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
  source_content_hash: sha256:2ca31dd3b67835b45e98b46f4b6c0e0be08d69daa2abfe1e855b7b1f70016490
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
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# one-class-svm-novelty-detection

## Summary

Use scHPL's one-class SVM at each node of a hierarchical classification tree to reject cells from unseen cell populations that were held out from training, complementing the linear SVM used for classifying known populations.

## When to use

Use when a hierarchical single-cell classifier must flag cells from cell populations that were excluded from training (novelty/unseen-population detection), e.g. testing whether held-out 'Group 3' AMB nodes are rejected.

## When NOT to use

- scHPL is not a batch correction tool — if datasets from different batches are not yet aligned (e.g. with scVI or scArches), align them first rather than feeding raw combined data
- The goal is accurate classification of known populations rather than rejection of unseen ones — the linear SVM matches or exceeds one-class SVM accuracy on known classes, and the one-class SVM degrades more on small populations
- No labeled population annotations exist to build the hierarchical tree in the first place

## Inputs

- Annotated simulated single-cell dataset with population labels (downloadable from the scHPL Zenodo, doi:10.5281/zenodo.4557712)
- Cell population hierarchy / annotations for constructing the classification tree
- Held-out (unseen) test cells from the removed population, e.g. Group 3 AMB nodes at tree layers 2 and 3

## Outputs

- Trained scHPL classification trees (linear-SVM variant and one-class-SVM variant)
- Per-cell predicted labels and rejection flags for held-out Group 3 cells under both classifiers
- Table of rejection rates (expected: 100% one-class SVM vs 38.9% linear SVM) and a Fig. 3E-style per-node rejection-fraction heatmap

## How to apply

Remove the target population (e.g. the simulated 'Group 3', four AMB nodes at tree layers 2 and 3) from the training set and keep it as the unseen-population test set. Build the hierarchical tree with scHPL's learn function on the remaining populations, aligning datasets beforehand (e.g. with scVI or scArches) since scHPL is not a batch correction tool. Train the tree twice with scHPL's train function — once with linear SVM nodes and once with one-class SVM nodes — then apply predict to the held-out cells and record the fraction rejected as unseen per classifier. Compile rejection rates with scHPL's evaluate module; on Group 3 the one-class SVM should reject 100% of held-out cells versus only 38.9% for the linear SVM, demonstrating the one-class SVM's superior novelty detection.

## Related tools

- **scHPL** (Trains the hierarchical classification tree with one-class SVM or linear SVM nodes and predicts/rejects held-out cells (learn, train, predict, evaluate modules)) — https://github.com/lcmmichielsen/scHPL
- **scArches** (Recommended alignment of datasets before running scHPL, since scHPL is not a batch correction tool) — https://github.com/theislab/scarches

## Examples

```
from scHPL import learn, train, predict; tree = learn(annotations, data_aligned); classifier = train(tree, data_aligned, classifier='one-class SVM'); labels = predict(classifier, group3_heldout)
```

## Evaluation signals

- One-class SVM rejects 100% of held-out Group 3 cells; linear SVM rejects only 38.9% — computed rates must match these reported values
- The per-node rejection heatmap (Fig. 3E style) covers all four held-out AMB nodes at tree layers 2 and 3
- Both classifier variants are trained on the identical reduced training set (Group 3 removed) so rejection rates are directly comparable
- Datasets were aligned (scVI/scArches) before tree construction, as scHPL requires pre-aligned inputs
- For known populations, linear SVM accuracy should remain equal to or higher than one-class SVM, with the one-class SVM showing a larger performance drop on small populations

## Limitations

- The one-class SVM's classification performance decreases more than the linear SVM's on populations with a small number of cells
- When a new dataset is added, the alignment — and consequently scHPL — has to be rerun with current batch correction tools
- Batch effects make it hard to determine whether tree mistakes stem from erroneous alignment or mismatches created by scHPL
- Hierarchical relationships may not hold in all biological situations, e.g. dedifferentiation such as beta to alpha cell conversions in type 2 diabetes

## Evidence

- [readme] For each node in the tree either a linear SVM, kNN, or one-class SVM, which enables the detection of unknown populations, is trained.: "For each node in the tree either a linear SVM, kNN, or one-class SVM, which enables the detection of unknown populations, is trained."
- [intro] iii) adopt an advanced rejection procedure including a one-class classifier to discover new cell (sub)populations.: "(iii) we adopt an advanced rejection procedure including a one-class classifier to be able to discover new cell (sub)populations"
- [discussion] For both classifiers, we saw a decrease in performance on populations with a small number of cells, but for the one-class SVM this effect was more apparent.: "For both classifiers, we saw a decrease in performance on populations with a small number of cells, but for the one-class SVM this effect was more apparent."
- [discussion] Since batch effects are inevitable when combining datasets, we require datasets to be aligned before running scHPL.: "Since batch effects are inevitable when combining datasets, we require datasets to be aligned before running scHPL"
- [methods] The trained classification tree can be used to predict the labels of a new unlabeled dataset.: "The trained classification tree can be used to predict the labels of a new unlabeled dataset."
