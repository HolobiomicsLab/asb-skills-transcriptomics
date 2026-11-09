---
name: dendrogram-iterative-cutting
description: Use when designating landmark clusters from single-cell expression data
  by cutting a Ward.D hclust dendrogram at multiple candidate k values and selecting
  the cut via a purity-versus-number-of-clusters inflection criterion.
when_to_use_negative:
- The development process starts from multiple independent progenitor cell types or
  the relevant subset information is unknown — Mpath does not yet support these cases
- No FACS-sort labels or other known subset annotations are available to compute cluster
  purity for choosing the cut
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: Mpath
  role: Algorithm whose initial clustering stage designates landmark clusters via
    iterative dendrogram cutting and purity-based cut selection
  repo: JinmiaoChenLab/Mpath
- name: R
  role: 'Implementation environment: hclust with Ward.D linkage and cutree for iteratively
    cutting the dendrogram'
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/ncomms11988
    title: Mpath maps multi-branching single-cell trajectories revealing progenitor
      cell progression during development
  source_content_hash: sha256:016ec19848e4d3063b7d565c6514d81d8453c28abfeaacea8a712dcd79a1fc57
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JinmiaoChenLab/Mpath
  tools:
  - Mpath
  - R
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# dendrogram-iterative-cutting

## Summary

Iteratively cut a hierarchical (agglomerative) cell clustering dendrogram over a range of candidate cluster counts to designate landmark clusters in Mpath, choosing the optimal cut where cluster purity (Shannon index of FACS-sort label composition) reaches its inflection point.

## When to use

Use when designating landmark clusters from single-cell expression data by cutting a Ward.D hclust dendrogram at multiple candidate k values and selecting the cut via a purity-versus-number-of-clusters inflection criterion.

## When NOT to use

- The development process starts from multiple independent progenitor cell types or the relevant subset information is unknown — Mpath does not yet support these cases
- No FACS-sort labels or other known subset annotations are available to compute cluster purity for choosing the cut

## Inputs

- Expression matrix over Mpath-selected genes (cells x genes)
- Per-cell FACS-sort labels used as purity reference

## Outputs

- landmark_assignments.csv (cell-to-cluster assignments for the optimal cut)
- Purity-versus-number-of-clusters curve used to pick the inflection point
- Set of designated landmark clusters (e.g., 11 clusters yielding 8 landmarks on the DC progenitor data)

## How to apply

Load the expression matrix restricted to Mpath-selected genes together with the FACS-sort label for each cell. Compute Euclidean distances between cells and build the agglomerative hierarchy with Ward.D linkage in R (hclust). Iteratively cut the dendrogram over a range of candidate cluster counts k; for each cut, quantify purity of every cluster as the Shannon index of the composition of FACS-sort labels it contains and discard candidate cluster sets failing the purity and minimum-cluster-size cutoffs. Select the optimal cluster set at the inflection point of the purity (or landmark-count) curve — on the DC progenitor data the inflexion of landmarks-versus-total-clusters occurred at 11 clusters, from which 8 landmark clusters were designated. Write the final cell-to-cluster assignments for the selected cut (landmark_assignments.csv) as the landmark designation feeding Mpath's subsequent cell state transition mapping.

## Related tools

- **Mpath** (Algorithm whose initial clustering stage designates landmark clusters via iterative dendrogram cutting and purity-based cut selection) — JinmiaoChenLab/Mpath
- **R** (Implementation environment: hclust with Ward.D linkage and cutree for iteratively cutting the dendrogram)

## Examples

```
hc <- hclust(dist(expr_matrix), method="ward.D"); for (k in 2:20) { cl <- cutree(hc, k=k); purity[k] <- mean_shannon_index(cl, facs_labels) }
```

## Evaluation signals

- The purity-versus-number-of-clusters curve shows a clear inflection point that matches the selected cut (e.g., 11 candidate clusters on the DC progenitor data)
- Every cluster in the selected cut passes the purity (Shannon index of FACS-label composition) and minimum-cluster-size cutoffs
- The number of landmark clusters designated from the optimal cut is smaller than the number of candidate clusters (8 landmarks from 11 clusters on the DC progenitor data)
- landmark_assignments.csv assigns every input cell to exactly one landmark cluster and is consistent with cutree on the hclust hierarchy at the chosen k
- Downstream Mpath neighborhood-based cell state transitions reproduce the reported multi-branching trajectories (macrophage/DC progenitors through common DC progenitor to pre-dendritic cells)

## Limitations

- Mpath depends on signature genes of differentiated cell types (end-stage contrasts) and FACS-sorted terminal subsets, so the purity-based cut selection requires labeled terminal subsets
- Mpath does not yet support development processes starting from multiple independent progenitor cell types (planned future extension)
- Mpath is not yet extended to highly heterogeneous single-cell data sets where specific subset information is unknown

## Evidence

- [abstract] Here we present Mpath, an algorithm that derives multi-branching developmental trajectories using neighborhood-based cell state transitions.: "Here we present Mpath, an algorithm that derives multi-branching developmental trajectories using neighborhood-based cell state transitions."
- [readme] Mpath: an algorithm for constructing multi-branching cell lineages from single-cell data: "Mpath: an algorithm for constructing multi-branching cell lineages from single-cell data"
- [discussion] without requiring a priori knowledge of the temporal ordering of developmental subsets, and utilizing genes only from end-stage contrasts.: "without requiring a priori knowledge of the temporal ordering of developmental subsets, and utilizing genes only from end-stage contrasts."
