---
name: bipartite-graph-construction-from-clusterings
description: Use when you have K base partitions of a cell-by-feature latent embedding
  (e.g. from a dual denoising autoencoder) and need a single consensus cell clustering
  that exploits cell-to-cluster relationships.
when_to_use_negative:
- Input is raw scRNA-seq count data rather than a learned latent representation from
  the autoencoder — preprocessing and SDDA must run first
- You need only a single flat clustering with no ensemble/consensus step — plain K-means
  on the embedding suffices
- No MATLAB environment available and no equivalent implementation of the transfer
  cut generalized eigen-problem is at hand
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scBGEDA
  role: Full pipeline whose third module performs bipartite graph ensemble clustering
  repo: https://github.com/wangyh082/scBGEDA
- name: BC.m
  role: 'Main MATLAB function of the third module: orchestrates basic clustering generation
    and consensus'
  repo: https://github.com/wangyh082/scBGEDA
- name: BGEC.m
  role: Implements the bipartite graph ensemble clustering on the cell-cluster graph
  repo: https://github.com/wangyh082/scBGEDA
- name: rand_index.m
  role: Computes ARI values after consensus clustering for validation
  repo: https://github.com/wangyh082/scBGEDA
- name: computeNMI.m
  role: Computes NMI values after consensus clustering for validation
  repo: https://github.com/wangyh082/scBGEDA
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btad075
    title: 'scBGEDA: deep single-cell clustering analysis via a dual denoising autoencoder
      with bipartite graph ensemble clustering'
  source_content_hash: sha256:09355bfe49b745f2780e3658e85d9af0cafdf1fd4fd126c3f2a43e0c72f3cc32
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wangyh082/scBGEDA
  tools:
  - BC.m
  - BGEC.m
  - MATLAB
  - computeNMI.m
  - rand_index.m
  - scBGEDA
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# bipartite-graph-construction-from-clusterings

## Summary

Constructs a bipartite graph whose nodes are cells and K-means basic clusters (affinity matrix B with bij=1 if cell zi belongs to cluster Cj), then reduces it via the transfer cut criterion to a small cluster-only graph G_R to obtain a graph-based consensus clustering of scRNA-seq latent embeddings.

## When to use

Use when you have K base partitions of a cell-by-feature latent embedding (e.g. from a dual denoising autoencoder) and need a single consensus cell clustering that exploits cell-to-cluster relationships.

## When NOT to use

- Input is raw scRNA-seq count data rather than a learned latent representation from the autoencoder — preprocessing and SDDA must run first
- You need only a single flat clustering with no ensemble/consensus step — plain K-means on the embedding suffices
- No MATLAB environment available and no equivalent implementation of the transfer cut generalized eigen-problem is at hand

## Inputs

- Latent embedding matrix Z produced by the single-cell dual denoising autoencoder (SDDA.py output)
- K=100 K-means basic clusterings with each k randomly chosen between kmin=2 and kmax (e.g. 40)
- Cell-to-cluster membership encoded in cross-affinity matrix B

## Outputs

- Consensus cluster label for every cell (consensus_clustering_labels.csv)
- Reduced graph G_R with kc cluster nodes and affinity matrix E_R = B^T * tau~_Z^{-1} * B
- Median ARI and NMI values reported by rand_index.m and computeNMI.m

## How to apply

Run K=100 K-means basic clusterings on the latent representation Z, drawing each k randomly from kmin=2 to kmax (e.g. 60 in the source card, 40 in the README invocation of BC 'Adam' 100 2 40). Build the bipartite graph G={Z, phi, B} in BGEC.m, where both samples and clusters are nodes and the cross-affinity matrix B encodes cell-to-cluster membership (bij=1 if cell zi belongs to cluster Cj, else 0). Solve the generalized eigen-problem on G and transfer the solution via the transfer cut approach to the reduced graph G_R, which has only kc cluster nodes and affinity matrix E_R = B^T * tau~_Z^{-1} * B; the first k eigenvectors of G_R recover the corresponding eigenvectors of G. Run K-means on those recovered eigenvectors to produce the final consensus partition, implemented as the third module of scBGEDA via BC.m/BGEC.m in MATLAB.

## Related tools

- **scBGEDA** (Full pipeline whose third module performs bipartite graph ensemble clustering) — https://github.com/wangyh082/scBGEDA
- **BC.m** (Main MATLAB function of the third module: orchestrates basic clustering generation and consensus) — https://github.com/wangyh082/scBGEDA
- **BGEC.m** (Implements the bipartite graph ensemble clustering on the cell-cluster graph) — https://github.com/wangyh082/scBGEDA
- **rand_index.m** (Computes ARI values after consensus clustering for validation) — https://github.com/wangyh082/scBGEDA
- **computeNMI.m** (Computes NMI values after consensus clustering for validation) — https://github.com/wangyh082/scBGEDA

## Examples

```
matlab -r "BC 'Adam' 100 2 40 [1111, 2222, 3333, 4444, 5555, 6666, 7777, 8888, 9999, 10000]"
```

## Evaluation signals

- Every cell in the dataset receives exactly one consensus cluster label
- The consensus clustering is evaluable by the package's rand_index.m and computeNMI.m routines, yielding finite ARI and NMI values
- The reduced affinity matrix E_R = B^T * tau~_Z^{-1} * B has dimensions kc x kc, matching the number of unique clusters in the K basic partitions
- B entries are binary: bij=1 exactly when cell zi belongs to cluster Cj, else 0
- The final K-means on the recovered eigenvectors of G converges and the output reports median ARI and NMI

## Limitations

- Basic-cluster k range must be chosen (kmin=2 to bcsUpK, e.g. 40); a poor upper bound may limit consensus quality
- Requires MATLAB for the third module (BC.m/BGEC.m), separate from the Python environment used for SDDA.py
- Consensus quality depends on the upstream latent representation from the dual denoising autoencoder; a weak embedding degrades the graph ensemble

## Evidence

- [supplementary] the first k eigenvectors of G_R yield the corresponding eigenvectors of G, on which K-means produces the final consensus clustering: [evidence span withheld — anchor status: not_found]
- [supplementary] affinity matrix E_R = B^T * tau~_Z^{-1} * B: [evidence span withheld — anchor status: not_found]
- [supplementary] it first generates K=100 K-means basic clusterings (with each k randomly chosen between kmin=2 and kmax=60) on the latent representation Z: [evidence span withheld — anchor status: not_found]
- [abstract] a bipartite graph ensemble clustering algorithm is designed to exploit the relationships between cells and the learned latent embedded space: "a bipartite graph ensemble clustering algorithm is designed to exploit the relationships between cells and the learned latent embedded space by means of a graph-based consensus function"
- [readme] BC.m | Main function of the third module of scBGEDA: "BC.m         | Main function of the third module of scBGEDA"
