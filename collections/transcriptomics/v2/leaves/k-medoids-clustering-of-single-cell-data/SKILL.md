---
name: k-medoids-clustering-of-single-cell-data
description: 'Use when building trajectory-inference input for Totem: you have a SingleCellExperiment
  with a low-dimensional embedding (e.g. 5-dimensional LMDS) and need many candidate
  clusterings with varying k to construct MSTs and rank clusterings.'
when_to_use_negative:
- The dataset combines extremely rare and extremely prevalent cell types — the article
  notes k-medoids 'might not work properly' and manual cluster merging is required
- The embedding is a high-dimensional PCA (e.g. 30 principal components) — k-medoids
  may produce 1–2 cell clusters; prefer a lower-dimensional embedding like 5D LMDS
- The input is not a single-cell expression matrix with an embedding but an already-clustered
  or feature-table dataset
edam_operation: http://edamontology.org/operation_3680
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_3170
tools:
- name: Totem
  role: Provides RunClustering/RunCLARA functions for the repeated k-medoids runs
    and the min.cluster.size filtering
  repo: https://github.com/elolab/Totem
- name: SingleCellExperiment
  role: Container format for the prepared single-cell data passed into clustering
- name: dyndimred
  role: Generates the low-dimensional (5-dimension LMDS) embedding on which CLARA
    clustering is run
- name: R
  role: Runtime environment for the Totem clustering workflow
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad515
    title: Cell-connectivity-guided trajectory inference from single-cell data
  source_content_hash: sha256:d030ede9fe3827e51047e33db03eb09e4bcb472b62bde9fe7774e6be1e9f5d60
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/elolab/Totem
  tools:
  - R
  - SingleCellExperiment
  - Totem
  - dyndimred
  edam_operation: http://edamontology.org/operation_3680
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# k-medoids-clustering-of-single-cell-data

## Summary

Repeatedly run the CLARA k-medoids algorithm (as implemented in Totem's RunClustering/RunCLARA) on a low-dimensional single-cell embedding, sampling k across a range (default 3–20) over many runs (default 10,000) to generate a large pool of candidate clusterings for trajectory inference.

## When to use

Use when building trajectory-inference input for Totem: you have a SingleCellExperiment with a low-dimensional embedding (e.g. 5-dimensional LMDS) and need many candidate clusterings with varying k to construct MSTs and rank clusterings.

## When NOT to use

- The dataset combines extremely rare and extremely prevalent cell types — the article notes k-medoids 'might not work properly' and manual cluster merging is required
- The embedding is a high-dimensional PCA (e.g. 30 principal components) — k-medoids may produce 1–2 cell clusters; prefer a lower-dimensional embedding like 5D LMDS
- The input is not a single-cell expression matrix with an embedding but an already-clustered or feature-table dataset

## Inputs

- SingleCellExperiment object prepared with Totem's PrepareTotem (e.g. binary_tree_1 simulated dataset)
- Low-dimensional embedding of the cells, e.g. 5-dimensional LMDS produced by dyndimred

## Outputs

- R object of retained clustering results (clustering_results.rds), each clustering restricted to clusters of >= 5 cells
- Pool of up to 10,000 k-medoids clusterings with k ranging 3–20, ready for MST construction and ranking

## How to apply

First produce a low-dimensional embedding of the SingleCellExperiment data — the article uses 5-dimensional landmark MDS (LMDS) from dyndimred — since k-medoids instability increases on high-dimensional PCA input. Then invoke Totem's RunClustering/RunCLARA functions with runs = 10000 (default), k sampled across 3–20 (default), and RNG seed handling enabled so the clustering pool is reproducible. Apply the min.cluster.size = 5 filter to drop any clustering that contains a cluster with fewer than five cells; only retained clusterings feed downstream MST construction (via Slingshot) and Calinski-Harabasz/connectivity ranking. The rationale is that Totem's cell-connectivity scoring requires a large, diverse pool of clusterings whose topologies can be estimated as minimum spanning trees.

## Related tools

- **Totem** (Provides RunClustering/RunCLARA functions for the repeated k-medoids runs and the min.cluster.size filtering) — https://github.com/elolab/Totem
- **SingleCellExperiment** (Container format for the prepared single-cell data passed into clustering)
- **dyndimred** (Generates the low-dimensional (5-dimension LMDS) embedding on which CLARA clustering is run)
- **R** (Runtime environment for the Totem clustering workflow)

## Examples

```
clustering_results <- RunClustering(dataset, dimred = "lmds", dimred_param = 5, runs = 10000, k_min = 3, k_max = 20, min.cluster.size = 5)
```

## Evaluation signals

- The number of retained clusterings equals the run count (default 10,000) minus clusterings filtered out by min.cluster.size
- Every retained clustering contains only clusters with at least five cells per the min.cluster.size = 5 criterion
- Re-running with RNG seed handling enabled reproduces the same clustering pool (reproducibility check)
- The retained pool covers the requested k range (default 3–20) and can be fed to MST construction with Slingshot
- Downstream ranking by Calinski-Harabasz score and cell connectivity is possible on the returned clustering_results.rds object

## Limitations

- k-medoids may not work well with PCA if the number of principal components is high (e.g. 30), producing 1–2 cell clusters
- On datasets with extremely rare and extremely prevalent cell types simultaneously, k-medoids may fail and require manual cluster merging
- Clusterings containing clusters with fewer than five cells are discarded, so the retained run count is below the requested 10,000

## Evidence

- [intro] CLARA k-medoids run count and k range: "the next step is to perform clustering that involves running the k-medoids (CLARA) algorithm a large number of times (default 10k)."
- [intro] min.cluster.size filter: "The second parameter (`min.cluster.size`) determines the clusterings that are filtered out because they include too small clusters."
- [intro] LMDS embedding as clustering input: "we use the landmark Multi-Dimensional Scaling (LMDS) from `dyndimred` package [4] to generate a low-dimensional embedding with 5 dimensions."
- [discussion] PCA failure mode for k-medoids: "the k-medoids algorithm may not work well with PCA if the number principal components is high (e.g. 30)"
- [readme] README Totem role: "Totem generates a large number of clustering results with a *k*-medoids algorithm (CLARA) and constructs an MST for each clustering."
