---
name: progressive-learning-cell-annotation-across-batches
description: Use when multiple annotated single-cell datasets (e.g., simulated batches
  or PBMC-FACS) must be merged into one hierarchical classifier, especially when some
  batches may lack cell populations and you need to assess whether the learned tree
  stays stable.
when_to_use_negative:
- Datasets have not been aligned/batch-corrected first — scHPL is explicitly 'not
  a batch correction tool', so mistakes may stem from erroneous alignment rather than
  mismatches created by scHPL
- Cell populations violate the tree assumption, e.g. dedifferentiation such as beta-to-alpha
  cell conversions in type 2 diabetes, where a hierarchical tree is not adequate
- The goal is annotating a single pre-labeled reference against one query dataset
  with marker genes (methods like SCINA or Garnett assume marker-gene priors and are
  a different paradigm)
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scHPL
  role: 'Core method: runs progressive learning (scHPL.learn) to match populations
    across batches, construct the hierarchical tree, and train per-node linear SVM
    / one-class SVM classifiers'
  repo: https://github.com/lcmmichielsen/scHPL
- name: scArches
  role: Recommended upstream alignment of datasets before scHPL matches cell populations
    (also used in the treeArches framework around scHPL)
  repo: https://github.com/theislab/scarches
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-021-23196-8
    title: Hierarchical progressive learning of cell identities in single-cell data
  source_content_hash: sha256:ccf2d055f083f8411225b247554a13575688b0f1fcf3784a94b4de5426d2d162
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lcmmichielsen/scHPL
  tools:
  - scArches
  - scHPL
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# progressive-learning-cell-annotation-across-batches

## Summary

Progressively learn a hierarchical classification tree of cell identities across multiple annotated batches/datasets using scHPL, matching cell populations across batches and training a per-node classifier (linear SVM or one-class SVM) that can also flag unknown populations. Useful for combining datasets annotated at different resolutions and testing how batch composition affects tree stability.

## When to use

Use when multiple annotated single-cell datasets (e.g., simulated batches or PBMC-FACS) must be merged into one hierarchical classifier, especially when some batches may lack cell populations and you need to assess whether the learned tree stays stable.

## When NOT to use

- Datasets have not been aligned/batch-corrected first — scHPL is explicitly 'not a batch correction tool', so mistakes may stem from erroneous alignment rather than mismatches created by scHPL
- Cell populations violate the tree assumption, e.g. dedifferentiation such as beta-to-alpha cell conversions in type 2 diabetes, where a hierarchical tree is not adequate
- The goal is annotating a single pre-labeled reference against one query dataset with marker genes (methods like SCINA or Garnett assume marker-gene priors and are a different paradigm)

## Inputs

- Annotated scRNA-seq datasets (AnnData objects) with cell labels and batch indicator metadata columns and an aligned expression matrix in .X
- An expected reference hierarchy of cell populations for tree comparison
- Simulated multi-batch data with populations such as 'Group5' and the filtered PBMC-FACS dataset (Zenodo: doi 10.5281/zenodo.3357167 / scHPL Zenodo 10.5281/zenodo.4557712)

## Outputs

- Learned hierarchical classification trees (up to 60 per dataset condition across batch orders x folds)
- Tree stability counts vs. the expected hierarchy (e.g., divergent-tree tallies per classifier)
- tree_stability_results.csv with tree stability counts and linear SVM vs. one-class SVM comparison outcomes

## How to apply

Prepare each annotated dataset as an AnnData object with cell labels and batch indicators as metadata columns and the aligned expression matrix copied to .X, as required by scHPL.learn; scHPL requires datasets to be aligned first (e.g., with scVI or scArches) because it is not a batch-correction tool. Run the scHPL progressive learning procedure over batch orders and folds (e.g., six batch orders x 10 folds = 60 trees per condition) so the method automatically matches cell populations across batches and constructs the hierarchical tree. Train per-node classifiers, repeating with a linear SVM and a one-class SVM; the one-class SVM additionally enables detection of unknown populations. To test stability under missing populations, withhold a population (e.g., remove 'Group5' from Batch 2 of the simulated data), let scHPL construct the tree with each classifier, and count runs where the tree diverges from the expected reference hierarchy. Write tree stability counts and classifier comparison outcomes to a results table such as tree_stability_results.csv.

## Related tools

- **scHPL** (Core method: runs progressive learning (scHPL.learn) to match populations across batches, construct the hierarchical tree, and train per-node linear SVM / one-class SVM classifiers) — https://github.com/lcmmichielsen/scHPL
- **scArches** (Recommended upstream alignment of datasets before scHPL matches cell populations (also used in the treeArches framework around scHPL)) — https://github.com/theislab/scarches

## Examples

```
import scHPL; tree = scHPL.learn(adata_aligned, classifier='oneclass-svm')  # progressive learning over batches, per scHPL tutorial.ipynb
```

## Evaluation signals

- PBMC-FACS tree consistency: expect ~58/60 learned trees consistent with the expected reference hierarchy (i.e., only 2/60 divergent) across the six batch orders x 10 folds
- Missing-population contrast: with 'Group5' removed from Batch 2 of the simulated data, the linear SVM mis-constructs all 60 trees while the one-class SVM correctly learns the expected tree in roughly two-thirds of runs (2 of 3)
- Learned tree topology matches the known/expected hierarchy node-for-node (e.g., population matched as subpopulation of the correct parent, not a sibling branch)
- tree_stability_results.csv exists and its tallies reproduce the reported stability counts for both classifier settings
- Datasets were aligned before running scHPL (alignment step completed upstream), otherwise divergences are not attributable to the classifier

## Limitations

- Performance decreases on populations with a small number of cells, and this effect is more apparent for the one-class SVM
- Adding a new dataset requires rerunning the alignment and consequently scHPL with current batch correction tools
- It is hard to determine whether tree mistakes stem from erroneous dataset alignment or mismatches created by scHPL
- The hierarchical-relationship assumption is inherent — situations like dedifferentiation (beta-to-alpha cell conversions in type 2 diabetes) are not adequately represented by a tree
- Datasets must be aligned before running scHPL; many brain cell populations are missing from the Cell Ontology and annotation terminology is inconsistent across studies

## Evidence

- [methods] automatic cross-batch tree construction: "scHPL is a method that automatically finds relationships between cell populations across multiple datasets and uses this to construct a hierarchical classification tree."
- [methods] per-node classifiers: "For each node in the tree either a linear SVM, kNN, or one-class SVM is trained."
- [discussion] alignment requirement: "Since batch effects are inevitable when combining datasets, we require datasets to be aligned before running scHPL"
- [readme] not a batch correction tool (README): "NOTE: scHPL is not a batch correction tool, we advise to align the datasets before matching the cell populations."
- [discussion] small-population sensitivity: "For both classifiers, we saw a decrease in performance on populations with a small number of cells, but for the one-class SVM this effect was more apparent."
