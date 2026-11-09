---
name: transfer-cut-graph-partitioning
description: 'Use when a consensus clustering must be derived from many base partitions
  of a single-cell latent embedding (the SDDA output Z), combining sample and cluster
  nodes in a bipartite graph and solving via transfer cut. Triggers: K-means ensemble
  inputs, ARI/NMI consensus evaluation.'
when_to_use_negative:
- The input is raw scRNA-seq count data or a preprocessed expression matrix — this
  module requires the latent embedding Z from the dual denoising autoencoder (SDDA.py)
  first
- Only a single clustering is available and no ensemble of basic clusterings can be
  generated — the consensus function needs K=100 base partitions
- MATLAB is unavailable and BC.m/BGEC.m cannot be executed — the transfer cut implementation
  is MATLAB-based
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: BC.m
  role: 'Main function of the third module: orchestrates generation of the 100 K-means
    basic clusterings and calls the bipartite graph ensemble clustering'
  repo: https://github.com/wangyh082/scBGEDA
- name: BGEC.m
  role: 'Implements Bipartite Graph Ensemble Clustering: constructs the sample-cluster
    graph and solves the transfer cut consensus problem'
  repo: https://github.com/wangyh082/scBGEDA
- name: MATLAB
  role: Runtime environment in which BC.m, BGEC.m, rand_index.m, and computeNMI.m
    are executed
- name: rand_index.m
  role: Computes ARI values after clustering to evaluate the consensus partition
  repo: https://github.com/wangyh082/scBGEDA
- name: computeNMI.m
  role: Computes NMI values after clustering to evaluate the consensus partition
  repo: https://github.com/wangyh082/scBGEDA
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btad075
    title: 'scBGEDA: deep single-cell clustering analysis via a dual denoising autoencoder
      with bipartite graph ensemble clustering'
  source_content_hash: sha256:9c3d68dc8b59342f10601ae735bc899971ab5532a7b222cd7dcd36a61e149f0b
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
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 15
---

# transfer-cut-graph-partitioning

## Summary

scBGEDA's third module: bipartite graph ensemble clustering that generates K=100 K-means basic clusterings on the latent embedding Z and merges them into one consensus partition via a graph-based consensus function solved with a transfer cut. It turns a deep autoencoder's representation into a robust cell-type clustering.

## When to use

Use when a consensus clustering must be derived from many base partitions of a single-cell latent embedding (the SDDA output Z), combining sample and cluster nodes in a bipartite graph and solving via transfer cut. Triggers: K-means ensemble inputs, ARI/NMI consensus evaluation.

## When NOT to use

- The input is raw scRNA-seq count data or a preprocessed expression matrix — this module requires the latent embedding Z from the dual denoising autoencoder (SDDA.py) first
- Only a single clustering is available and no ensemble of basic clusterings can be generated — the consensus function needs K=100 base partitions
- MATLAB is unavailable and BC.m/BGEC.m cannot be executed — the transfer cut implementation is MATLAB-based

## Inputs

- Latent embedding matrix Z from the single-cell dual denoising autoencoder (SDDA.py output)
- Dataset name (e.g., 'Adam') used to locate the embedding files
- Ensemble parameters: ensemble size m (100), minimum cluster count bcsLowK (2), maximum cluster count bcsUpK (e.g., 40)
- Random seed vector shared with SDDA.py (e.g., 1111...10000)

## Outputs

- consensus_clustering_labels.csv with exactly one cluster label per cell
- Median ARI and NMI values over seeds reported by BC.m
- Intermediate bipartite graph structures G and reduced graph G_R (affinity E_R) inside the MATLAB pipeline

## How to apply

Load the latent representation Z produced by SDDA.py (e.g., '256, 32' hidden dims) as input. Generate K=100 base partitions by running K-means on Z, drawing each k randomly between kmin=2 and kmax (40 in the README example, up to 60 per the module description), invoking MATLAB as 'BC <dataname> 100 ...' (BC.m). Build the bipartite graph G={Z, phi, B} in BGEC.m where both cells and basic clusters are nodes and the binary cross-affinity matrix B encodes cell-to-cluster membership (bij=1 if cell zi belongs to cluster Cj). Solve the generalized eigen-problem for the consensus function, then transfer the solution via the transfer cut to a much smaller graph G_R with only kc cluster nodes and affinity E_R = B^T * tau~_Z^{-1} * B; the first k eigenvectors of G_R yield the corresponding eigenvectors of G, and K-means on these produces the final consensus clustering. Write the per-cell consensus labels to consensus_clustering_labels.csv and report median ARI (rand_index.m) and NMI (computeNMI.m).

## Related tools

- **BC.m** (Main function of the third module: orchestrates generation of the 100 K-means basic clusterings and calls the bipartite graph ensemble clustering) — https://github.com/wangyh082/scBGEDA
- **BGEC.m** (Implements Bipartite Graph Ensemble Clustering: constructs the sample-cluster graph and solves the transfer cut consensus problem) — https://github.com/wangyh082/scBGEDA
- **MATLAB** (Runtime environment in which BC.m, BGEC.m, rand_index.m, and computeNMI.m are executed)
- **rand_index.m** (Computes ARI values after clustering to evaluate the consensus partition) — https://github.com/wangyh082/scBGEDA
- **computeNMI.m** (Computes NMI values after clustering to evaluate the consensus partition) — https://github.com/wangyh082/scBGEDA

## Examples

```
matlab -r "BC 'Adam' 100 2 40 [1111, 2222, 3333, 4444, 5555, 6666, 7777, 8888, 9999, 10000]"
```

## Evaluation signals

- The consensus clustering assigns exactly one cluster label to every cell in the dataset (no unassigned cells)
- consensus_clustering_labels.csv is evaluable by rand_index.m and computeNMI.m without schema errors
- BC.m reports the clustering performance with median values of ARI and NMI over the provided random seeds
- K=100 basic clusterings were generated with each k drawn between bcsLowK=2 and bcsUpK (e.g., 40)
- The reduced graph G_R contains only kc cluster nodes with affinity E_R = B^T * tau~_Z^{-1} * B, i.e., dimensionality equals the number of basic clusters, not cells

## Limitations

- The transfer cut criterion requires solving a generalized eigen-problem, whose cost scales with the number of basic clusters; correctness depends on the eigenvector transfer from G_R back to G
- Consensus quality depends on the diversity of base partitions — with kmin=2 and a low bcsUpK the ensemble may be too coarse
- Evaluation in the article is against ground truth via ARI/NMI only; the README lists requirements (MATLAB pipeline, e.g., jgraph 0.2.1, scipy 1.5.4) that must be satisfied before running

## Evidence

- [abstract] a bipartite graph ensemble clustering algorithm is designed to exploit the relationships between cells and the learned latent embedded space by means of a graph-based consensus function: "a bipartite graph ensemble clustering algorithm is designed to exploit the relationships between cells and the learned latent embedded space by means of a graph-based consensus function"
- [readme] matlab -r "BC 'Adam' 100 2 40 [1111, 2222, 3333, 4444, 5555, 6666, 7777, 8888, 9999, 10000]": "matlab -r "BC 'Adam' 100 2 40 [1111, 2222, 3333, 4444, 5555, 6666, 7777, 8888, 9999, 10000]""
- [readme] The final output reports the clustering performance and the median values of ARI and NMI is provided, respectively.: "The final output reports the clustering performance and the median values of ARI and NMI is provided, respectively."
- [supplementary] BGEC.m       | Bipartite Graph Ensemble Clustering: "BGEC.m       | Bipartite Graph Ensemble Clustering"
