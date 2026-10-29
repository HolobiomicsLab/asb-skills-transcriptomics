---
name: euclidean-minimum-spanning-tree-cell-graph-construction
description: Use when benchmarking single-cell trajectory methods on a progenitor
  dataset with a known branching event (e.g. cDC1 vs cDC2 preDC commitment) and a
  conventional tree-based baseline is needed to compare against neighborhood-based
  methods like Mpath.
when_to_use_negative:
- When the expected lineage topology is genuinely multi-branching with multiple cell
  fates from a single progenitor type — the article showed conventional MST placed
  cDC1- and cDC2-committed preDCs on the wrong branch, whereas Mpath resolved them
- When the goal is fate prediction from differentiated-cell signature genes traveling
  back in time — that requires Mpath's neighborhood-based state-transition framework,
  not a plain Euclidean MST
- When the input is already a precomputed cell-cell graph or trajectory object rather
  than a raw expression matrix
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: Mpath
  role: Reference method against which the Euclidean-MST baseline trajectory is compared
    on the identical DC progenitor input
  repo: JinmiaoChenLab/Mpath
- name: R
  role: Implementation environment for computing Euclidean distances, building the
    minimum spanning tree, and writing the comparison table
- name: SCUBA
  role: Co-benchmarked trajectory-inference method run alongside the MST baseline
    on the same matrix
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/ncomms11988
    title: Mpath maps multi-branching single-cell trajectories revealing progenitor
      cell progression during development
  source_content_hash: sha256:1cb23fba94cb99f44ddc65fcf1dcde82db9eb64f7307c075c73bce40d5487835
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JinmiaoChenLab/Mpath
  tools:
  - Mpath
  - R
  - SCUBA
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# euclidean-minimum-spanning-tree-cell-graph-construction

## Summary

Baseline trajectory-inference skill that builds a conventional minimum spanning tree over pairwise Euclidean distances between single cells, used to benchmark whether a tree-based method captures branching events such as the cDC1/cDC2-committed preDC split.

## When to use

Use when benchmarking single-cell trajectory methods on a progenitor dataset with a known branching event (e.g. cDC1 vs cDC2 preDC commitment) and a conventional tree-based baseline is needed to compare against neighborhood-based methods like Mpath.

## When NOT to use

- When the expected lineage topology is genuinely multi-branching with multiple cell fates from a single progenitor type — the article showed conventional MST placed cDC1- and cDC2-committed preDCs on the wrong branch, whereas Mpath resolved them
- When the goal is fate prediction from differentiated-cell signature genes traveling back in time — that requires Mpath's neighborhood-based state-transition framework, not a plain Euclidean MST
- When the input is already a precomputed cell-cell graph or trajectory object rather than a raw expression matrix

## Inputs

- QC-passed single-cell expression matrix (e.g. mouse DC progenitor scRNA-seq data spanning CDP to preDC stages)
- Cell/stage annotations (preDC_c, preDC_d, CDP, mature cDC1/cDC2 labels)
- Optionally, mature cDC1/cDC2 DEG signature lists (Supplementary Data 1) for interpreting branch placement

## Outputs

- Minimum spanning tree graph over cells built on the Euclidean distance matrix (n cells, n-1 edges)
- Per-method comparison table (trajectory_method_comparison.csv) recording branch detection (yes/no) and placement of preDC_c and preDC_d
- Tree visualization/plot showing which branch cDC1- and cDC2-committed preDCs fall on

## How to apply

Load the QC-passed single-cell expression matrix (e.g. the DC progenitor matrix spanning macrophage/DC progenitors, CDP and preDC stages) into R. Compute the pairwise Euclidean distance matrix between cells and construct a conventional minimum spanning tree over these distances, giving a connected graph of n cells with n-1 edges. Inspect the resulting tree topology for a branching event separating the preDC subsets and record the branch placement of preDC_c (cDC1-committed) and preDC_d (cDC2-committed) cells. Tabulate per-method outcomes (branch detected yes/no, placement of preDC_c and preDC_d) in trajectory_method_comparison.csv. Rationale: Mpath was benchmarked against this conventional MST plus SCUBA, monocle, diffusion map and PCA, and only Mpath's neighborhood-based cell state transitions faithfully recapitulated the preDC branching, which makes the MST a negative control for tree methods that cannot represent multi-branching lineages.

## Related tools

- **Mpath** (Reference method against which the Euclidean-MST baseline trajectory is compared on the identical DC progenitor input) — JinmiaoChenLab/Mpath
- **R** (Implementation environment for computing Euclidean distances, building the minimum spanning tree, and writing the comparison table)
- **SCUBA** (Co-benchmarked trajectory-inference method run alongside the MST baseline on the same matrix)

## Examples

```
mst <- igraph::mst(igraph::graph_from_adjacency_matrix(as.matrix(dist(expr, method="euclidean")), mode="undirected", weighted=TRUE))
```

## Evaluation signals

- The MST graph is connected and contains exactly n-1 edges over the QC-passed cell set
- trajectory_method_comparison.csv exists and records, for each method (SCUBA, monocle, diffusion map, PCA, MST, Mpath), whether the cDC1/cDC2 preDC branching event was detected (yes/no)
- The recorded branch placement of preDC_c and preDC_d on the MST tree matches the expected negative-control outcome (wrong-branch placement), mirroring the article's benchmark result
- The MST was run on the identical QC-passed expression matrix used for Mpath, ensuring a fair per-method comparison

## Limitations

- Conventional MST imposed the wrong branch placement: it placed cDC1- and cDC2-committed preDCs on the wrong branch in the DC progenitor benchmark
- As a single-tree method it cannot represent multi-branching single-cell developmental trajectories with multiple cell fates from a single progenitor cell type, the core capability the article attributes to Mpath
- The comparison demonstrated Mpath outperformed the MST baseline; MST results should be interpreted as a baseline, not as fate-resolved trajectories

## Evidence

- [abstract] Mpath derives multi-branching trajectories via neighborhood transitions: "Here we present Mpath, an algorithm that derives multi-branching developmental trajectories using neighborhood-based cell state transitions."
- [other] MST baseline places preDCs on wrong branch: [evidence span withheld — anchor status: not_found]
- [other] Workflow step for building the conventional MST on Euclidean distances: [evidence span withheld — anchor status: not_found]
- [abstract] Branching event at the preDC stage defines the target signal: "The Mpath-generated trajectories detect a branching event at the preDC stage revealing preDC subsets that are exclusively committed to cDC1 or cDC2 lineages."
- [discussion] Multi-branching resolution is the capability the baseline lacks: "The Mpath algorithm is able to resolve multi-branching single-cell developmental trajectories, allowing for multiple cell fates stemming from a single progenitor cell type."
