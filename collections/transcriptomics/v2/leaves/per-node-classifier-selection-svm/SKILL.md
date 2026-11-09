---
name: per-node-classifier-selection-svm
description: Use when constructing an scHPL hierarchical tree over aligned single-cell
  datasets and you must decide which classifier (linear SVM, kNN, or one-class SVM)
  to train per node, especially when unknown/new cell populations must be detectable
  via rejection.
when_to_use_negative:
- The datasets have not been aligned/batch-corrected beforehand — scHPL is not a batch
  correction tool, so tree mistakes could stem from erroneous alignment rather than
  classifier choice
- The goal is plain label prediction of a fully annotated reference without any need
  to detect unknown populations — the linear SVM already performs equal or better
  and is simpler
- Dedifferentiation or non-tree-like transitions (e.g. beta-to-alpha cell conversion
  in type 2 diabetes) are expected, where the hierarchical-relationship assumption
  itself is inadequate
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scHPL
  role: Implements the hierarchical progressive learning tree whose nodes are trained
    with linear SVM, kNN, or one-class SVM; scHPL.learn constructs trees and trains
    per-node classifiers
  repo: https://github.com/lcmmichielsen/scHPL
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-021-23196-8
    title: Hierarchical progressive learning of cell identities in single-cell data
  source_content_hash: sha256:0ebe423037bd0f8298b21bde062d3926e1399ff6da4c78b085c73e2b13178cad
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lcmmichielsen/scHPL
  tools:
  - scHPL
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# per-node-classifier-selection-svm

## Summary

Select and train the classifier at each node of an scHPL hierarchical classification tree — a linear SVM for labeled populations versus a one-class SVM whose rejection-based boundary enables detection of unknown cell (sub)populations — and validate the choice via tree-stability across batch orders and folds.

## When to use

Use when constructing an scHPL hierarchical tree over aligned single-cell datasets and you must decide which classifier (linear SVM, kNN, or one-class SVM) to train per node, especially when unknown/new cell populations must be detectable via rejection.

## When NOT to use

- The datasets have not been aligned/batch-corrected beforehand — scHPL is not a batch correction tool, so tree mistakes could stem from erroneous alignment rather than classifier choice
- The goal is plain label prediction of a fully annotated reference without any need to detect unknown populations — the linear SVM already performs equal or better and is simpler
- Dedifferentiation or non-tree-like transitions (e.g. beta-to-alpha cell conversion in type 2 diabetes) are expected, where the hierarchical-relationship assumption itself is inadequate

## Inputs

- AnnData objects (aligned expression matrix in .X, cell labels and batch indicators as metadata columns) for simulated and PBMC-FACS datasets
- An scHPL hierarchical classification tree with per-node classifiers to train
- Batch orderings and cross-validation folds (6 batch orders x 10 folds = 60 runs per condition)

## Outputs

- Per-node trained classifiers (linear SVM or one-class SVM) within the classification tree
- Tree stability counts vs the expected reference hierarchy (e.g. 58/60 consistent PBMC trees, 2/60 divergent)
- tree_stability_results.csv with classifier-comparison outcomes for the missing-Group5 experiment

## How to apply

After scHPL.learn has constructed the hierarchical classification tree from the six batch orders x 10 folds (60 trees per dataset condition), train a classifier at each node of the tree, choosing between a linear SVM and a one-class SVM. The linear SVM achieves equal or higher accuracy on labeled populations, but when a batch is missing a population entirely (e.g. Group5 withheld from Batch 2 of the simulated data) the linear SVM mis-constructed all 60 learned trees, whereas the one-class SVM correctly learned the expected tree in two-thirds of the runs (succeeding in 2 of 3 runs). Thus prefer the one-class SVM when progressive discovery of unseen populations or robustness to missing populations is required; accept its known accuracy drop on small populations. Compare each learned tree against the expected reference hierarchy (e.g. 2/60 divergent PBMC trees, 58/60 consistent) and tally divergences to tree_stability_results.csv to validate the classifier choice.

## Related tools

- **scHPL** (Implements the hierarchical progressive learning tree whose nodes are trained with linear SVM, kNN, or one-class SVM; scHPL.learn constructs trees and trains per-node classifiers) — https://github.com/lcmmichielsen/scHPL

## Examples

```
pip install scHPL; import scHPL; tree = scHPL.learn(adata_list, classifier='oneclasssvm')
```

## Evaluation signals

- With Group5 completely removed from Batch 2 of the simulated data, the linear SVM must mis-construct all 60 learned trees while the one-class SVM correctly learns the expected tree in 2/3 of runs
- On PBMC-FACS, tally divergent trees vs the expected hierarchy: 2/60 divergent (58/60 consistent) is the reported baseline
- AnnData input check: expression matrix copied to .X with cell labels and batch indicators present as metadata columns before calling scHPL.learn
- Run the full sweep of 6 batch orders x 10 folds = 60 trees per dataset condition per classifier; missing-count discrepancies indicate a broken run
- One-class SVM performance should show a stronger decrease than the linear SVM on populations with a small number of cells — a sanity check on expected relative behavior

## Limitations

- Datasets must be aligned before running scHPL; with batch effects it is hard to resolve whether tree mistakes stem from erroneous alignment or mismatches created by scHPL
- One-class SVM performance decreases more than the linear SVM on populations with a small number of cells
- When a new dataset is added, the alignment — and consequently scHPL — has to be rerun
- A tree is inadequate in situations such as dedifferentiation (e.g. beta to alpha cell conversions in type 2 diabetes)

## Evidence

- [methods] per-node classifier options: "For each node in the tree either a linear SVM, kNN, or one-class SVM is trained."
- [abstract] one-class SVM enables novelty detection: "For each node in the tree either a linear SVM, kNN, or one-class SVM, which enables the detection of unknown populations, is trained."
- [readme] alignment prerequisite: "NOTE: scHPL is not a batch correction tool, we advise to align the datasets before matching the cell populations."
- [discussion] small-population accuracy caveat: "For both classifiers, we saw a decrease in performance on populations with a small number of cells, but for the one-class SVM this effect was more apparent."
- [readme] installation: "The easiest way to install scHPL is through the following command::

    pip install scHPL"
