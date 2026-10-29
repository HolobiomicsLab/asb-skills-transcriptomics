---
name: clara-kmedoids-mst-topology-inference
description: Use when inferring a tree-shaped trajectory (linear, bifurcation, multifurcation,
  tree) from single-cell data in a low-dimensional embedding, and clustering quality
  (not manual tuning) must determine the milestone topology.
when_to_use_negative:
- The data contain disconnected trajectories or trajectories with cycles — Totem cannot
  generate these.
- Multi-root (converging) trajectories requiring correct pseudotime — dynplot will
  not generate correct pseudotime and a manual work-around is needed.
- Datasets with extremely rare and extremely prevalent cell types simultaneously,
  where k-medoids may fail and manual cluster merging is required.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Totem
  role: Orchestrates repeated CLARA clustering, MST construction, filtering, and connectivity-based
    clustering selection
  repo: https://github.com/elolab/Totem
- name: Slingshot
  role: Constructs the Minimum Spanning Tree backbone and linear curves through the
    lineages of the tree for each clustering
- name: dyndimred
  role: Provides LMDS dimensionality reduction producing the 5-dimensional embedding
    used as clustering input
- name: SingleCellExperiment
  role: Input container prepared for Totem analysis
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad515
    title: Cell-connectivity-guided trajectory inference from single-cell data
  source_content_hash: sha256:cf09a5b80ae3a83d57d59160168b652612cae88420b9e80d27684139f4173b5b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/elolab/Totem
  tools:
  - BiocManager
  - R
  - SingleCellExperiment
  - Slingshot
  - TinGa
  - Totem
  - dyndimred
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# clara-kmedoids-mst-topology-inference

## Summary

Infer tree-shaped (linear, bifurcating, multifurcating) single-cell trajectories by generating many CLARA k-medoids clusterings, building a Slingshot MST per clustering, and selecting clusterings via the Calinski-Harabasz score and cell connectivity (the Totem approach).

## When to use

Use when inferring a tree-shaped trajectory (linear, bifurcation, multifurcation, tree) from single-cell data in a low-dimensional embedding, and clustering quality (not manual tuning) must determine the milestone topology.

## When NOT to use

- The data contain disconnected trajectories or trajectories with cycles — Totem cannot generate these.
- Multi-root (converging) trajectories requiring correct pseudotime — dynplot will not generate correct pseudotime and a manual work-around is needed.
- Datasets with extremely rare and extremely prevalent cell types simultaneously, where k-medoids may fail and manual cluster merging is required.

## Inputs

- SingleCellExperiment object with single-cell expression data (e.g. dynverse synthetic binary_tree_1 or gold-standard trajectories from Zenodo doi:10.5281/zenodo.1443566)
- Low-dimensional cell embedding (e.g. 5-dimensional LMDS from dyndimred)
- Optional prior clustering to guide the clustering search

## Outputs

- A tree-shaped inferred trajectory (milestone network with leaves and branch points) from the selected clustering
- Ranked clustering candidates with Calinski-Harabasz scores and cell-connectivity values
- Cell-connectivity visualization identifying likely end points and branching points

## How to apply

First prepare a SingleCellExperiment and reduce the data to a low-dimensional embedding (e.g. 5-dimensional LMDS via dyndimred; avoid PCA with high component counts). Run the k-medoids CLARA algorithm a large number of times (default 10k runs) over a range of k values, filtering out clusterings whose smallest cluster falls below min.cluster.size. Construct a Minimum Spanning Tree with Slingshot for each retained clustering, then rank clusterings with the Calinski-Harabasz score and the cell-connectivity measure, which highlights likely leaves (end points) and branch points; users may also supply a prior clustering and rank by ARI to it. The rationale is that trajectory inference succeeds largely by finding a clustering that yields biologically meaningful milestone connections, so many clusterings plus automated ranking removes the need for manual parameter tuning.

## Related tools

- **Totem** (Orchestrates repeated CLARA clustering, MST construction, filtering, and connectivity-based clustering selection) — https://github.com/elolab/Totem
- **Slingshot** (Constructs the Minimum Spanning Tree backbone and linear curves through the lineages of the tree for each clustering)
- **dyndimred** (Provides LMDS dimensionality reduction producing the 5-dimensional embedding used as clustering input)
- **SingleCellExperiment** (Input container prepared for Totem analysis)

## Examples

```
remotes::install_github("elolab/Totem")
```

## Evaluation signals

- The number of CLARA clustering runs matches the configured value (default 10k) before MST construction.
- Clusterings with clusters smaller than min.cluster.size are excluded from downstream MST construction.
- The selected clustering maximizes the ranking criteria (Calinski-Harabasz score and cell connectivity), and the inferred topology has correct milestone connections.
- Cell-connectivity values flag leaves (low connectivity end points) and branch points consistent with the embedding.
- In the dynverse benchmark, the geometric-mean overall score of the four metrics (correlation of geodesic distances, featureimp correlation, F1 branches, HIM) can be compared against Slingshot/TinGa baselines per topology.

## Limitations

- k-medoids is unstable on high-dimensional PCA input (e.g. 30 principal components), producing 1–2 cell clusters.
- k-medoids can fail on extremely imbalanced datasets combining very rare and very prevalent cell types, requiring manual cluster merging.
- No support for disconnected trajectories or trajectories with cycles; no correct pseudotime for multi-root (converging) trajectories.
- LMDS is used over full MDS because it is faster for large datasets; 3-dimensional visualization embeddings are not supported.

## Evidence

- [intro] Totem generates a large number of clustering results with a *k*-medoids algorithm (CLARA) and constructs an MST for each clustering.: "Totem generates a large number of clustering results with a *k*-medoids algorithm (CLARA) and constructs an MST for each clustering."
- [intro] Totem uses the Calinski-Harabasz score and the cell connectivity to rank the clustering results.: "`Totem` uses the `Calinski-Harabasz score` and the cell connectivity to rank the clustering results."
- [intro] the next step is to perform clustering that involves running the k-medoids (CLARA) algorithm a large number of times (default 10k).: "the next step is to perform clustering that involves running the k-medoids (CLARA) algorithm a large number of times (default 10k)."
- [intro] The second parameter (min.cluster.size) determines the clusterings that are filtered out because they include too small clusters.: "The second parameter (`min.cluster.size`) determines the clusterings that are filtered out because they include too small clusters."
- [discussion] the k-medoids algorithm may not work well with PCA if the number principal components is high (e.g. 30): "the k-medoids algorithm may not work well with PCA if the number principal components is high (e.g. 30)"
- [readme] Totem uses Slingshot to find linear curves through the lineages of the tree.: [evidence span withheld — anchor status: external_doc]
