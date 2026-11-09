---
name: hierarchical-clustering-of-single-cell-transcriptomes
description: Use when you must group individual cells from a single-cell expression
  matrix into an initial set of discrete cell-state clusters prior to defining Mpath
  landmark clusters, using FACS-sort labels to judge cluster quality.
when_to_use_negative:
- The analysis goal is ordering cells along a continuous trajectory without needing
  discrete cell-state landmarks.
- No FACS-sort (or equivalent known-subset) labels are available to score cluster
  purity, so the inflection-point selection cannot be justified.
- The dataset starts from multiple independent progenitor cell types or is highly
  heterogeneous with unknown subset information — Mpath explicitly does not yet support
  these cases.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: Mpath
  role: R package whose initial stage designates landmark clusters; this hierarchical
    clustering step feeds its neighborhood-based cell state transition mapping
  repo: https://github.com/JinmiaoChenLab/Mpath
- name: R
  role: runtime for dist() and hclust() with Ward.D linkage, and for purity/inflection-point
    analysis of candidate cluster cuts
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/ncomms11988
    title: Mpath maps multi-branching single-cell trajectories revealing progenitor
      cell progression during development
  source_content_hash: sha256:d46b1f0a3b2f64effc4300c3d65c9c197f8bf95c5ce60257b150c4bd523c280b
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
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# hierarchical-clustering-of-single-cell-transcriptomes

## Summary

Agglomerative hierarchical clustering of single-cell RNA-seq expression data (Euclidean distance, Ward.D linkage in R) to pre-cluster cells before Mpath landmark designation, with cluster-set selection driven by FACS-label purity and an inflection-point criterion.

## When to use

Use when you must group individual cells from a single-cell expression matrix into an initial set of discrete cell-state clusters prior to defining Mpath landmark clusters, using FACS-sort labels to judge cluster quality.

## When NOT to use

- The analysis goal is ordering cells along a continuous trajectory without needing discrete cell-state landmarks.
- No FACS-sort (or equivalent known-subset) labels are available to score cluster purity, so the inflection-point selection cannot be justified.
- The dataset starts from multiple independent progenitor cell types or is highly heterogeneous with unknown subset information — Mpath explicitly does not yet support these cases.

## Inputs

- expression matrix over Mpath-selected genes (cells x genes)
- FACS-sort label for each cell (known sorting subset annotation)

## Outputs

- cell-to-cluster assignments for the selected dendrogram cut
- cluster purity (Shannon index) per candidate cluster count k
- final landmark designation (e.g. landmark_assignments.csv)

## How to apply

Load the expression matrix restricted to Mpath-selected genes together with the FACS-sort label of each cell. Compute Euclidean distances between cells and build an agglomerative hierarchy with Ward.D linkage using R's hclust. Iteratively cut the dendrogram over a range of candidate cluster counts k, and for each cut score the purity of every cluster as the Shannon index of its FACS-sort label composition, discarding candidate cuts that fail purity and minimum-cluster-size cutoffs. Plot cluster purity against the number of clusters and select the optimal cut at the inflection point of the purity curve. On the Mpath dendritic-cell progenitor data this procedure designated the optimal set at 11 clusters, from which 8 landmark clusters were designated. The rationale is that landmarks must represent homogeneous, discrete cell states so that subsequent neighborhood-based state-transition mapping is meaningful.

## Related tools

- **Mpath** (R package whose initial stage designates landmark clusters; this hierarchical clustering step feeds its neighborhood-based cell state transition mapping) — https://github.com/JinmiaoChenLab/Mpath
- **R** (runtime for dist() and hclust() with Ward.D linkage, and for purity/inflection-point analysis of candidate cluster cuts)

## Examples

```
hc <- hclust(dist(as.matrix(expr[, mpath_genes])), method = "ward.D"); for (k in ks) { cut <- cutree(hc, k); purity[k] <- shannon_index(cut, facs_labels) }
```

## Evaluation signals

- The dendrogram is built with Euclidean distance and Ward.D linkage (hclust), not another metric/linkage combination.
- Purity of each candidate cluster is quantified as the Shannon index of its FACS-sort label composition, and candidate cuts failing purity or minimum-cluster-size cutoffs are discarded.
- The selected cut sits at the inflection point of the cluster-purity versus number-of-clusters curve.
- On the DC progenitor benchmark, the procedure reproduces the reference result: optimal set at 11 clusters yielding 8 landmark clusters.
- The output is a complete cell-to-cluster assignment file (landmark_assignments.csv) covering every input cell exactly once.

## Limitations

- Mpath depends on signature genes of differentiated cell types (end-stage contrasts) and FACS-sorted terminal subsets, so clustering quality assessment requires known subset labels.
- Mpath does not yet support development processes starting from multiple independent progenitor cell types (planned future extension).
- Mpath is not yet extended to highly heterogeneous single-cell data sets where specific subset information is unknown.
- Demonstrated utility is limited to two data sets: mouse bone marrow-derived DC progenitors and primary human myoblasts.

## Evidence

- [abstract] Here we present Mpath, an algorithm that derives multi-branching developmental trajectories using neighborhood-based cell state transitions.: "Here we present Mpath, an algorithm that derives multi-branching developmental trajectories using neighborhood-based cell state transitions."
- [discussion] Mpath uses count of neighbouring cells to estimate potential state transition.: "Mpath uses count of neighbouring cells to estimate potential state transition. Via neighborhood-based cell state transition mapping"
- [supplementary] Mpath reference manual provides detailed documentation of Mpath R package.: "Mpath reference manual provides detailed documentation of Mpath R package."
- [readme] Mpath: an algorithm for constructing multi-branching cell lineages from single-cell data: "Mpath: an algorithm for constructing multi-branching cell lineages from single-cell data"
