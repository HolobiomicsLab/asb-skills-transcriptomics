---
name: rejection-fraction-heatmap-visualization
description: Use when benchmarking how a hierarchical classifier rejects cells from
  unseen populations held out of training, e.g. comparing one-class SVM vs linear
  SVM rejection fractions across tree nodes.
when_to_use_negative:
- Datasets are not yet aligned — since scHPL is not a batch correction tool, rejection
  fractions computed on unaligned data are not interpretable; align first with scVI
  or scArches
- All cell populations in the test set are present in the training data, so there
  are no unseen populations whose rejection behavior needs to be visualized
edam_operation: http://edamontology.org/operation_0337
tools:
- name: scHPL
  role: Provides the learn, train, predict, and evaluate functions used to build the
    tree, train linear/one-class SVM nodes, classify held-out cells, and compile per-node
    rejection rates for the heatmap
  repo: https://github.com/lcmmichielsen/scHPL
- name: scarches
  role: Recommended alignment tool (with scVI) to preprocess datasets before scHPL
    classification and rejection assessment
  repo: https://github.com/theislab/scarches
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-021-23196-8
    title: Hierarchical progressive learning of cell identities in single-cell data
  source_content_hash: sha256:1aa9144ec7432643fe1e306ec30f65e28c18f099f47e15920f36eb6c595265bf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lcmmichielsen/scHPL
  tools:
  - scHPL
  - scarches
  edam_operation: http://edamontology.org/operation_0337
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# rejection-fraction-heatmap-visualization

## Summary

Compiles per-node rejection rates from scHPL's hierarchical classifier into a table and renders a Fig. 3E-style heatmap comparing how the linear SVM and one-class SVM reject cells from unseen (held-out) cell populations such as 'Group 3'.

## When to use

Use when benchmarking how a hierarchical classifier rejects cells from unseen populations held out of training, e.g. comparing one-class SVM vs linear SVM rejection fractions across tree nodes.

## When NOT to use

- Datasets are not yet aligned — since scHPL is not a batch correction tool, rejection fractions computed on unaligned data are not interpretable; align first with scVI or scArches
- All cell populations in the test set are present in the training data, so there are no unseen populations whose rejection behavior needs to be visualized

## Inputs

- Simulated single-cell dataset with population annotations (e.g. AMB hierarchy with 'Group 3' held out)
- Trained scHPL hierarchical classification tree using linear SVM nodes
- Trained scHPL hierarchical classification tree using one-class SVM nodes
- Predicted labels / rejection flags for the held-out cells from scHPL's predict function

## Outputs

- Table of rejection rates per classifier and per tree node (expected: 100% one-class SVM vs 38.9% linear SVM on Group 3)
- Fig. 3E-style heatmap of per-node rejection fractions covering all four held-out AMB nodes at layers 2 and 3

## How to apply

Remove the held-out population (e.g. 'Group 3', corresponding to four AMB nodes at tree layers 2 and 3) from the training set, build the hierarchical tree with scHPL's learn function on the remaining populations, and train the tree twice with scHPL's train function — once with linear SVM nodes and once with one-class SVM nodes. Run scHPL's predict function on the held-out cells under both trained trees and record, per classifier and per node, the fraction of cells rejected as unseen. Use scHPL's evaluate module to compile these rejection rates into a table and render the per-node rejection fractions as a Fig. 3E-style heatmap. Because scHPL is not a batch correction tool, datasets must be aligned beforehand (e.g. with scVI or scArches) before any classification or rejection assessment.

## Related tools

- **scHPL** (Provides the learn, train, predict, and evaluate functions used to build the tree, train linear/one-class SVM nodes, classify held-out cells, and compile per-node rejection rates for the heatmap) — https://github.com/lcmmichielsen/scHPL
- **scarches** (Recommended alignment tool (with scVI) to preprocess datasets before scHPL classification and rejection assessment) — https://github.com/theislab/scarches

## Examples

```
pip install scHPL  # then use scHPL.train/predict on the linear-SVM and one-class-SVM trees and scHPL.evaluate to compile the rejection-rate table for the Fig. 3E heatmap
```

## Evaluation signals

- One-class SVM rejects 100% of held-out Group 3 cells while the linear SVM rejects only 38.9%
- Heatmap covers all four held-out AMB nodes located at tree layers 2 and 3
- Every per-node rejection fraction lies in [0, 1] and is reported for both classifier variants on the identical trained tree structure
- Rejection table is compiled via scHPL's evaluate module rather than ad-hoc counting, ensuring per-node fractions are attributable to the correct tree node

## Limitations

- One-class SVM performance decreases more than the linear SVM on populations with a small number of cells, which can distort rejection fractions for rare held-out populations
- scHPL is not a batch correction tool; misalignment can confound whether rejections reflect truly unseen populations or batch effects
- The reported rejection rates (100% vs 38.9%) are specific to the simulated data with Group 3 removed and may not transfer to other datasets

## Evidence

- [full_text] One-class SVM detects unseen populations better than linear SVM: [evidence span withheld — anchor status: not_found]
- [methods] Node-level classifiers in scHPL: "For each node in the tree either a linear SVM, kNN, or one-class SVM is trained."
- [discussion] Alignment prerequisite: "Since batch effects are inevitable when combining datasets, we require datasets to be aligned before running scHPL"
- [discussion] One-class SVM degrades more on small populations: "For both classifiers, we saw a decrease in performance on populations with a small number of cells, but for the one-class SVM this effect was more apparent."
- [readme] Rejection capability of one-class SVM nodes: "For each node in the tree either a linear SVM, kNN, or one-class SVM, which enables the detection of unknown populations, is trained."
