---
name: tree-stability-assessment-across-batch-orders
description: Use when multiple annotated, aligned datasets (e.g. simulated batches
  or PBMC-FACS) are combined with scHPL and you need to quantify how batch presentation
  order or missing populations affect the learned tree, or to compare linear SVM vs
  one-class SVM node classifiers.
when_to_use_negative:
- Datasets have not been aligned/integrated first — scHPL is not a batch correction
  tool, so population matching across unaligned batches is out of scope
- The goal is annotating a new unlabeled dataset with an already-trained tree (use
  scHPL predict) rather than assessing tree construction stability
- Only a single annotated dataset with a fixed known hierarchy is available and no
  cross-batch progressive learning is involved — there is no batch-order variation
  to assess
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scHPL
  role: 'Core method: scHPL.learn performs the progressive learning that constructs
    the hierarchical classification trees over batch orders and folds; node classifiers
    are trained as linear SVM or one-class SVM'
  repo: https://github.com/lcmmichielsen/scHPL
- name: scArches
  role: Recommended upstream alignment of datasets before scHPL matches cell populations,
    since scHPL requires aligned inputs
  repo: https://github.com/theislab/scarches
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-021-23196-8
    title: Hierarchical progressive learning of cell identities in single-cell data
  source_content_hash: sha256:e6e2b75e53ff381b8d5b71b2877b824d15371d41b7afb481ff4aa1fa98da5ea0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lcmmichielsen/scHPL
  tools:
  - scArches
  - scHPL
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# tree-stability-assessment-across-batch-orders

## Summary

Assess the stability and correctness of hierarchical classification trees learned by scHPL by running progressive learning over multiple batch orders and folds, then comparing each learned tree against the expected reference hierarchy — including stress tests where a cell population (e.g. 'Group5') is withheld from a training batch.

## When to use

Use when multiple annotated, aligned datasets (e.g. simulated batches or PBMC-FACS) are combined with scHPL and you need to quantify how batch presentation order or missing populations affect the learned tree, or to compare linear SVM vs one-class SVM node classifiers.

## When NOT to use

- Datasets have not been aligned/integrated first — scHPL is not a batch correction tool, so population matching across unaligned batches is out of scope
- The goal is annotating a new unlabeled dataset with an already-trained tree (use scHPL predict) rather than assessing tree construction stability
- Only a single annotated dataset with a fixed known hierarchy is available and no cross-batch progressive learning is involved — there is no batch-order variation to assess

## Inputs

- Aligned annotated single-cell datasets as AnnData objects (simulated multi-batch data; PBMC-FACS from the scRNA-seq benchmark Zenodo record 3357167) with cell labels and batch indicator metadata columns and the expression matrix in .X
- Expected reference hierarchy (cell population taxonomy) for comparison
- scHPL Zenodo datasets (doi:10.5281/zenodo.4557712) containing the simulated and aligned inter-dataset experiment data

## Outputs

- 60 hierarchical classification trees per dataset condition (six batch orders x 10 folds) for each classifier (linear SVM, one-class SVM)
- Tree stability counts: number of divergent vs consistent trees per condition (e.g. 58/60 consistent PBMC trees; 2/60 divergent)
- Missing-'Group5' experiment outcome: runs where linear SVM mis-constructs all trees vs one-class SVM succeeds (2 of 3 runs)
- tree_stability_results.csv with tree stability counts and classifier comparison outcomes

## How to apply

Prepare each annotated dataset as an AnnData object with cell labels and batch indicators as metadata columns and the aligned expression matrix in .X, as required by scHPL.learn. Run the scHPL progressive learning procedure over six batch orders x 10 folds to construct 60 hierarchical classification trees per dataset condition (simulated and PBMC-FACS), letting scHPL automatically match cell populations across batches. Train per-node classifiers twice — once with a linear SVM and once with a one-class SVM — and compare each learned tree against the expected reference hierarchy, tallying divergent trees (e.g. 2/60 divergent PBMC trees, i.e. 58/60 consistent). Then run the missing-population stress test: completely remove 'Group5' from Batch 2 of the simulated data, rebuild trees with each classifier, and count runs where the tree is mis-constructed. The reported contrast is that the linear SVM mis-constructs all 60 learned trees while the one-class SVM recovers the expected tree in two-thirds of the runs; write counts to tree_stability_results.csv. Rationale: batch effects mean datasets must be aligned before matching populations, so any tree error must be attributed to alignment versus scHPL-created mismatches carefully.

## Related tools

- **scHPL** (Core method: scHPL.learn performs the progressive learning that constructs the hierarchical classification trees over batch orders and folds; node classifiers are trained as linear SVM or one-class SVM) — https://github.com/lcmmichielsen/scHPL
- **scArches** (Recommended upstream alignment of datasets before scHPL matches cell populations, since scHPL requires aligned inputs) — https://github.com/theislab/scarches

## Examples

```
import scHPL; tree = scHPL.learn(adata_list, classifier='onevslin', annotate=False, batch_order=fold_batch_order)
```

## Evaluation signals

- PBMC-FACS trees: at least 58 of 60 learned trees (6 batch orders x 10 folds) match the expected reference hierarchy
- Missing-'Group5' contrast reproduces: linear SVM mis-constructs trees in all runs, one-class SVM correctly learns the expected tree in 2 of 3 runs
- Each AnnData input carries cell labels and batch indicators as metadata columns with the aligned expression matrix in .X, as scHPL.learn expects
- Six batch orders x 10 folds yields exactly 60 trees per dataset condition and classifier, and all counts are recorded in tree_stability_results.csv
- Divergent trees are tallied against the expected reference hierarchy rather than ad hoc visual inspection

## Limitations

- scHPL is not a batch correction tool — datasets must be aligned beforehand, so tree mistakes may stem from erroneous alignment rather than mismatches created by scHPL and it is hard to disentangle the two
- One-class SVM performance decreases more than linear SVM on populations with a small number of cells
- Adding a new dataset requires rerunning the alignment and consequently scHPL with current batch correction tools
- The hierarchical-relationship assumption is inherently limited, e.g. dedifferentiation such as beta-to-alpha cell conversions in type 2 diabetes is not tree-adequate

## Evidence

- [results] scHPL automatically finds relationships between cell populations across multiple datasets and constructs hierarchical trees; stability is assessed by presenting batches in different orders: "For each node in the tree either a linear SVM, kNN, or one-class SVM is trained."
- [readme] Linear vs one-class SVM: one-class SVM enables detection of unknown populations: "For each node in the tree either a linear SVM, kNN, or one-class SVM, which enables the detection of unknown populations, is trained."
- [discussion] Datasets must be aligned before running scHPL; batch effects are inevitable when combining datasets: "Since batch effects are inevitable when combining datasets, we require datasets to be aligned before running scHPL"
- [discussion] It is hard to resolve whether tree mistakes come from erroneous alignment or mismatches created by scHPL: "it will be hard to resolve whether mistakes in the constructed tree are caused by the erroneous alignment of datasets or by mismatches created by scHPL"
- [readme] Simulated and aligned inter-dataset experiment data are downloadable from the scHPL Zenodo record: [evidence span withheld — anchor status: external_doc]
- [discussion] One-class SVM performance decreases more on small populations than linear SVM: "For both classifiers, we saw a decrease in performance on populations with a small number of cells, but for the one-class SVM this effect was more apparent."
