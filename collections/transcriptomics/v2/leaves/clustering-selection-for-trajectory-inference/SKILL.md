---
name: clustering-selection-for-trajectory-inference
description: Use when a Totem-style trajectory pipeline has produced thousands of
  candidate clusterings and you must automatically pick which clustering(s) to feed
  into Slingshot MST construction, rather than hand-picking a biologically sensible
  model.
when_to_use_negative:
- The input is a raw single-cell expression matrix or low-dimensional embedding without
  prior clustering generation — clustering selection presupposes a pool of candidate
  clusterings and a computed cell-connectivity vector.
- The data contain a disconnected trajectory or trajectories with cycles, which Totem
  cannot generate.
- The dataset mixes extremely rare and extremely prevalent cell types, where k-medoids
  fails and the connectivity-based ranking would be unreliable.
edam_operation: http://edamontology.org/operation_3465
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: Totem
  role: 'Provides the clustering-selection machinery: ranks candidate clusterings
    by Calinski-Harabasz score and cell connectivity and retrieves the top-ranked
    set via SelectClusterings(selection.method=1).'
  repo: https://github.com/elolab/Totem
- name: R
  role: Implementation environment; fpc::calinhara computes the variance ratio criterion
    on the connectivity vector.
- name: Slingshot
  role: Consumes the selected clustering to construct the MST backbone of the tree-shaped
    trajectory.
provenance:
  source_task_ids:
  - task_008
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad515
    title: Cell-connectivity-guided trajectory inference from single-cell data
  source_content_hash: sha256:f5da21e2f126c9cd312c9e608c6b32944d20f9c87380d0428b8e83c2442cd0f8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/elolab/Totem
  tools:
  - R
  - SingleCellExperiment
  - Slingshot
  - Totem
  - dyndimred
  edam_operation: http://edamontology.org/operation_3465
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# clustering-selection-for-trajectory-inference

## Summary

Ranks a large pool of candidate CLARA k-medoids clusterings using the Calinski-Harabasz variance ratio criterion computed on the one-dimensional cell-connectivity vector, then selects the top-ranked clustering(s) whose cells have similar connectivity for constructing the MST of a tree-shaped trajectory in Totem.

## When to use

Use when a Totem-style trajectory pipeline has produced thousands of candidate clusterings and you must automatically pick which clustering(s) to feed into Slingshot MST construction, rather than hand-picking a biologically sensible model.

## When NOT to use

- The input is a raw single-cell expression matrix or low-dimensional embedding without prior clustering generation — clustering selection presupposes a pool of candidate clusterings and a computed cell-connectivity vector.
- The data contain a disconnected trajectory or trajectories with cycles, which Totem cannot generate.
- The dataset mixes extremely rare and extremely prevalent cell types, where k-medoids fails and the connectivity-based ranking would be unreliable.

## Inputs

- Totem object containing the candidate CLARA clustering results (clustering_results.rds)
- One-dimensional cell-connectivity vector computed from the MSTs of the candidate clusterings
- Optional: user-provided prior clustering used to bias selection via ARI

## Outputs

- selected_clusterings.rds containing the top-ranked clustering(s) and their ranking by Calinski-Harabasz score and cell connectivity
- Selected clustering(s) used to construct Minimum Spanning Trees with Slingshot

## How to apply

After the CLARA stage (default 10,000 runs, k in 3–20, min.cluster.size = 5 filter), extract the one-dimensional cell-connectivity vector — not the raw low-dimensional LMDS embedding — as the data representation. For each candidate clustering, compute the variance ratio criterion (Calinski-Harabasz score, fpc::calinhara) using that connectivity vector, since the goal is to find clusters whose cells have similar connectivity. Rank clusterings by the Calinski-Harabasz score combined with cell connectivity (optionally also ARI to a user-provided prior clustering) and retrieve the top-ranked set with SelectClusterings(selection.method=1). Save the selected clustering(s) and their ranking as selected_clusterings.rds; the selected clustering is then used with Slingshot to construct the trajectory MST. The rationale is that cell connectivity marks likely leaves and branch points, so clusterings consistent with the connectivity structure yield correct milestone connections.

## Related tools

- **Totem** (Provides the clustering-selection machinery: ranks candidate clusterings by Calinski-Harabasz score and cell connectivity and retrieves the top-ranked set via SelectClusterings(selection.method=1).) — https://github.com/elolab/Totem
- **R** (Implementation environment; fpc::calinhara computes the variance ratio criterion on the connectivity vector.)
- **Slingshot** (Consumes the selected clustering to construct the MST backbone of the tree-shaped trajectory.)

## Examples

```
vrc <- fpc::calinhara(connectivity_vector, clustering_labels); selected <- SelectClusterings(totem_obj, selection.method = 1)
```

## Evaluation signals

- selected_clusterings.rds contains a non-empty top-ranked selection of clusterings ordered by the Calinski-Harabasz score and cell-connectivity ranking.
- The data representation passed to the Calinski-Harabasz computation is the one-dimensional cell-connectivity vector, not the raw 5-dimensional LMDS embedding.
- Every selected clustering has already passed the min.cluster.size = 5 filter (no clusters with fewer than five cells).
- Downstream Slingshot MST construction succeeds and the resulting trajectory's branch points and leaves agree with high-connectivity regions of the embedding.

## Limitations

- Default parameters of trajectory inference methods are often suboptimal and their tuning requires methodological expertise; Totem mitigates this by ranking, but the connectivity-based ranking still depends on the quality of the generated clusterings.
- k-medoids may not work well with high-dimensional PCA input (e.g. 30 principal components), producing 1–2 cell clusters, which would corrupt the ranking stage.
- If the dataset includes extremely rare cell types and extremely prevalent cell types at the same time, the k-medoids might not work properly and manual cluster merging is needed before selection.
- Totem cannot generate disconnected trajectories or trajectories that have cycles, so no clustering selection can recover such topologies.

## Evidence

- [intro] Rank clustering results using the Calinski-Harabasz score and cell connectivity: "`Totem` uses the `Calinski-Harabasz score` and the cell connectivity to rank the clustering results."
- [intro] Clustering generation with CLARA and MST estimation: "`Totem` generates a large number of clustering results with a *k*-medoids algorithm (CLARA) and constructs an MST for each clustering."
- [abstract] Cell connectivity enables automatic selection and visualization of topology: "Besides automatic selection of an appropriate trajectory, cell connectivity enables to visually pinpoint branching points and milestones relevant to the trajectory."
- [intro] Selection precedes Slingshot MST construction: "After clustering we need to select one or several clusterings that are used to construct the Minimum Spanning Trees (MSTs) with Slingshot."
- [intro] Prior clustering option: "The user can also provide `a prior clustering` to direct the search towards clusterings that are more similar with the prior clustering."
