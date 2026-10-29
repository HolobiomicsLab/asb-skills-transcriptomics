---
name: louvain-clustering-of-control-cells
description: Use when a perturbation scRNAseq analysis requires reference cell-state
  labels from the control condition alone (scanpy neighbors + Louvain on control cells)
  before NCFS feature weighting and semi-supervised clustering.
when_to_use_negative:
- When the goal is de novo clustering of the full perturbed-plus-control dataset without
  control-guided anchoring — ICAT's sslouvain, not plain Louvain on controls, is the
  appropriate step
- When cell states or marker genes are already known a priori and supervised label
  transfer would be more appropriate
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scanpy
  role: Provides sc.pp.neighbors and sc.tl.louvain used to build the control-cell
    graph and perform the unsupervised Louvain clustering
- name: icat-sc
  role: Provides simulate.SingleCellDataset for generating control data and icat.optimize.optimize_louvain
    for grid-searching n_neighbors/resolution via the Calinski-Harabasz Index
  repo: https://github.com/BradhamLab/icat
- name: AnnData
  role: Data structure holding the control counts, treatment labels, and resulting
    louvain cluster assignments
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad278
    title: 'ICAT: a novel algorithm to robustly identify cell states following perturbations
      in single-cell transcriptomes'
  source_content_hash: sha256:9b3b1d2ef58ad3512bf81c57ab41d507b7d3faffd88ee231d285c91318b8d97c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BradhamLab/icat
  tools:
  - AnnData
  - icat-sc
  - numpy
  - scanpy
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# louvain-clustering-of-control-cells

## Summary

Unsupervised Louvain clustering of control-arm single-cell data to define control-defined cell states whose labels anchor ICAT's downstream semi-supervised clustering of perturbed cells.

## When to use

Use when a perturbation scRNAseq analysis requires reference cell-state labels from the control condition alone (scanpy neighbors + Louvain on control cells) before NCFS feature weighting and semi-supervised clustering.

## When NOT to use

- When the goal is de novo clustering of the full perturbed-plus-control dataset without control-guided anchoring — ICAT's sslouvain, not plain Louvain on controls, is the appropriate step
- When cell states or marker genes are already known a priori and supervised label transfer would be more appropriate

## Inputs

- Normalized (sc.pp.log1p) control-arm AnnData object
- Hyperparameter grids for n_neighbors and resolution (optional, via icat.optimize)

## Outputs

- Per-control-cell Louvain cluster labels (e.g. controls.obs['louvain']) defining control cell states
- Chosen n_neighbors and resolution values (louvain_n, louvain_r) when optimize_louvain is used

## How to apply

Isolate the control-arm AnnData (e.g. controls.obs['treatment'] == 'control') after log1p normalization, build a neighbor graph with scanpy sc.pp.neighbors, and run sc.tl.louvain to obtain unsupervised control cell-state labels. Optionally optimize the n_neighbors and resolution hyperparameters first using icat.optimize.optimize_louvain, which maximizes the Calinski-Harabasz Index over a grid (e.g. neighbors 3-50 step 2, resolution 0.3-1.2 step 0.02). Record the resulting labels (e.g. controls.obs.louvain) as control_clusters, since these labels are later fed as immutable targets into NCFS gradient-ascent feature weighting and the semi-supervised sslouvain step while perturbed cells are reinitialized as singletons. The rationale is that ICAT identifies control-defined states first, then uses those labels to guide clustering of treated cells so co-clustering reflects state membership rather than treatment-driven integration artifacts.

## Related tools

- **scanpy** (Provides sc.pp.neighbors and sc.tl.louvain used to build the control-cell graph and perform the unsupervised Louvain clustering)
- **icat-sc** (Provides simulate.SingleCellDataset for generating control data and icat.optimize.optimize_louvain for grid-searching n_neighbors/resolution via the Calinski-Harabasz Index) — https://github.com/BradhamLab/icat
- **AnnData** (Data structure holding the control counts, treatment labels, and resulting louvain cluster assignments)

## Examples

```
louvain_n, louvain_r = optimize.optimize_louvain(controls, min_neighbors=3, max_neighbors=50, neighbor_step=2, min_res=0.3, max_res=1.2, res_step=0.02); sc.pp.neighbors(controls, n_neighbors=louvain_n); sc.tl.louvain(controls, resolution=louvain_r)
```

## Evaluation signals

- Control labels (control_clusters) are populated for every control cell and absent/NaN for perturbed cells
- optimize_louvain returns a plausible (louvain_n, louvain_r) pair within the search grid (neighbors 3-50, resolution 0.3-1.2) that maximizes the Calinski-Harabasz Index
- On the equal-population simulated dataset (populations=2, genes=1000), the downstream sslouvain clusters achieve ARI ≈ 1 against true population labels, implying correct control-state anchors
- Louvain assignments on controls recover the simulated population structure (recoverable number of clusters ≈ number of simulated populations)

## Limitations

- Louvain hyperparameters (n_neighbors, resolution) must be tuned per dataset; defaults may not generalize — the README recommends grid search prior to running the full ICAT workflow
- Control-defined states assume control cells adequately represent all extant states; novel perturbation-unique states may be forced into existing clusters unless initialized as singletons downstream

## Evidence

- [intro] The method works by first identifying a set of conrol-defined cell states by performing unsupervised clustering.: "The method works by first identifying a set of conrol-defined cell states by performing unsupervised clustering."
- [readme] optimize_louvain grid search: "optimizing the Calinski-Harabasz Index over a grid of `n` and `r` values"
- [methods] control clustering commands: "sc.pp.neighbors(controls, n_neighbors=louvain_n)
sc.tl.louvain(controls, resolution=louvain_r)"
- [intro] semi-supervised rationale: "perform semi-supervised clustering such that the originally identified control labels remain constant"
