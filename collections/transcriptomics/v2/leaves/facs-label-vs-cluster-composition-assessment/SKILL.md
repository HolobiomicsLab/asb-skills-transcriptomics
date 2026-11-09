---
name: facs-label-vs-cluster-composition-assessment
description: Use when single-cell expression data have accompanying FACS-sort labels
  for each cell and an unsupervised clustering must be validated/optimized by how
  purely each cluster recapitulates the sorted subsets.
when_to_use_negative:
- No FACS-sorted subset labels are available for the cells, so label-composition purity
  cannot be computed (Mpath relies on FACS-sorted terminal subsets).
- The dataset is highly heterogeneous with no known subset information; the article
  states Mpath is not yet extended to such data.
- The analysis goal is trajectory construction rather than choosing the cluster set;
  this skill only covers the purity-based landmark designation step.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: Mpath
  role: Algorithm and R package whose landmark-designation stage performs the Ward.D
    hierarchical clustering and FACS-label purity scoring of dendrogram cuts
  repo: https://github.com/JinmiaoChenLab/Mpath
- name: R
  role: Environment in which hclust (Ward.D), dendrogram cutting, and Shannon-index
    purity computation are executed
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/ncomms11988
    title: Mpath maps multi-branching single-cell trajectories revealing progenitor
      cell progression during development
  source_content_hash: sha256:ffc20edb3c0efe6f15bfaac162d2afb55d2db20ee7207bf4d624c14d237bc290
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
yes_rate: 0.6666666666666666
n_claims: 6
---

# facs-label-vs-cluster-composition-assessment

## Summary

A cluster-quality assessment skill from Mpath in which candidate dendrogram cuts are scored by the Shannon-index purity of FACS-sort label composition per cluster, so the optimal number of landmark clusters is chosen at the inflection of the purity-versus-k curve.

## When to use

Use when single-cell expression data have accompanying FACS-sort labels for each cell and an unsupervised clustering must be validated/optimized by how purely each cluster recapitulates the sorted subsets.

## When NOT to use

- No FACS-sorted subset labels are available for the cells, so label-composition purity cannot be computed (Mpath relies on FACS-sorted terminal subsets).
- The dataset is highly heterogeneous with no known subset information; the article states Mpath is not yet extended to such data.
- The analysis goal is trajectory construction rather than choosing the cluster set; this skill only covers the purity-based landmark designation step.

## Inputs

- Expression matrix restricted to Mpath-selected genes
- Per-cell FACS-sort labels for each cell
- Candidate range of dendrogram cut heights / cluster counts k

## Outputs

- Per-cluster Shannon-index purity scores per candidate cut
- Purity-versus-number-of-clusters plot with inflection point
- landmark_assignments.csv with cell-to-cluster assignments for the selected cut

## How to apply

Load the expression matrix over Mpath-selected genes together with the FACS-sort label of each cell, compute Euclidean distances, and build an agglomerative hierarchy with Ward.D linkage via R's hclust. Iteratively cut the dendrogram over a range of candidate cluster counts k. For each candidate cut, quantify every cluster's purity as the Shannon index of the composition of FACS-sort labels it contains, and discard candidate cluster sets failing the purity and minimum-cluster-size cutoffs. Plot cluster purity against the number of clusters and select the optimal set at the inflection point; on the DC progenitor data this occurred at 11 clusters, from which 8 landmark clusters were designated. Write the final cell-to-cluster assignments of the selected cut to landmark_assignments.csv.

## Related tools

- **Mpath** (Algorithm and R package whose landmark-designation stage performs the Ward.D hierarchical clustering and FACS-label purity scoring of dendrogram cuts) — https://github.com/JinmiaoChenLab/Mpath
- **R** (Environment in which hclust (Ward.D), dendrogram cutting, and Shannon-index purity computation are executed)

## Examples

```
hc <- hclust(dist(expr[mpath_genes, ]), method="ward.D"); assign <- cutree(hc, k=8)  # k chosen at inflection of Shannon-purity curve; write landmark_assignments.csv
```

## Evaluation signals

- Every cluster in a retained candidate cut has purity quantified as the Shannon index of its FACS-sort label composition, and candidate sets failing purity or minimum-cluster-size cutoffs are discarded.
- The selected cut sits at the inflection point of the purity-versus-number-of-clusters curve.
- On the mouse DC progenitor reference data, the procedure reproduces the reported result: optimal cut at 11 clusters yielding 8 landmark clusters.
- The output file landmark_assignments.csv contains exactly one cluster assignment per input cell.
- Euclidean distances and Ward.D linkage are used, matching the article's specified hierarchy construction.

## Limitations

- Depends on FACS-sorted subset labels being available; without prior sorted subsets, label-purity cannot be assessed.
- Mpath does not yet support development processes starting from multiple independent progenitor cell types.
- Demonstrated utility is limited to two data sets: mouse bone marrow-derived DC progenitors and primary human myoblasts.

## Evidence

- [results] Mpath designates landmarks by first clustering cells and then selecting the optimal cluster set at the inflexion point of the number of landmarks versus the total number of clusters: [evidence span withheld — anchor status: not_found]
- [methods] quantify purity of every cluster as the Shannon index of the composition of FACS-sort labels it contains: [evidence span withheld — anchor status: not_found]
- [discussion] Mpath is able to predict individual cell's fate using signature genes of differentiated cell types.: "Mpath is able to predict individual cell's fate using signature genes of differentiated cell types."
- [supplementary] Mpath reference manual provides detailed documentation of Mpath R package.: "Mpath reference manual provides detailed documentation of Mpath R package."
