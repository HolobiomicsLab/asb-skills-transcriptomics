---
name: reproducible-random-seed-sampling
description: Use when a trajectory-inference or ensemble-clustering step requires
  many randomly initialized clusterings on a low-dimensional embedding and results
  must be bit-reproducible across runs via seeded RNG.
when_to_use_negative:
- Dataset with extremely rare and extremely prevalent cell types at the same time,
  where k-medoids may not work properly and manual cluster merging is needed
- Embedding is a high-dimensional PCA (e.g. 30 principal components), where k-medoids
  is unstable and produces 1–2 cell clusters rather than meaningful partitions
- Data contain a trajectory with cycles or disconnected components, which Totem cannot
  model regardless of the clustering ensemble
edam_operation: http://edamontology.org/operation_3432
tools:
- name: Totem
  role: Provides RunClustering/RunCLARA with runs, k-range, min.cluster.size, and
    RNG seed handling for the clustering ensemble
  repo: https://github.com/elolab/Totem
- name: dyndimred
  role: Generates the 5-dimensional LMDS embedding on which the seeded CLARA sampling
    is run
- name: SingleCellExperiment
  role: Container class for the input data and the embedding passed to RunClustering
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad515
    title: Cell-connectivity-guided trajectory inference from single-cell data
  source_content_hash: sha256:06c541cf28619a0f7404a95eaf3d8a04c78782a4cf5bf41526fd9afb80082201
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/elolab/Totem
  tools:
  - R
  - SingleCellExperiment
  - Totem
  - dyndimred
  edam_operation: http://edamontology.org/operation_3432
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5882352941176471
n_claims: 17
---

# reproducible-random-seed-sampling

## Summary

Generate a large ensemble of random clusterings by running CLARA k-medoids many times (default 10,000) over a sampled range of k with RNG seeds enabled, so the stochastic sampling underlying Totem's cell-connectivity MST search is reproducible.

## When to use

Use when a trajectory-inference or ensemble-clustering step requires many randomly initialized clusterings on a low-dimensional embedding and results must be bit-reproducible across runs via seeded RNG.

## When NOT to use

- Dataset with extremely rare and extremely prevalent cell types at the same time, where k-medoids may not work properly and manual cluster merging is needed
- Embedding is a high-dimensional PCA (e.g. 30 principal components), where k-medoids is unstable and produces 1–2 cell clusters rather than meaningful partitions
- Data contain a trajectory with cycles or disconnected components, which Totem cannot model regardless of the clustering ensemble

## Inputs

- SingleCellExperiment object (e.g. binary_tree_1 dataset) prepared with PrepareTotem
- Low-dimensional embedding (5-dimensional LMDS from dyndimred) stored in the reducedDim slot

## Outputs

- R object (clustering_results.rds) holding all retained CLARA clusterings with k in 3–20, reproducible for a fixed seed
- Filtered clustering ensemble in which every clustering contains only clusters of at least five cells

## How to apply

Start from a low-dimensional embedding of the SingleCellExperiment object — in Totem's vignette, the 5-dimensional landmark MDS (LMDS) embedding produced by dyndimred. Invoke Totem's RunClustering/RunCLARA functions with runs = 10000 (the default number of k-medoids runs) and k sampled across the default range 3–20, enabling RNG seed handling so repeated executions yield identical clustering ensembles. Apply the min.cluster.size = 5 filter to drop any clustering containing clusters with fewer than five cells, and return the retained set as the output R object (e.g. clustering_results.rds) for downstream MST construction with Slingshot. Seeding matters because the whole downstream cell-connectivity ranking depends on this stochastic ensemble; without seeds, MSTs, connectivity scores, and Calinski-Harabasz rankings are not reproducible.

## Related tools

- **Totem** (Provides RunClustering/RunCLARA with runs, k-range, min.cluster.size, and RNG seed handling for the clustering ensemble) — https://github.com/elolab/Totem
- **dyndimred** (Generates the 5-dimensional LMDS embedding on which the seeded CLARA sampling is run)
- **SingleCellExperiment** (Container class for the input data and the embedding passed to RunClustering)

## Examples

```
remotes::install_github("elolab/Totem")
```

## Evaluation signals

- Number of retained clusterings equals 10,000 minus the runs filtered out by min.cluster.size
- Every retained clustering contains only clusters of at least five cells (min.cluster.size = 5 criterion)
- Re-running with the same RNG seed yields an identical set of clustering results (bit-level reproducibility)
- Sampled k values all fall within the configured range (default 3–20)
- Each retained clustering yields a valid MST topology when passed to Slingshot in the downstream step

## Limitations

- k-medoids may not work well with PCA if the number of principal components is high (e.g. 30), producing tiny 1–2 cell clusters
- On extremely imbalanced datasets (rare plus prevalent cell types), k-medoids may fail and clusters must be merged manually
- Filtering by min.cluster.size reduces the ensemble below the nominal run count, so the effective number of clusterings is data-dependent

## Evidence

- [intro] CLARA runs 10k times over k range: "the next step is to perform clustering that involves running the k-medoids (CLARA) algorithm a large number of times (default 10k)."
- [intro] min.cluster.size filter semantics: "The second parameter (`min.cluster.size`) determines the clusterings that are filtered out because they include too small clusters."
- [intro] Ensemble feeds MST-based connectivity: "`Totem` generates a large number of clustering results with a *k*-medoids algorithm (CLARA) and constructs an MST for each clustering."
- [intro] LMDS 5-dim embedding as clustering input: "we use the landmark Multi-Dimensional Scaling (LMDS) from `dyndimred` package [4] to generate a low-dimensional embedding with 5 dimensions."
- [readme] Totem package purpose (README): "A user-friendly tool for inferring tree-shaped trajectories from single cell data."
