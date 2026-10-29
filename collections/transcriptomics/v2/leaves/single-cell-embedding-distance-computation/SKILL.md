---
name: single-cell-embedding-distance-computation
description: Use when cell embeddings (e.g. adata.obsm['scace_emb']) and cluster labels
  are available and you must quantify cluster separation by size-weighted inter-centroid
  distance to decide which clusters to merge or when merging stops.
when_to_use_negative:
- When the goal is de novo initialization of clusters rather than merging — scAce
  uses Leiden there, not the weighted-distance merging rule
- When the number of clusters is already known and fixed, so no data-adaptive merging
  decision is needed
- When only raw gene expression is available — the criterion operates on learned cell
  embeddings, not the raw count matrix
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scAce
  role: Produces the cell embeddings and cluster labels on which the size-weighted
    distance computation and adaptive merging criterion are run
  repo: https://github.com/sldyns/scAce
- name: scanpy
  role: Provides the AnnData container holding adata.obsm['scace_emb'] embeddings
    and adata.obs['scace_cluster'] labels used as distance-computation inputs
- name: Python 3.8
  role: Implementation language of the scAce package that computes the embedding distances
    and merging steps
  repo: https://github.com/sldyns/scAce
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btad546
    title: 'scAce: an adaptive embedding and clustering method for single-cell gene
      expression data'
  source_content_hash: sha256:5d2ae2f6ca855f6f580e0382f41b56f5a0be2bd21454e8f6c28dcc3d32643f13
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sldyns/scAce
  tools:
  - Python
  - Python 3.8
  - scAce
  - scanpy
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# single-cell-embedding-distance-computation

## Summary

Computes intra-cluster distances and size-weighted inter-cluster centroid distances (Eqs. 7-10 of scAce) on learned cell embeddings to drive data-adaptive cluster merging, replacing the need to pre-estimate the number of clusters.

## When to use

Use when cell embeddings (e.g. adata.obsm['scace_emb']) and cluster labels are available and you must quantify cluster separation by size-weighted inter-centroid distance to decide which clusters to merge or when merging stops.

## When NOT to use

- When the goal is de novo initialization of clusters rather than merging — scAce uses Leiden there, not the weighted-distance merging rule
- When the number of clusters is already known and fixed, so no data-adaptive merging decision is needed
- When only raw gene expression is available — the criterion operates on learned cell embeddings, not the raw count matrix

## Inputs

- scanpy AnnData object with cell embeddings in adata.obsm['scace_emb']
- Initial cluster labels in adata.obs['scace_cluster']
- Cluster centroids and cluster sizes derived from the embeddings

## Outputs

- Size-weighted inter-cluster distance matrix for all cluster pairs (Eqs. 7-10)
- Intra-cluster distances per cluster
- Merge decision sequence per iteration with resulting cluster counts (cluster_merging_results.csv)
- Final cluster count after the d_bar/2 stopping criterion is reached

## How to apply

Load cell embeddings from adata.obsm['scace_emb'] and cluster labels from adata.obs['scace_cluster'] in a scanpy AnnData object. For every cluster pair, compute intra-cluster distances and size-weighted inter-cluster centroid distances as defined by Eqs. 7-10 of Methods 2.3, where each inter-cluster distance is weighted by cluster sizes (w_ij). The merging rule merges the pair whose weighted distance (w_ij * d_inter) is smallest, and merging repeats iteratively — recomputing labels and centroids after each merge — as long as the minimum size-weighted distance is below the adaptive stopping threshold d_bar/2. The process stops when no pair's weighted distance falls under d_bar/2, and the merge decision sequence plus final cluster count are recorded (e.g. cluster_merging_results.csv). Rationale: scAce's adaptive cluster merging achieves improved clustering without estimating the number of clusters in advance.

## Related tools

- **scAce** (Produces the cell embeddings and cluster labels on which the size-weighted distance computation and adaptive merging criterion are run) — https://github.com/sldyns/scAce
- **scanpy** (Provides the AnnData container holding adata.obsm['scace_emb'] embeddings and adata.obs['scace_cluster'] labels used as distance-computation inputs)
- **Python 3.8** (Implementation language of the scAce package that computes the embedding distances and merging steps) — https://github.com/sldyns/scAce

## Examples

```
import scanpy as sc; emb = adata.obsm['scace_emb']; labels = adata.obs['scace_cluster']  # compute w_ij*d_inter per pair, merge while min < d_bar/2
```

## Evaluation signals

- cluster_merging_results.csv exists and contains a field per merge iteration plus a final cluster count
- The final cluster count is a positive integer consistent with the recorded merge sequence
- At every recorded merge, the merged pair's size-weighted distance (w_ij * d_inter) was the smallest among all pairs and below d_bar/2
- The algorithm halts exactly when the minimum weighted inter-cluster distance exceeds d_bar/2
- Recomputing cluster labels and centroids after each merge is reflected in updated labels and centroids in the outputs

## Limitations

- The stopping threshold d_bar/2 is adaptive to the data; without recomputing d_bar after each merge the criterion may over- or under-merge
- Distances must be computed on the learned embeddings (scace_emb), so poor embedding quality propagates into wrong merge decisions
- Merging depends on cluster sizes through w_ij, so very unbalanced cluster populations can bias which pair is merged first

## Evidence

- [methods] size-weighted inter-cluster distance merges below d_bar/2: [evidence span withheld — anchor status: not_found]
- [abstract] adaptive cluster merging avoids estimating cluster number: "we develop an adaptive cluster merging approach which achieves improved clustering results without the need to estimate the number of clusters in advance"
- [readme] iterative alternation of network update and cluster merging: "the network parameters, cell embeddings, cluster labels and centroids are iteratively updated by alternately performing network update and cluster merging steps"
- [readme] output locations of embeddings and cluster labels in AnnData: "The output adata contains cluster labels in `adata.obs['scace_cluster']` and the cell embeddings in `adata.obsm['scace_emb']`"
