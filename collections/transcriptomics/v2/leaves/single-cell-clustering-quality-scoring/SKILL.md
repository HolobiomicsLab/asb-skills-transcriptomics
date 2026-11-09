---
name: single-cell-clustering-quality-scoring
description: Use when many candidate clusterings exist (e.g. Totem's ~10k CLARA runs
  over a range of k) and an appropriate clustering must be automatically selected
  before constructing the trajectory MST.
when_to_use_negative:
- The goal is disconnected trajectories or trajectories with cycles — Totem cannot
  generate these, so scoring its clusterings is pointless
- The dataset simultaneously contains extremely rare and extremely prevalent cell
  types where k-medoids fails and manual cluster merging is needed first
- The candidate clusterings contain clusters below the min.cluster.size threshold
  (default 5 cells) — these should be filtered out before ranking
edam_operation: http://edamontology.org/operation_3432
tools:
- name: Totem
  role: Provides the cell-connectivity vector, candidate clusterings, and SelectClusterings(selection.method=1)
    ranking/selection function
  repo: https://github.com/elolab/Totem
- name: fpc
  role: Supplies calinhara, used to compute the variance ratio criterion (Calinski-Harabasz
    score) on the connectivity vector
- name: R
  role: Implementation language for the Totem scoring and selection workflow
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btad515
    title: Cell-connectivity-guided trajectory inference from single-cell data
  source_content_hash: sha256:cb915644419541e81190e55634360cff2ff8c24d073c07f29ce2eda01142fe02
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/elolab/Totem
  tools:
  - R
  - Totem
  - fpc
  edam_operation: http://edamontology.org/operation_3432
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# single-cell-clustering-quality-scoring

## Summary

Scores and ranks candidate single-cell clusterings by the variance ratio criterion (Calinski-Harabasz score) computed on Totem's one-dimensional cell-connectivity vector, enabling automatic selection of the clustering used to build the trajectory MST.

## When to use

Use when many candidate clusterings exist (e.g. Totem's ~10k CLARA runs over a range of k) and an appropriate clustering must be automatically selected before constructing the trajectory MST.

## When NOT to use

- The goal is disconnected trajectories or trajectories with cycles — Totem cannot generate these, so scoring its clusterings is pointless
- The dataset simultaneously contains extremely rare and extremely prevalent cell types where k-medoids fails and manual cluster merging is needed first
- The candidate clusterings contain clusters below the min.cluster.size threshold (default 5 cells) — these should be filtered out before ranking

## Inputs

- Totem object containing candidate clusterings (e.g. from ~10k CLARA k-medoids runs over a range of k values)
- One-dimensional cell-connectivity vector (per-cell connectivity estimated from MSTs built per clustering)
- Optional prior clustering used to bias ranking via ARI similarity

## Outputs

- selected_clusterings.rds — non-empty top-ranked selection of clusterings ordered by Calinski-Harabasz score and cell-connectivity ranking
- Ranking of candidate clusterings by the combined Calinski-Harabasz/connectivity criterion

## How to apply

Load the Totem object holding the candidate clusterings and extract the one-dimensional cell-connectivity vector per cell — critically, this vector, not the whole dimensionally reduced embedding, is the data representation fed to the criterion. For each candidate clustering compute the variance ratio criterion (Calinski-Harabasz score, fpc::calinhara) with the connectivity vector as the data. Rank all clusterings by the Calinski-Harabasz score combined with cell connectivity, optionally incorporating ARI to a user-supplied prior clustering to steer the search, and retrieve the top-ranked set with SelectClusterings(selection.method=1). Save the selected clustering(s) and their ranking as selected_clusterings.rds for downstream MST construction with Slingshot. The rationale is that clusterings whose cells show similar connectivity produce biologically meaningful tree backbones with correct milestone connections.

## Related tools

- **Totem** (Provides the cell-connectivity vector, candidate clusterings, and SelectClusterings(selection.method=1) ranking/selection function) — https://github.com/elolab/Totem
- **fpc** (Supplies calinhara, used to compute the variance ratio criterion (Calinski-Harabasz score) on the connectivity vector)
- **R** (Implementation language for the Totem scoring and selection workflow)

## Examples

```
top <- SelectClusterings(totem_obj, selection.method = 1)  # ranks clusterings by Calinski-Harabasz score on the cell-connectivity vector; save as selected_clusterings.rds
```

## Evaluation signals

- The selected_clusterings.rds output contains a non-empty, ordered top-ranked set of clusterings
- The Calinski-Harabasz score was computed with the one-dimensional connectivity vector as data representation, not the raw low-dimensional embedding
- Ranking reflects the combined Calinski-Harabasz score and cell connectivity (plus ARI to prior clustering, if one was supplied)
- Clusterings violating min.cluster.size (default 5 cells per cluster) were excluded before ranking

## Limitations

- k-medoids may not work well with PCA if the number of principal components is high (e.g. 30), producing 1-2 cell clusters that skew the ranking
- k-medoids can fail on extremely imbalanced datasets (rare plus prevalent cell types), requiring manual cluster merging before selection
- Providing a prior clustering biases selection toward clusterings similar to the prior, which may not reflect an independent optimum

## Evidence

- [intro] Rank clustering results using the Calinski-Harabasz score and cell connectivity: "`Totem` uses the `Calinski-Harabasz score` and the cell connectivity to rank the clustering results."
- [intro] Trajectory success depends on good clustering: "much of the success of the trajectory inference depends on finding a good clustering that produces a biologically meaningful trajectory with the correct milestone connections."
- [intro] Clusterings feed MST construction with Slingshot: "After clustering we need to select one or several clusterings that are used to construct the Minimum Spanning Trees (MSTs) with Slingshot."
- [methods] VRC computed on connectivity vector from source card workflow: [evidence span withheld — anchor status: not_found]
- [readme] README description of Totem's purpose: [evidence span withheld — anchor status: external_doc]
