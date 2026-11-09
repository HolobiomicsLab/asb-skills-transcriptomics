---
name: inflection-point-based-cluster-set-selection
description: Use when reconstructing a single-cell trajectory that needs discrete
  cell-state landmarks, and the number of clusters k must be chosen objectively from
  clustering-purity or landmark-count curves (e.g. FACS-label-guided cuts of hclust
  dendrograms).
when_to_use_negative:
- The starting cell types are multiple independent progenitor populations — Mpath
  is designed for processes stemming from a single progenitor cell type and does not
  yet support this (planned future extension).
- Specific subset information is unknown and no FACS-sorted terminal subsets or signature
  genes from end-stage contrasts are available — landmark designation and fate prediction
  depend on them.
edam_operation: http://edamontology.org/operation_0313
tools:
- name: Mpath
  role: 'Provides the landmark designation algorithm: dendrogram cutting, purity filtering,
    and inflexion-point selection of the optimal cluster set'
  repo: JinmiaoChenLab/Mpath
- name: R
  role: Implementation environment for Euclidean distances, hclust with Ward.D linkage,
    and iterative cutree over candidate cluster counts
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/ncomms11988
    title: Mpath maps multi-branching single-cell trajectories revealing progenitor
      cell progression during development
  source_content_hash: sha256:3c5c60c0df9edd85ea7983728d30391516eca7f9e90dd07b9318a75f7d227c9f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JinmiaoChenLab/Mpath
  tools:
  - Mpath
  - R
  edam_operation: http://edamontology.org/operation_0313
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# inflection-point-based-cluster-set-selection

## Summary

Mpath's landmark designation procedure: agglomerative clustering (Ward.D) of single-cell expression data, iterative cutting of the dendrogram, and selection of the optimal cluster set at the inflexion point of landmarks-versus-clusters (or purity) curves, yielding discrete cell-state landmark clusters.

## When to use

Use when reconstructing a single-cell trajectory that needs discrete cell-state landmarks, and the number of clusters k must be chosen objectively from clustering-purity or landmark-count curves (e.g. FACS-label-guided cuts of hclust dendrograms).

## When NOT to use

- The starting cell types are multiple independent progenitor populations — Mpath is designed for processes stemming from a single progenitor cell type and does not yet support this (planned future extension).
- Specific subset information is unknown and no FACS-sorted terminal subsets or signature genes from end-stage contrasts are available — landmark designation and fate prediction depend on them.

## Inputs

- Expression matrix over Mpath-selected genes for all single cells
- FACS-sort labels (known terminal subset identity) per cell

## Outputs

- Final landmark designation: cell-to-cluster assignments for the selected dendrogram cut (landmark_assignations.csv)
- Set of landmark clusters representing discrete cell states (e.g. 8 landmarks from 11 clusters on DC progenitor data)

## How to apply

Load the expression matrix over Mpath-selected genes together with the FACS-sort label for each cell. Compute Euclidean distances between cells and build an agglomerative hierarchy with Ward.D linkage via R's hclust. Iteratively cut the dendrogram over a candidate range of cluster counts k; for each cut, quantify every cluster's purity as the Shannon index of the composition of FACS-sort labels it contains, and discard candidate sets failing purity and minimum-cluster-size cutoffs. Plot the number of landmarks against the total number of clusters (and/or cluster purity against number of clusters) and select the optimal cut at the inflexion point of the curve. On the mouse DC progenitor (CDP/preDC) data this inflexion occurred at 11 clusters, from which 8 landmark clusters were designated; write the resulting cell-to-cluster assignments to landmark_assignations.csv.

## Related tools

- **Mpath** (Provides the landmark designation algorithm: dendrogram cutting, purity filtering, and inflexion-point selection of the optimal cluster set) — JinmiaoChenLab/Mpath
- **R** (Implementation environment for Euclidean distances, hclust with Ward.D linkage, and iterative cutree over candidate cluster counts)

## Examples

```
hc <- hclust(dist(expr_matrix), method="ward.D"); clusters <- cutree(hc, k=11)
```

## Evaluation signals

- The plot of number of landmarks versus total number of clusters shows a clear inflexion point; on the DC progenitor data this occurs at 11 clusters yielding 8 landmark clusters.
- Every retained cluster in the selected cut passes the purity and minimum-cluster-size cutoffs, with cluster purity quantified as the Shannon index of FACS-sort label composition.
- The dendrogram was built from Euclidean distances with Ward.D linkage in hclust, matching the Mpath reference workflow.
- The output landmark_assignations.csv assigns exactly one landmark cluster to every cell in the input expression matrix (a complete partition).

## Limitations

- Mpath does not yet support development processes starting from multiple independent progenitor cell types (planned future extension).
- Mpath is not yet extended to highly heterogeneous single-cell data sets where specific subset information is unknown (planned future extension).
- Landmark designation depends on FACS-sorted terminal subsets; candidates failing the purity or minimum-cluster-size cutoffs are discarded.

## Evidence

- [results] Mpath designates landmarks by first clustering cells and then selecting the optimal cluster set: [evidence span withheld — anchor status: not_found]
- [abstract] Here we present Mpath, an algorithm that derives multi-branching developmental trajectories using neighborhood-based cell state transitions.: "Here we present Mpath, an algorithm that derives multi-branching developmental trajectories using neighborhood-based cell state transitions."
- [discussion] Mpath uses count of neighbouring cells to estimate potential state transition.: "Mpath uses count of neighbouring cells to estimate potential state transition."
- [readme] Mpath: an algorithm for constructing multi-branching cell lineages from single-cell data: "Mpath: an algorithm for constructing multi-branching cell lineages from single-cell data"
- [discussion] In future, we will extend the Mpath algorithm for development processes starting from multiple independent progenitor cell types: "In future, we will extend the Mpath algorithm for development processes starting from multiple independent progenitor cell types"
