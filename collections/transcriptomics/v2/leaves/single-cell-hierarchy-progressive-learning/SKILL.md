---
name: single-cell-hierarchy-progressive-learning
description: Use when two or more annotated scRNA-seq datasets must be merged into
  a single hierarchical cell-population tree, especially when annotations differ in
  resolution (e.g., AMB2016 vs AMB2018 visual cortex) and new populations may need
  to be discovered.
when_to_use_negative:
- Datasets are not batch-corrected/aligned — scHPL explicitly is not a batch correction
  tool and advises aligning datasets before matching populations
- Cell populations do not follow a tree-structured relationship, e.g. dedifferentiation
  states like beta-to-alpha cell conversions in type 2 diabetes
- You only need flat cell-type label transfer from a fixed atlas without hierarchy
  learning or discovery of new populations
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2640
tools:
- name: scHPL
  role: 'Core method: learns relationships between cell populations across datasets
    and constructs the hierarchical classification tree (scHPL.learn), trains per-node
    classifiers, and predicts labels for new datasets'
  repo: https://github.com/lcmmichielsen/scHPL
- name: Seurat
  role: Alignment of the AMB2016 and AMB2018 datasets before population matching (integrated
    expression matrix fed to scHPL)
- name: scikit-learn
  role: Provides the linear SVM trained at each node of the learned tree for integrated
    data retaining many dimensions
- name: scArches
  role: Recommended batch-correction/alignment framework (via treeArches) to align
    datasets before scHPL population matching
  repo: https://github.com/theislab/scarches
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-021-23196-8
    title: Hierarchical progressive learning of cell identities in single-cell data
  source_content_hash: sha256:03589c03cebf046e3ee4618a24011536a4f539076d1aecb5af1437f452bd87da
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lcmmichielsen/scHPL
  tools:
  - Seurat
  - scArches
  - scHPL
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2640
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# single-cell-hierarchy-progressive-learning

## Summary

Reconstruct and continuously update a hierarchical classification tree of cell populations across annotated scRNA-seq datasets using scHPL, which automatically matches populations across datasets, detects new subpopulations via one-class rejection, and trains a linear SVM, kNN, or one-class SVM at each node.

## When to use

Use when two or more annotated scRNA-seq datasets must be merged into a single hierarchical cell-population tree, especially when annotations differ in resolution (e.g., AMB2016 vs AMB2018 visual cortex) and new populations may need to be discovered.

## When NOT to use

- Datasets are not batch-corrected/aligned — scHPL explicitly is not a batch correction tool and advises aligning datasets before matching populations
- Cell populations do not follow a tree-structured relationship, e.g. dedifferentiation states like beta-to-alpha cell conversions in type 2 diabetes
- You only need flat cell-type label transfer from a fixed atlas without hierarchy learning or discovery of new populations

## Inputs

- Aligned AnnData objects (integrated expression matrix in .X, batch indicators and population labels in metadata) for each dataset, e.g., Seurat-aligned AMB2016 and AMB2018 visual cortex data
- Choice of classifier per node (linear SVM, kNN, or one-class SVM)
- Reference hierarchy for validation (e.g., Tasic et al. Extended Data Fig. 6 cluster correspondences)

## Outputs

- Hierarchical classification tree relating cell populations across datasets
- Trained per-node classifiers (linear SVM / kNN / one-class SVM) for label prediction on new unlabeled datasets
- Predicted labels for new unlabeled datasets, including rejection of unknown populations

## How to apply

First align the datasets (e.g., with Seurat or scVI/scArches), since scHPL is not a batch correction tool and requires aligned inputs; place the integrated expression matrix in the AnnData .X slot with batch indicators and population labels in metadata. Call scHPL.learn on the aligned datasets so it automatically matches cell populations across datasets and constructs the hierarchical classification tree, attaching unseen populations via its rejection/one-class mechanism. For each node in the learned tree, train a linear SVM, kNN, or one-class SVM — use the linear SVM for integrated data retaining many dimensions, as it matched or exceeded one-class SVM accuracy, whose performance drops more on small populations. Export and inspect the tree, then validate against a reference hierarchy, e.g. comparing the AMB2016–AMB2018 tree to Tasic et al. Extended Data Fig. 6, confirming that 'L6b VISp Col8a1 Rprm' attaches under 'L6b Sla' rather than 'L6b Rgs12' and that the three L4 populations merge.

## Related tools

- **scHPL** (Core method: learns relationships between cell populations across datasets and constructs the hierarchical classification tree (scHPL.learn), trains per-node classifiers, and predicts labels for new datasets) — https://github.com/lcmmichielsen/scHPL
- **Seurat** (Alignment of the AMB2016 and AMB2018 datasets before population matching (integrated expression matrix fed to scHPL))
- **scikit-learn** (Provides the linear SVM trained at each node of the learned tree for integrated data retaining many dimensions)
- **scArches** (Recommended batch-correction/alignment framework (via treeArches) to align datasets before scHPL population matching) — https://github.com/theislab/scarches

## Examples

```
import scHPL; tree = scHPL.learn(amb2016_aligned, amb2018_aligned, classifier='linear_svm')
```

## Evaluation signals

- Learned tree reproduces the published hierarchy from Tasic et al. Extended Data Fig. 6 (near-perfect agreement for AMB2016–AMB2018)
- 'L6b VISp Col8a1 Rprm' is assigned as a subpopulation of 'L6b Sla' rather than 'L6b Rgs12'
- The three L4 populations merge into a single branch in the learned tree
- Linear SVM accuracy is equal to or higher than the one-class SVM across datasets; check per-node classifier choice, especially for small populations
- Erroneous tree structure should trigger marker-gene expression checks to detect mislabeled populations (as done in the PBMC inter-dataset experiment)

## Limitations

- Adding a new dataset requires rerunning the alignment and consequently scHPL with current batch correction tools
- Batch effects make it hard to determine whether tree mistakes stem from erroneous alignment or mismatches created by scHPL
- Many brain cell populations are missing from Cell Ontology and there is wide lack of consistency in population annotation between studies, limiting external validation
- Inherent limitation of the hierarchical-relationship assumption: a tree is inadequate for dedifferentiation-like conversions (e.g., beta to alpha cells in type 2 diabetes)
- One-class SVM performance decreases more on populations with a small number of cells

## Evidence

- [methods] scHPL is not a batch correction tool, we advise to align the datasets before matching the cell populations.: "scHPL is not a batch correction tool, we advise to align the datasets before matching the cell populations."
- [discussion] Linear SVM outperforms or matches one-class SVM: "For both classifiers, we saw a decrease in performance on populations with a small number of cells, but for the one-class SVM this effect was more apparent."
- [discussion] Col8a1 Rprm mapping: "such as the mapping of “L6b VISp Col8a1 Rprm” as a subpopulation of “L6b Sla” instead of “L6b Rgs12”"
- [readme] Tree construction and node classifiers (README): "automatically finds relationships between cell populations across multiple datasets and uses this to construct a hierarchical classification tree."
- [discussion] Rerun on new data: "when a new dataset is added, the alignment—and consequently also scHPL—has to be rerun"
