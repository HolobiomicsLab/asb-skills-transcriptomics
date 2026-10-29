---
name: k-medoids-clustering-of-pathway-curves
description: Use when a set of coordinated pathways has per-cell PAL scores along
  a trajectory (pseudotime) and you want to discover shared stage-activity patterns,
  e.g. developmental stage-specific pathway modules.
when_to_use_negative:
- Gene sets have not passed the coordination test (compute.kld adjusted p >= 0.05)
  — noise curves would dominate clusters
- Data is not ordered along a trajectory (no pseudotime), e.g. dissociated static
  samples — curve fitting is meaningless without an ordering
- Spatial transcriptomics data where the goal is spatial relevance testing (compute.spatial.kld)
  rather than trajectory-stage patterns
edam_operation: http://edamontology.org/operation_3454
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: GSDensity
  role: Computes PALs via random walk with restart (run.rwr.list) on the cell-gene
    graph; PALs are averaged in pseudotime bins to form the curves being clustered
  repo: https://github.com/qingnanl/gsdensity
- name: cluster (pam)
  role: Provides the k-medoids algorithm (pam) used to partition the fitted pathway
    curves into 8 groups
- name: monocle3
  role: Supplies the pseudotime ordering used to define the 20 equal-width bins along
    which PALs are averaged
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-023-44206-x
    title: Pathway centric analysis for single-cell RNA-seq and spatial transcriptomics
      data with GSDensity
  source_content_hash: sha256:632528f0948c7cf71f5975adb87532fbd06fb5b949ef2674c0ee11a36fe20df1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/qingnanl/gsdensity
  tools:
  - GSDensity
  - RANN
  - Seurat
  - cluster (pam)
  - cluster (pam, k-medoids)
  - igraph
  - monocle3
  - msigdbr
  edam_operation: http://edamontology.org/operation_3454
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 16
---

# k-medoids-clustering-of-pathway-curves

## Summary

Groups fitted pathway activity level (PAL) curves along pseudotime into stage-pattern clusters using k-medoids (pam), as done to organize 97 KEGG/BIOCARTA pathways active across E17.5 mouse cortex development into 8 clusters in the GSDensity mouse brain development analysis (Fig. 4).

## When to use

Use when a set of coordinated pathways has per-cell PAL scores along a trajectory (pseudotime) and you want to discover shared stage-activity patterns, e.g. developmental stage-specific pathway modules.

## When NOT to use

- Gene sets have not passed the coordination test (compute.kld adjusted p >= 0.05) — noise curves would dominate clusters
- Data is not ordered along a trajectory (no pseudotime), e.g. dissociated static samples — curve fitting is meaningless without an ordering
- Spatial transcriptomics data where the goal is spatial relevance testing (compute.spatial.kld) rather than trajectory-stage patterns

## Inputs

- Per-cell PAL scores for ~97 significantly coordinated KEGG/BIOCARTA gene sets (adjusted p < 0.05 from compute.kld)
- Monocle3 pseudotime ordering per cell for E17.5 mouse cortex scRNA-seq (GSE153162)
- Number of clusters k (e.g. 8) and number of pseudotime bins (e.g. 20)

## Outputs

- A k-medoids (pam) partition assigning each pathway curve to one of 8 stage-pattern clusters
- Fitted PAL-vs-pseudotime curves (per pathway, averaged over 20 equal-width pseudotime bins)
- Cluster composition/membership table comparable to the published figure

## How to apply

First compute per-cell PALs for the significantly coordinated gene sets with GSDensity's run.rwr.list (restart = 0.75) on a cell-gene nearest-neighbor graph built with compute.nn.edges (nn.use = 300). Partition cells into 20 equal-width pseudotime bins (ordering from monocle3 on the GSE153162 E17.5 mouse brain data) and average PAL per pathway per bin to obtain a fitted PAL-vs-pseudotime curve for each pathway. Apply k-medoids clustering (cluster::pam) with k = 8 to the 97 fitted curves that passed the coordination test (compute.kld, n.grids = 100, n.times = 100 permutations, adjusted p-value < 0.05), yielding stage-pattern groups analogous to Fig. 4. The rationale is that averaging within pseudotime bins denoises single-cell PALs so that curve shape — not cell-level noise — drives cluster assignment.

## Related tools

- **GSDensity** (Computes PALs via random walk with restart (run.rwr.list) on the cell-gene graph; PALs are averaged in pseudotime bins to form the curves being clustered) — https://github.com/qingnanl/gsdensity
- **cluster (pam)** (Provides the k-medoids algorithm (pam) used to partition the fitted pathway curves into 8 groups)
- **monocle3** (Supplies the pseudotime ordering used to define the 20 equal-width bins along which PALs are averaged)

## Examples

```
pam_result <- pam(t(fitted_pal_curves[, sig_pathways]), k = 8); cluster_assignments <- pam_result$clustering
```

## Evaluation signals

- The coordination test preceding clustering must retain exactly the 97 KEGG/BIOCARTA pathways with adjusted p-value < 0.05
- The 8-cluster k-medoids partition must visually match the cluster composition of the published Fig. 4
- Each fitted curve should consist of 20 averaged PAL values (one per equal-width pseudotime bin)
- Cluster silhouette/compactness should indicate coherent within-cluster PAL curve shapes across pseudotime bins

## Limitations

- Neighborhood size (nn.use = 300) is an important parameter when constructing the cell-gene graph feeding PAL computation
- GSDensity is not suitable for targeted gene data with limited numbers of genes; imputation of unmeasured genes is suggested but not implemented
- MCA embedding step requires large memory (~25 Gb for a 40,000-cell dataset)

## Evidence

- [abstract] Combined with trajectory analysis, GSDensity identifies curated pathways active at various stages of mouse brain development: "GSDensity, combined with trajectory analysis can identify curated pathways that are active at various stages of mouse brain development"
- [intro] PAL computed via network propagation with pathway genes as seeds: "Pathway activity for each cell can be calculated using network propagation in a nearest-neighbor cell-gene graph, with pathway genes used as seeds for random walks"
- [methods] Workflow step: build graph and run RWR with given parameters: "el <- compute.nn.edges(coembed = ce, nn.use = 300)"
- [results] Trajectory workflow description from source card: [evidence span withheld — anchor status: not_found]
- [discussion] Memory cost of upstream MCA embedding: "The largest memory cost was in the MCA embedding step, with about 25 Gb for a medium sized dataset"
- [readme] README quick-start showing compute.kld coordination test preceding downstream analysis: [evidence span withheld — anchor status: not_found]
