---
name: progressive-learning-tree-construction
description: Use when multiple annotated single-cell datasets (e.g., from different
  labs or protocols, such as Saunders, Zeisel, Tabula Muris brain data) must be combined
  into one consistent cell-population hierarchy, especially when annotation terminologies
  differ between studies.
when_to_use_negative:
- Datasets are not yet batch-corrected/aligned — scHPL is not a batch correction tool
  and mismatches may result from erroneous alignment
- The cell populations violate a hierarchical relationship, e.g., dedifferentiation
  like beta-to-alpha cell conversions in type 2 diabetes
- A standardized constant terminology across datasets is assumed available and simple
  name-based matching would suffice (scHPL's value is discovering relationships names
  cannot)
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scHPL
  role: 'Primary method: learns relationships between cell populations across datasets
    and constructs the hierarchical classification tree (scHPL.learn); trains per-node
    classifiers (scHPL.train) and predicts labels for new datasets (scHPL.predict)'
  repo: https://github.com/lcmmichielsen/scHPL
- name: Seurat
  role: Used to align/batch-correct the Saunders, Zeisel, and Tabula Muris datasets
    before running scHPL
- name: scArches
  role: Recommended (via treeArches) as an alternative batch-correction/alignment
    tool to align datasets before matching cell populations
  repo: https://github.com/theislab/scarches
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-021-23196-8
    title: Hierarchical progressive learning of cell identities in single-cell data
  source_content_hash: sha256:13b1bd812c3a742371c042bdae541bc18200238c0ed7180b7b65ed93d16f5e4d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lcmmichielsen/scHPL
  tools:
  - Seurat
  - scArches
  - scHPL
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# progressive-learning-tree-construction

## Summary

Construct a hierarchical classification tree of cell identities by progressively learning relationships between cell populations across multiple annotated scRNA-seq datasets with scHPL, adding datasets sequentially so that populations are matched across studies and merged into a shared hierarchy.

## When to use

Use when multiple annotated single-cell datasets (e.g., from different labs or protocols, such as Saunders, Zeisel, Tabula Muris brain data) must be combined into one consistent cell-population hierarchy, especially when annotation terminologies differ between studies.

## When NOT to use

- Datasets are not yet batch-corrected/aligned — scHPL is not a batch correction tool and mismatches may result from erroneous alignment
- The cell populations violate a hierarchical relationship, e.g., dedifferentiation like beta-to-alpha cell conversions in type 2 diabetes
- A standardized constant terminology across datasets is assumed available and simple name-based matching would suffice (scHPL's value is discovering relationships names cannot)

## Inputs

- Aligned annotated scRNA-seq datasets (AnnData objects with cell labels and batch indicators as metadata columns and aligned expression matrix in .X), e.g., Seurat-aligned Saunders, Zeisel, and Tabula Muris brain datasets

## Outputs

- A hierarchical classification tree relating cell populations across datasets (reproducing matches such as Saunders mural cells to Tabula Muris brain pericytes)
- Per-node trained classifiers (linear SVM, kNN, or one-class SVM)

## How to apply

Align the datasets first (e.g., Seurat-aligned AnnData objects with labels and batch indicators in metadata and aligned expression in .X), since scHPL is not a batch correction tool. Start scHPL.learn with the first annotated dataset (e.g., Saunders) and progressively add subsequent datasets (e.g., Zeisel low-resolution annotations, then Tabula Muris) so the method automatically finds relationships between populations across datasets — including non-name-based matches such as Saunders mural cells matching Tabula Muris brain pericytes — and constructs the hierarchical tree. At each tree node train a classifier (linear SVM, kNN, or one-class SVM); the article advises a linear SVM for integrated data with many dimensions. The resulting tree can then be used with scHPL.predict to label new unlabeled datasets, with a rejection option for populations that do not match tree nodes.

## Related tools

- **scHPL** (Primary method: learns relationships between cell populations across datasets and constructs the hierarchical classification tree (scHPL.learn); trains per-node classifiers (scHPL.train) and predicts labels for new datasets (scHPL.predict)) — https://github.com/lcmmichielsen/scHPL
- **Seurat** (Used to align/batch-correct the Saunders, Zeisel, and Tabula Muris datasets before running scHPL)
- **scArches** (Recommended (via treeArches) as an alternative batch-correction/alignment tool to align datasets before matching cell populations) — https://github.com/theislab/scarches

## Examples

```
from scHPL import learn; tree = learn.progressive_learning(data_saunders, data_zeisel, data_tabula_muris)
```

## Evaluation signals

- The reproduced tree contains the reported cross-dataset matches, e.g., Saunders mural cell populations matched to Tabula Muris brain pericytes, checkable against Fig. 7G
- Per-node linear-SVM accuracy is equal to or higher than the one-class SVM baseline; note both classifiers show decreased performance on small populations, more pronounced for the one-class SVM
- Predictions on an independent dataset (e.g., Rosenberg) match the assignments shown in the corresponding paper figure, with rejection where populations do not match tree nodes
- Original dataset annotations are preserved while known hierarchies are learned (the abstract's stated success criterion)

## Limitations

- scHPL requires datasets to be aligned beforehand; when a new dataset is added, the alignment — and consequently scHPL — has to be rerun with current batch correction tools
- Batch effects make it hard to determine whether tree mistakes stem from erroneous alignment or mismatches created by scHPL
- The hierarchical-relationship assumption fails in situations such as dedifferentiation (e.g., beta-to-alpha cell conversions in type 2 diabetes)
- Many brain cell populations are missing from the Cell Ontology, so it cannot be used to verify learned relationships

## Evidence

- [methods] scHPL is a method that automatically finds relationships between cell populations across multiple datasets and uses this to construct a hierarchical classification tree.: "scHPL is a method that automatically finds relationships between cell populations across multiple datasets and uses this to construct a hierarchical classification tree."
- [methods] For each node in the tree either a linear SVM, kNN, or one-class SVM is trained.: "For each node in the tree either a linear SVM, kNN, or one-class SVM is trained."
- [discussion] Since batch effects are inevitable when combining datasets, we require datasets to be aligned before running scHPL: "Since batch effects are inevitable when combining datasets, we require datasets to be aligned before running scHPL"
- [readme] NOTE: scHPL is not a batch correction tool, we advise to align the datasets before matching the cell populations. We advise doing this with scVI or scArches: "NOTE: scHPL is not a batch correction tool, we advise to align the datasets before matching the cell populations. We advise doing this with scVI or scArches"
- [abstract] show that scHPL can successfully learn known cellular hierarchies from multiple datasets while preserving the original annotations: "show that scHPL can successfully learn known cellular hierarchies from multiple datasets while preserving the original annotations"
