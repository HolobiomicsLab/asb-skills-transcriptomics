---
name: adaptive-cluster-merging
description: 'Use when running scAce (or benchmarking against it) on scRNA-seq count
  data and the number of clusters is unknown: after VAE pre-training and cluster initialization,
  this criterion decides which Leiden-derived or split-derived clusters to merge and
  when to stop.'
when_to_use_negative:
- The number of clusters is already known and fixed — merging will alter a user-supplied
  cluster count rather than preserve it
- Cell embeddings have not been produced by scAce's VAE pre-training (e.g. embeddings
  from PCA or another method), since the criterion alternates merging with scAce's
  own network updates
- Input is raw counts without the required preprocessing (filtering, normalize_per_cell,
  log1p, scale) — the ZINB-based embeddings and distance criterion assume the preprocessed
  space
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scAce
  role: provides run_scace, which performs VAE pre-training, cluster initialization,
    and the alternating network-update / cluster-merging iterations
  repo: https://github.com/sldyns/scAce
- name: scanpy
  role: AnnData container for the expression matrix, embeddings and cluster labels;
    preprocessing (filtering, normalize_per_cell, log1p, scale)
- name: Leiden
  role: obtains the de novo initial cluster labels that the merging step then adaptively
    refines
provenance:
  source_task_ids:
  - task_006
  - task_008
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad546
    title: 'scAce: an adaptive embedding and clustering method for single-cell gene
      expression data'
  source_content_hash: sha256:56e8ca039b36472b90accc5ad8d336f1a708f4f4657289395b433375edd062d0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sldyns/scAce
  tools:
  - Leiden
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
yes_rate: 0.8181818181818182
n_claims: 11
---

# adaptive-cluster-merging

## Summary

scAce's adaptive cluster merging criterion (Eqs. 7-10): iteratively merge the cluster pair with the smallest size-weighted inter-cluster centroid distance while it is below an adaptive threshold d_bar/2, alternating with VAE network updates, so that a good cluster number is found without pre-estimating it.

## When to use

Use when running scAce (or benchmarking against it) on scRNA-seq count data and the number of clusters is unknown: after VAE pre-training and cluster initialization, this criterion decides which Leiden-derived or split-derived clusters to merge and when to stop.

## When NOT to use

- The number of clusters is already known and fixed — merging will alter a user-supplied cluster count rather than preserve it
- Cell embeddings have not been produced by scAce's VAE pre-training (e.g. embeddings from PCA or another method), since the criterion alternates merging with scAce's own network updates
- Input is raw counts without the required preprocessing (filtering, normalize_per_cell, log1p, scale) — the ZINB-based embeddings and distance criterion assume the preprocessed space

## Inputs

- AnnData with cell embeddings in adata.obsm['scace_emb'] from a pre-trained scAce VAE
- Initial cluster labels in adata.obs['scace_cluster'] (Leiden de novo initialization, or cluster splitting of existing results from other methods)
- Pre-processed scRNA-seq count matrix (filter_genes min_cells=3, filter_cells min_genes=200, normalize_per_cell, log1p, scale)

## Outputs

- Final cluster labels in adata.obs['scace_cluster'] and final cell embeddings in adata.obsm['scace_emb']
- cluster_merging_results.csv: merge-decision sequence per iteration (pairs merged, cluster count at each step) plus a final positive-integer cluster count
- Final ARI against ground-truth cell-type labels for downstream comparison

## How to apply

Load cell embeddings (adata.obsm['scace_emb']) and initial cluster labels (adata.obs['scace_cluster']) into a scanpy AnnData. Compute intra-cluster distances and size-weighted inter-cluster centroid distances (w_ij * d_ij_inter) for every cluster pair per Eqs. 7-10 of Methods 2.3. Merge the pair with the smallest weighted distance whenever the minimum weighted inter-cluster distance is below the adaptive stopping threshold d_bar/2, recomputing cluster labels and centroids after each merge. Alternate this merging step with VAE network-update steps — network parameters, cell embeddings, cluster labels and centroids are iteratively updated until the iteration stops (no pair meets the criterion). Log the cluster count after each merge iteration; e.g. on Mouse kidney scAce went 25 -> 17 -> 10 -> 9 -> 8 clusters across merging iterations (final ARI 0.93). Save the merge decision sequence, per-iteration cluster counts, and the final positive-integer cluster count.

## Related tools

- **scAce** (provides run_scace, which performs VAE pre-training, cluster initialization, and the alternating network-update / cluster-merging iterations) — https://github.com/sldyns/scAce
- **scanpy** (AnnData container for the expression matrix, embeddings and cluster labels; preprocessing (filtering, normalize_per_cell, log1p, scale))
- **Leiden** (obtains the de novo initial cluster labels that the merging step then adaptively refines)

## Examples

```
from scace import run_scace; adata = run_scace(adata); print(adata.obs['scace_cluster'].nunique(), adata.obsm['scace_emb'].shape)
```

## Evaluation signals

- cluster_merging_results.csv exists with a field per merge iteration and a final cluster count that is a positive integer consistent with the recorded merge sequence and the d_bar/2 stopping rule
- Every merged pair had the smallest size-weighted inter-cluster distance (w_ij * d_ij_inter) among all pairs at that iteration, and that distance was below d_bar/2
- Cluster count is monotonically non-increasing across iterations (merges only), and iteration stops exactly when no pair meets the criterion
- On Mouse kidney, the reproduced trajectory matches the reported 25 -> 17 -> 10 -> 9 -> 8 clusters with final ARI = 0.93
- Final labels land in adata.obs['scace_cluster'] and final embeddings in adata.obsm['scace_emb'] of the returned AnnData

## Limitations

- Benchmarks show giving scDeepCluster (avg ARI 0.72) or ADClust (avg ARI 0.61) the same initialization and merging still trails scAce (avg ARI 0.82) — the merging criterion alone is insufficient without scAce's iterative network update
- Other refinement methods can misbehave on the same data: SCCAF stopped early at 17 clusters on Mouse kidney, and ADClust over-merged to 5 mixed clusters
- Merging requires a pre-trained VAE network and initial labels as prerequisites; it is not a standalone clustering initializer

## Evidence

- [methods] adaptive cluster merging step to iteratively update cluster labels and cell embeddings: "an adaptive cluster merging step to iteratively
update cluster labels and cell embeddings"
- [readme] network params, embeddings, labels and centroids iteratively updated: "given the pre-trained VAE network and the initial cluster labels, the network parameters, cell embeddings, cluster labels and centroids are iteratively updated by alternately performing network"
- [abstract] adaptive merging avoids estimating cluster number: "we develop an adaptive cluster merging approach which achieves improved clustering results without the need to estimate the number of clusters in advance"
- [readme] Leiden supplies de novo initial labels before merging: "With *de novo* initialization, Leiden is used to obtain initial cluster labels"
