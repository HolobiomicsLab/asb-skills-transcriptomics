---
name: hierarchical-classifier-training
description: Use when an annotated scRNA-seq dataset (e.g., simulated AMB data or
  PBMC-FACS/AMB2018 from the scRNA-seq benchmark Zenodo) must be classified through
  a hierarchical tree and cells from held-out or unknown populations must be flagged
  as 'unseen' rather than misassigned.
when_to_use_negative:
- 'Datasets are not yet aligned: scHPL is not a batch correction tool, so alignment
  (scVI/scArches) must be run first rather than expecting scHPL to handle batch effects.'
- The goal is only annotating known populations at maximum accuracy — the linear SVM
  matches or exceeds one-class SVM accuracy on known populations, so the one-class
  SVM's rejection behavior is not needed.
- A flat (non-hierarchical) label space is sufficient and no detection of new/unseen
  cell populations is required.
edam_operation: http://edamontology.org/operation_2995
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scHPL
  role: 'Core tool: learn function builds the hierarchical tree, train function fits
    a linear SVM, kNN, or one-class SVM per node, predict classifies new cells and
    rejects unseen ones, evaluate compiles rejection rates'
  repo: https://github.com/lcmmichielsen/scHPL
- name: scArches
  role: Recommended pre-alignment of datasets before scHPL matches cell populations
    and trains the tree (treeArches framework)
  repo: https://github.com/theislab/scarches
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-021-23196-8
    title: Hierarchical progressive learning of cell identities in single-cell data
  source_content_hash: sha256:941c624d180aabed06348f7b457d00edcaab3378a65df5c91aa942c5a400da4f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lcmmichielsen/scHPL
  tools:
  - scArches
  - scHPL
  edam_operation: http://edamontology.org/operation_2995
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# hierarchical-classifier-training

## Summary

Train scHPL's hierarchical classification tree on annotated single-cell data, fitting either a linear SVM, kNN, or one-class SVM at each tree node so that known cell populations are classified while unseen populations are rejected. The one-class SVM variant is specifically adopted to discover new cell (sub)populations missing from the training data.

## When to use

Use when an annotated scRNA-seq dataset (e.g., simulated AMB data or PBMC-FACS/AMB2018 from the scRNA-seq benchmark Zenodo) must be classified through a hierarchical tree and cells from held-out or unknown populations must be flagged as 'unseen' rather than misassigned.

## When NOT to use

- Datasets are not yet aligned: scHPL is not a batch correction tool, so alignment (scVI/scArches) must be run first rather than expecting scHPL to handle batch effects.
- The goal is only annotating known populations at maximum accuracy — the linear SVM matches or exceeds one-class SVM accuracy on known populations, so the one-class SVM's rejection behavior is not needed.
- A flat (non-hierarchical) label space is sufficient and no detection of new/unseen cell populations is required.

## Inputs

- Annotated scRNA-seq dataset (e.g., simulated dataset with 'Group 3' population; filtered PBMC-FACS and AMB2018 from the scRNA-seq benchmark Zenodo, doi:10.5281/zenodo.3357167)
- Aligned low-dimensional embeddings of training datasets (datasets must be aligned beforehand, e.g., with scVI or scArches)
- Cell population annotations / hierarchical relationships across datasets
- Classifier choice per node: linear SVM, kNN, or one-class SVM

## Outputs

- Trained hierarchical classification tree with a fitted classifier at each node
- Predicted labels for new unlabeled cells, including rejections of unseen populations
- Per-node rejection-fraction table (e.g., one-class SVM 100% vs linear SVM 38.9% on held-out Group 3)
- Fig. 3E-style heatmap of per-node rejection fractions

## How to apply

First construct the tree with scHPL's learn function on the annotated training populations, after aligning the datasets (e.g., with scVI or scArches) since scHPL is not a batch correction tool. Then call scHPL's train function once per classifier variant — linear SVM nodes vs one-class SVM nodes — training one classifier at each node of the hierarchical tree. Apply scHPL's predict function to classify new unlabeled cells and record per-node rejection fractions; compile rejection rates with scHPL's evaluate module. Validate by comparing rejection of held-out populations: in the simulated experiment with 'Group 3' (four AMB nodes at tree layers 2 and 3) removed, the one-class SVM correctly rejected 100% of held-out cells while the linear SVM rejected only 38.9%. Note the article's counterbalancing result: linear SVM accuracy is equal to or higher than one-class SVM on known populations, and one-class SVM performance decreases more on small populations.

## Related tools

- **scHPL** (Core tool: learn function builds the hierarchical tree, train function fits a linear SVM, kNN, or one-class SVM per node, predict classifies new cells and rejects unseen ones, evaluate compiles rejection rates) — https://github.com/lcmmichielsen/scHPL
- **scArches** (Recommended pre-alignment of datasets before scHPL matches cell populations and trains the tree (treeArches framework)) — https://github.com/theislab/scarches

## Examples

```
pip install scHPL
```

## Evaluation signals

- Held-out population rejection: one-class SVM should reject 100% of held-out 'Group 3' cells while the linear SVM rejects ~38.9%, matching the reported Fig. 3E values.
- The rejection heatmap covers all four held-out AMB nodes at tree layers 2 and 3.
- Trained tree contains a fitted classifier (linear SVM, kNN, or one-class SVM) at every node, as specified in the methods.
- Known-population accuracy should meet or exceed one-class SVM accuracy when linear SVM nodes are used (linear SVM accuracy was equal to or higher on all datasets in the article).
- Datasets were aligned (scVI/scArches) before tree construction, per the article's stated prerequisite.

## Limitations

- One-class SVM performance decreases more than linear SVM on populations with a small number of cells.
- scHPL is not a batch correction tool; adding a new dataset requires rerunning the alignment and consequently scHPL.
- Batch effects make it hard to determine whether tree mistakes stem from erroneous alignment or mismatches created by scHPL.
- The hierarchical-relationship assumption is inherently limited, e.g., for dedifferentiation like beta-to-alpha cell conversions in type 2 diabetes.

## Evidence

- [methods] For each node in the tree either a linear SVM, kNN, or one-class SVM is trained.: "For each node in the tree either a linear SVM, kNN, or one-class SVM is trained."
- [results] one-class SVM rejects all unseen cells: [evidence span withheld — anchor status: not_found]
- [readme] README: per-node classifier choice and rejection capability: "For each node in the tree either a linear SVM, kNN, or one-class SVM, which enables the detection of unknown populations, is trained."
- [readme] README: alignment prerequisite: "NOTE: scHPL is not a batch correction tool, we advise to align the datasets before matching the cell populations."
- [discussion] discussion: small-population effect on one-class SVM: "For both classifiers, we saw a decrease in performance on populations with a small number of cells, but for the one-class SVM this effect was more apparent."
