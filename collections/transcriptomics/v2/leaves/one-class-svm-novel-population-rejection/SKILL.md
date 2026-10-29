---
name: one-class-svm-novel-population-rejection
description: Use when constructing a scHPL hierarchical classification tree on annotated
  single-cell data where novel/unseen cell populations must be detected (rejection),
  or when reference batches may lack populations present in other batches.
when_to_use_negative:
- Datasets are not aligned/harmonized first — scHPL is not a batch correction tool,
  so population matching (and hence one-class rejection) will be confounded by batch
  effects
- Maximum classification accuracy on fully known populations is the goal — the linear
  SVM matched or exceeded the one-class SVM on all datasets, and its performance drop
  on small populations was less severe
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scHPL
  role: Framework that trains a one-class SVM at each node of the hierarchical classification
    tree and implements rejection of unseen populations via its train/predict/evaluate
    modules
  repo: https://github.com/lcmmichielsen/scHPL
- name: scArches
  role: Used (with scVI) to align datasets before scHPL learns/matches populations,
    a prerequisite for meaningful one-class SVM rejection
  repo: https://github.com/theislab/scarches
provenance:
  source_task_ids:
  - task_003
  - task_001
  source_papers:
  - doi: 10.1038/s41467-021-23196-8
    title: Hierarchical progressive learning of cell identities in single-cell data
  source_content_hash: sha256:334fca53c26e9fc920670f3ef00fc1696b965adce099334cf9a12acc6186d8f4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lcmmichielsen/scHPL
  tools:
  - scArches
  - scHPL
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# one-class-svm-novel-population-rejection

## Summary

A classifier choice in scHPL's hierarchical tree where a one-class SVM is trained per tree node instead of a linear SVM, enabling explicit rejection of cells as unseen (sub)populations and more stable tree construction when batch populations are missing.

## When to use

Use when constructing a scHPL hierarchical classification tree on annotated single-cell data where novel/unseen cell populations must be detected (rejection), or when reference batches may lack populations present in other batches.

## When NOT to use

- Datasets are not aligned/harmonized first — scHPL is not a batch correction tool, so population matching (and hence one-class rejection) will be confounded by batch effects
- Maximum classification accuracy on fully known populations is the goal — the linear SVM matched or exceeded the one-class SVM on all datasets, and its performance drop on small populations was less severe

## Inputs

- Aligned, annotated scRNA-seq datasets as AnnData objects with cell labels and batch indicators as metadata columns and aligned expression in .X
- A predefined or progressively learned hierarchical classification tree

## Outputs

- Hierarchical classification tree with one-class SVM trained at each node
- Predicted labels plus a set of cells rejected as unseen populations per fold
- Per-fold HF1-scores and rejection percentages (aggregated in hf1_rejection_results.csv)
- Tree-stability counts vs. the expected reference hierarchy (tree_stability_results.csv)

## How to apply

Train scHPL with classifier='one-class' so that a one-class SVM is fitted at each node of the hierarchical tree (alternative options are linear SVM or kNN per node). During prediction on held-out folds, record cells rejected at nodes as candidate unseen populations rather than forcing a label. Evaluate with scHPL's evaluate module computing HF1-scores and rejection percentages. Expect a trade-off: the linear SVM achieves higher classification accuracy (e.g. median HF1 0.94 vs 0.80 on AMB), but the one-class SVM is more robust to missing populations — in the simulated-data experiment where Group5 was withheld from Batch 2, the one-class SVM reconstructed the expected tree in two-thirds of the 60 runs while the linear SVM mis-constructed all 60. Datasets must be aligned (e.g. with scVI/scArches) before training, since scHPL is not a batch-correction tool.

## Related tools

- **scHPL** (Framework that trains a one-class SVM at each node of the hierarchical classification tree and implements rejection of unseen populations via its train/predict/evaluate modules) — https://github.com/lcmmichielsen/scHPL
- **scArches** (Used (with scVI) to align datasets before scHPL learns/matches populations, a prerequisite for meaningful one-class SVM rejection) — https://github.com/theislab/scarches

## Examples

```
pip install scHPL
```

## Evaluation signals

- In the Group5-withholding experiment on simulated data, the one-class SVM tree should match the expected hierarchy in ~2/3 of the 60 runs (vs. 0/60 for the linear SVM)
- Median HF1-scores and rejection percentages from scHPL's evaluate module should match the paper's results tables for each dataset/classifier combination
- One-class SVM accuracy should be equal to or below the linear SVM across datasets, with a comparatively larger performance drop on populations with few cells
- Rejected cells should correspond to populations absent from training batches, not to mislabels within known populations (as verified by marker-gene expression in the PBMC experiment)

## Limitations

- One-class SVM classification accuracy is lower than the linear SVM on all tested datasets (simulated, PBMC-FACS, AMB)
- Performance on populations with a small number of cells decreases more for the one-class SVM than for the linear SVM
- scHPL assumes tree-representable relationships; the one-class rejection cannot rescue situations where a hierarchy itself is inadequate (e.g. dedifferentiation such as beta-to-alpha cell conversion)
- Batch effects can make it hard to tell whether tree mistakes come from erroneous alignment or mismatches created by scHPL

## Evidence

- [methods] one-class classifier trained per node: "For each node in the tree either a linear SVM, kNN, or one-class SVM is trained."
- [intro] rejection rationale: "(iii) we adopt an advanced rejection procedure including a one-class classifier to be able to discover new cell (sub)populations"
- [discussion] small-population caveat: "For both classifiers, we saw a decrease in performance on populations with a small number of cells, but for the one-class SVM this effect was more apparent."
- [readme] detection of unknown populations (README): "For each node in the tree either a linear SVM, kNN, or one-class SVM, which enables the detection of unknown populations, is trained."
- [readme] alignment prerequisite (README): "NOTE: scHPL is not a batch correction tool, we advise to align the datasets before matching the cell populations."
