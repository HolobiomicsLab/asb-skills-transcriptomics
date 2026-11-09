---
name: major-minor-cell-type-label-splitting
description: Use when benchmarking a single-cell clustering method against curated
  cell-type annotations and you need to assess performance at both coarse (major)
  and fine-grained (minor) label granularity, e.g. resolving rare populations like
  MAIT-NKT cells.
when_to_use_negative:
- The dataset has only a single flat annotation level with no coarse/fine hierarchy
  to split
- The goal is clustering itself rather than evaluating clustering quality against
  reference labels
- Labels are continuous scores or predicted probabilities rather than discrete categorical
  cell types
edam_operation: http://edamontology.org/operation_3649
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2229
tools:
- name: SnapCCESS
  role: Generates consensus clusterings (base ensemble size 1 vs. ensemble size 50,
    epoch=1) that are scored by ARI against the major and minor label sets
  repo: https://github.com/PYangLab/SnapCCESS
- name: snapccess
  role: Python package used to generate multimodality embeddings and consensus clustering
    for each ensemble configuration
  repo: https://github.com/yulijia/SnapCCESS/tree/main/snapccess-py
- name: R
  role: Used via the SnapCCESS R wrapper to run the pipeline and prepare cell-type
    label splits on the tutorial/paper datasets
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad382
    title: Ensemble deep learning of embeddings for clustering multimodal single-cell
      omics data
  source_content_hash: sha256:f5d128861432f0220f0541d3e1d9f9a42457bd4a876297d8446ef36a6c86bb36
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PYangLab/SnapCCESS
  tools:
  - Python
  - R
  - SnapCCESS
  - remotes
  - snapccess
  edam_operation: http://edamontology.org/operation_3649
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# major-minor-cell-type-label-splitting

## Summary

Splits each dataset's cell-type annotations into two hierarchical label sets — coarse major cell types and fine-grained minor cell types — so clustering performance can be evaluated separately at both resolutions, as done in the SnapCCESS evaluation on six multimodal single-cell datasets.

## When to use

Use when benchmarking a single-cell clustering method against curated cell-type annotations and you need to assess performance at both coarse (major) and fine-grained (minor) label granularity, e.g. resolving rare populations like MAIT-NKT cells.

## When NOT to use

- The dataset has only a single flat annotation level with no coarse/fine hierarchy to split
- The goal is clustering itself rather than evaluating clustering quality against reference labels
- Labels are continuous scores or predicted probabilities rather than discrete categorical cell types

## Inputs

- Cell-type annotation file per dataset (e.g. CITE-seq cell type labels for the Ramaswamy CITE-seq data) with one label per cell
- Multimodal single-cell omics matrices (e.g. RNA and ADT from CITE-seq) aligned cell-by-cell with the annotations
- Consensus cluster assignments from the clustering method under evaluation (e.g. SnapCCESS with ensemble size 1 and 50)

## Outputs

- Major cell-type label set per dataset (coarse labels over all cells)
- Minor cell-type label set per dataset (fine-grained labels over all cells)
- ARI values of consensus clusterings against major and minor partitions per dataset (e.g. ari_ensemble_comparison.csv)
- Figures highlighting rare minor populations, e.g. MAIT-NKT cells in the Ramaswamy CITE-seq embedding (figure3.png, figure_S3.png)

## How to apply

From each dataset's cell-type annotation file (e.g. CITE-seq cell type columns such as the Ramaswamy CITE-seq data), derive two label vectors over the same cells: a major label set that collapses fine annotations into broad lineages, and a minor label set that retains fine-grained subtypes. Keep the two partitions mutually consistent so every minor label maps onto exactly one major label. Run the clustering being evaluated once per configuration (e.g. SnapCCESS epoch=1 with ensemble size 1 vs. ensemble size 50), then compute the adjusted Rand index (ARI) of each consensus clustering separately against the major labels and against the minor labels, per dataset. Tabulate the paired comparison (e.g. ari_ensemble_comparison.csv) so the ensemble-vs-base difference can be read at each granularity. Rationale: rare minor populations are the hardest to recover, so agreement on the minor partition is the stricter validation criterion — an ensemble of snapshots should match or improve the base ARI on minor cell types.

## Related tools

- **SnapCCESS** (Generates consensus clusterings (base ensemble size 1 vs. ensemble size 50, epoch=1) that are scored by ARI against the major and minor label sets) — https://github.com/PYangLab/SnapCCESS
- **snapccess** (Python package used to generate multimodality embeddings and consensus clustering for each ensemble configuration) — https://github.com/yulijia/SnapCCESS/tree/main/snapccess-py
- **R** (Used via the SnapCCESS R wrapper to run the pipeline and prepare cell-type label splits on the tutorial/paper datasets)

## Examples

```
pip install snapccess --index-url https://pypi.org/simple
```

## Evaluation signals

- The ensemble-size-50 ARI against the minor cell-type labels should match or exceed the base (ensemble size 1) ARI in each of the six datasets
- Major-partition ARI should be at least as high as minor-partition ARI for the same clustering (coarser labels are easier to recover)
- Tabulated ARI values should be consistent with those reported in the SnapCCESS paper (Bioinformatics 39(6), btad382)
- Ramaswamy CITE-seq embedding figures must render MAIT-NKT cells highlighted, matching Figure 3 and Supplementary Fig. S3
- Major and minor label sets must cover the same set of cells, with each minor label mapping to exactly one major label

## Limitations

- The SnapCCESS tutorials do not recommend best hyperparameters for new datasets; parameters for the paper's datasets are only in the paper
- ARI comparison depends on curated ground-truth annotations, which may not exist or may differ in granularity across datasets
- Rare minor populations can still be missed; the paper's workflow explicitly validates via the MAIT-NKT case study rather than claiming universal recovery

## Evidence

- [methods] Splitting cell-type annotations into major/minor label sets for ARI evaluation: [evidence span withheld — anchor status: not_found]
- [methods] ARI used to compare ensemble vs base clustering against minor labels: [evidence span withheld — anchor status: not_found]
- [results] Ensemble improves major and minor cell-type identification in all six datasets: [evidence span withheld — anchor status: not_found]
- [methods] Validation: figures must highlight MAIT-NKT cells as in Figure 3 and S3: [evidence span withheld — anchor status: not_found]
- [readme] Tutorials don't give best parameters; paper lists them: "This tutorial only explains how to use this package; it doesn't recommend the best parameters for your datasets."
