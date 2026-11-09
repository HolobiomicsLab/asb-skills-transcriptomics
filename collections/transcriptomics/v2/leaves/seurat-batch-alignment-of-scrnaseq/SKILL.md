---
name: seurat-batch-alignment-of-scrnaseq
description: Use when two or more scRNA-seq datasets (e.g. AMB2016 and AMB2018 visual
  cortex) must be matched at the cell-population level by scHPL, since batch effects
  make cross-dataset comparison unreliable without prior alignment.
when_to_use_negative:
- You need to correct batch effects themselves or interpret biological signal in the
  integrated space — scHPL only consumes the aligned output and the paper notes the
  alignment must be rerun when new data are added
- Only a single dataset is involved and no cross-dataset population matching is required
  — alignment adds no value before hierarchical learning
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: Seurat
  role: Performs the batch alignment/integration of the scRNA-seq datasets before
    population matching
- name: scHPL
  role: Downstream consumer of the aligned datasets; learns the hierarchical classification
    tree on the aligned .X matrix
  repo: https://github.com/lcmmichielsen/scHPL
- name: scArches
  role: Alternative alignment framework advised by the scHPL README (via treeArches)
    for integrating query data into a reference
  repo: https://github.com/theislab/scarches
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-021-23196-8
    title: Hierarchical progressive learning of cell identities in single-cell data
  source_content_hash: sha256:ed26834c562100d7abe4e89770d003cf79e083dbe46f22801e95f3bd11f0ad4c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lcmmichielsen/scHPL
  tools:
  - Seurat
  - scArches
  - scHPL
  - scikit-learn
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# seurat-batch-alignment-of-scrnaseq

## Summary

Align single-cell RNA-seq datasets across batches with Seurat so that batch effects are removed before hierarchical cell-population matching. This is a prerequisite because scHPL is not a batch-correction tool and will construct erroneous population relationships on unaligned data.

## When to use

Use when two or more scRNA-seq datasets (e.g. AMB2016 and AMB2018 visual cortex) must be matched at the cell-population level by scHPL, since batch effects make cross-dataset comparison unreliable without prior alignment.

## When NOT to use

- You need to correct batch effects themselves or interpret biological signal in the integrated space — scHPL only consumes the aligned output and the paper notes the alignment must be rerun when new data are added
- Only a single dataset is involved and no cross-dataset population matching is required — alignment adds no value before hierarchical learning

## Inputs

- Two or more annotated scRNA-seq count matrices (e.g. AMB2016 and AMB2018 AnnData objects with population labels in metadata)
- Batch indicator variables per dataset/cell

## Outputs

- Aligned/integrated expression matrix placed in the AnnData .X slot
- Metadata with batch indicators and population labels ready for scHPL.learn

## How to apply

Load the datasets as AnnData objects and run Seurat-based integration so the shared low-dimensional embedding lands in the .X slot, with batch indicators and population labels in the metadata. For the AMB2016-AMB2018 mouse primary visual cortex example, both datasets are Seurat-aligned before any scHPL call. The rationale is that scHPL automatically matches cell populations across datasets, so unremoved batch effects would either cause spurious mismatches or hide true correspondences, and afterwards it is hard to determine whether tree mistakes stem from erroneous alignment or mismatches created by scHPL. Note that alignment must be rerun whenever a new dataset is added, which in turn requires rerunning scHPL.

## Related tools

- **Seurat** (Performs the batch alignment/integration of the scRNA-seq datasets before population matching)
- **scHPL** (Downstream consumer of the aligned datasets; learns the hierarchical classification tree on the aligned .X matrix) — https://github.com/lcmmichielsen/scHPL
- **scArches** (Alternative alignment framework advised by the scHPL README (via treeArches) for integrating query data into a reference) — https://github.com/theislab/scarches

## Examples

```
import scHPL; tree = scHPL.learn(amb2016_aligned, amb2018_aligned, classifier='linear_svm')
```

## Evaluation signals

- After scHPL.learn on the aligned AMB2016/AMB2018 data, the learned tree reproduces the near-perfect agreement with the published Tasic et al. Extended Data Fig. 6 hierarchy
- Specific topology checks: 'L6b VISp Col8a1 Rprm' attaches under 'L6b Sla' rather than 'L6b Rgs12', and the three L4 populations merge into a single branch
- Downstream linear SVM classifiers trained at each node of the learned tree achieve accuracy equal to or higher than one-class SVM on the aligned datasets
- Cells from the two batches are intermixed in the integrated embedding rather than separating by batch, so population matching does not follow batch boundaries

## Limitations

- Batch effects make it hard to determine whether mistakes in the constructed tree are caused by erroneous alignment or by mismatches created by scHPL
- When a new dataset is added, the alignment — and consequently scHPL — has to be rerun with current batch correction tools
- Alignment alone does not guarantee a correct tree: inherent hierarchical-relationship assumptions can still fail (e.g. dedifferentiation-like conversions between cell states)

## Evidence

- [readme] scHPL is not a batch correction tool, we advise to align the datasets before matching the cell populations.: "NOTE: scHPL is not a batch correction tool, we advise to align the datasets before matching the cell populations."
- [discussion] Datasets must be aligned before running scHPL: "Since batch effects are inevitable when combining datasets, we require datasets to be aligned before running scHPL"
- [discussion] Rerunning alignment when new data arrive: "when a new dataset is added, the alignment—and consequently also scHPL—has to be rerun"
- [discussion] Diagnostic difficulty of erroneous alignment: "it will be hard to resolve whether mistakes in the constructed tree are caused by the erroneous alignment of datasets or by mismatches created by scHPL"
