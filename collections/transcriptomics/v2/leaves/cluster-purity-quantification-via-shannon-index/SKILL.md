---
name: cluster-purity-quantification-via-shannon-index
description: Use when labeled single-cell data (e.g. FACS-sorted subsets) must be
  partitioned into landmarks or reference clusters, and the number of clusters k must
  be chosen objectively from a purity-versus-k curve.
when_to_use_negative:
- The input data lacks FACS-sorted terminal subsets or signature-gene end-stage contrasts,
  since purity scoring and Mpath fate prediction depend on these labels
- The single-cell dataset is highly heterogeneous and specific subset information
  is not known — the article states this extension is future work
- Cells come from multiple independent progenitor cell types, which Mpath does not
  yet support
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0085
tools:
- name: Mpath
  role: 'R package implementing landmark designation: hierarchical clustering of single-cell
    expression data, purity-based selection of the optimal cluster cut, and writing
    of landmark assignments'
  repo: JinmiaoChenLab/Mpath
- name: R
  role: Environment in which the Mpath package runs; hclust with Ward.D linkage performs
    the agglomerative clustering that is cut at candidate k values
  repo: JinmiaoChenLab/Mpath
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/ncomms11988
    title: Mpath maps multi-branching single-cell trajectories revealing progenitor
      cell progression during development
  source_content_hash: sha256:d3b2a2dd03230256ff2e8c5c05db9148109f0948d01441804e7e0bf8d9081278
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JinmiaoChenLab/Mpath
  tools:
  - Mpath
  - R
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0085
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# Cluster purity quantification via Shannon index

## Summary

A model-selection skill from Mpath that scores candidate agglomerative cluster cuts by the Shannon index of FACS-sort-label composition per cluster, then selects the optimal cluster set at the inflection point of the purity curve to designate landmark clusters.

## When to use

Use when labeled single-cell data (e.g. FACS-sorted subsets) must be partitioned into landmarks or reference clusters, and the number of clusters k must be chosen objectively from a purity-versus-k curve.

## When NOT to use

- The input data lacks FACS-sorted terminal subsets or signature-gene end-stage contrasts, since purity scoring and Mpath fate prediction depend on these labels
- The single-cell dataset is highly heterogeneous and specific subset information is not known — the article states this extension is future work
- Cells come from multiple independent progenitor cell types, which Mpath does not yet support

## Inputs

- Expression matrix restricted to Mpath-selected genes (cells x genes)
- FACS-sort labels for each cell (terminal subset membership used as external purity reference)
- Range of candidate cluster counts k for dendrogram cuts

## Outputs

- landmark_assignments.csv (final cell-to-cluster assignments for the selected cut)
- Cluster purity scores (Shannon index per cluster) for each candidate k
- Purity-versus-number-of-clusters plot used to locate the inflection point

## How to apply

Compute Euclidean distances between cells over the Mpath-selected genes and build an agglomerative hierarchy with Ward.D linkage using R's hclust. Iteratively cut the dendrogram over a range of candidate cluster counts k. For each cut, quantify purity of every cluster as the Shannon index of the composition of FACS-sort labels it contains, and discard candidate cluster sets failing purity and minimum-cluster-size cutoffs. Plot cluster purity against the number of clusters and select the optimal set at the inflection point: on the DC progenitor data this occurred at 11 clusters, from which 8 landmark clusters were designated. Rationale: landmarks should each correspond to a discrete, label-pure cell state so that subsequent neighborhood-based state-transition mapping reflects genuine developmental transitions.

## Related tools

- **Mpath** (R package implementing landmark designation: hierarchical clustering of single-cell expression data, purity-based selection of the optimal cluster cut, and writing of landmark assignments) — JinmiaoChenLab/Mpath
- **R** (Environment in which the Mpath package runs; hclust with Ward.D linkage performs the agglomerative clustering that is cut at candidate k values) — JinmiaoChenLab/Mpath

## Examples

```
hc <- hclust(dist(expr_matrix), method='ward.D'); assignments <- cutree(hc, k = 11)
```

## Evaluation signals

- The chosen k sits at the inflection point of the purity-versus-number-of-clusters curve (e.g. k=11 for the DC progenitor data, yielding 8 landmark clusters)
- Every retained cluster in the selected cut passes the purity and minimum-cluster-size cutoffs before landmarks are designated
- landmark_assignments.csv exists and partitions all cells exactly once (assignments sum to the number of input cells)
- Cluster hierarchy reproduces when hclust is re-run with Euclidean distances and Ward.D linkage on the Mpath-selected genes
- Each landmark's Shannon index of FACS-sort-label composition is at or near the minimum (labels nearly homogeneous) for clusters passing cutoffs

## Limitations

- Mpath is not yet extended to highly heterogeneous single-cell data sets in which specific subset information is not known (planned future extension)
- Mpath depends on signature genes of differentiated cell types (end-stage contrasts) and FACS-sorted terminal subsets for fate prediction
- Mpath does not yet support development processes starting from multiple independent progenitor cell types (planned future extension)
- Demonstrated utility is limited to two data sets: mouse bone marrow-derived DC progenitors and primary human myoblasts

## Evidence

- [abstract] Mpath, an algorithm that derives multi-branching developmental trajectories using neighborhood-based cell state transitions: "Here we present Mpath, an algorithm that derives multi-branching developmental trajectories using neighborhood-based cell state transitions."
- [methods] on the DC progenitor data this occurred at 11 clusters, from which 8 landmark clusters were designated: [evidence span withheld — anchor status: not_found]
- [supplementary] Mpath reference manual provides detailed documentation of Mpath R package: "Mpath reference manual provides detailed documentation of Mpath R package."
- [readme] Mpath: an algorithm for constructing multi-branching cell lineages from single-cell data: "Mpath: an algorithm for constructing multi-branching cell lineages from single-cell data"
