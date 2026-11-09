---
name: single-cell-cluster-label-aggregation
description: Use when you have a latent cell embedding (e.g. from a dual denoising
  autoencoder) and K basic clusterings with varying k, and need a single consensus
  partition that is stable across random seeds and cluster counts.
when_to_use_negative:
- Input is raw gene expression counts or already a single hard clustering — the skill
  requires a latent embedding plus multiple basic partitions
- The goal is marker-gene or functional genomic analysis rather than consensus partitioning;
  scBGEDA handles that by a separate quantification of gene influence on clusters
- No MATLAB environment is available and BGEC.m/BC.m cannot be executed
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2269
tools:
- name: scBGEDA
  role: package providing the bipartite graph ensemble clustering pipeline
  repo: https://github.com/wangyh082/scBGEDA
- name: BC.m
  role: 'main driver of the third module: generates the basic clusterings and invokes
    consensus'
  repo: https://github.com/wangyh082/scBGEDA
- name: BGEC.m
  role: builds the bipartite graph over cells and basic clusters and computes the
    graph-based consensus function
- name: rand_index.m
  role: computes ARI of the consensus labels against ground truth
- name: computeNMI.m
  role: computes NMI of the consensus labels against ground truth
- name: MATLAB
  role: runtime environment used to execute BC.m and BGEC.m
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btad075
    title: 'scBGEDA: deep single-cell clustering analysis via a dual denoising autoencoder
      with bipartite graph ensemble clustering'
  source_content_hash: sha256:f0ec31e7226a2f61207d7e9213bc287f211bfbb4a8806ad5d7944d2f55cdb048
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
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# single-cell-cluster-label-aggregation

## Summary

Aggregates many basic K-means partitions of a single-cell latent embedding into one consensus clustering by treating cells and basic clusters as nodes of a bipartite graph and solving a graph-based consensus function via the transfer cut criterion (as implemented in scBGEDA's BGEC.m).

## When to use

Use when you have a latent cell embedding (e.g. from a dual denoising autoencoder) and K basic clusterings with varying k, and need a single consensus partition that is stable across random seeds and cluster counts.

## When NOT to use

- Input is raw gene expression counts or already a single hard clustering — the skill requires a latent embedding plus multiple basic partitions
- The goal is marker-gene or functional genomic analysis rather than consensus partitioning; scBGEDA handles that by a separate quantification of gene influence on clusters
- No MATLAB environment is available and BGEC.m/BC.m cannot be executed

## Inputs

- Latent embedding matrix Z of cells produced by the single-cell dual denoising autoencoder (SDDA.py)
- Ensemble size K (default 100) and random seeds, e.g. '1111, 2222, 3333, 4444, 5555, 6666, 7777, 8888, 9999, 10000'
- Basic clustering range parameters: bcsLowK (kmin, e.g. 2) and bcsUpK (kmax, e.g. 40 or 60)

## Outputs

- A single consensus cluster label for every cell (consensus_clustering_labels.csv)
- Median ARI and NMI values reported after clustering

## How to apply

Load the latent representation Z produced by SDDA.py, then generate K=100 basic clusterings with BC.m, drawing each run's k randomly between kmin=2 and kmax=60 (the README example uses 'BC 'Adam' 100 2 40'). Build the bipartite graph G={Z, phi, B} where B is the cell-to-cluster binary cross-affinity matrix (bij=1 if cell zi belongs to cluster Cj), implemented in BGEC.m. Solve the generalized eigen-problem of G and use the transfer cut to project it onto the much smaller graph G_R with only kc cluster nodes and affinity E_R = B^T * tau~_Z^{-1} * B; the first k eigenvectors of G_R recover the corresponding eigenvectors of G, and K-means on those eigenvectors yields the final consensus labels. The rationale is that co-membership of cells across many base partitions is encoded directly in graph structure, avoiding explicit co-association matrices. Score the consensus labels against ground truth with rand_index.m (ARI) and computeNMI.m (NMI).

## Related tools

- **scBGEDA** (package providing the bipartite graph ensemble clustering pipeline) — https://github.com/wangyh082/scBGEDA
- **BC.m** (main driver of the third module: generates the basic clusterings and invokes consensus) — https://github.com/wangyh082/scBGEDA
- **BGEC.m** (builds the bipartite graph over cells and basic clusters and computes the graph-based consensus function)
- **rand_index.m** (computes ARI of the consensus labels against ground truth)
- **computeNMI.m** (computes NMI of the consensus labels against ground truth)
- **MATLAB** (runtime environment used to execute BC.m and BGEC.m)

## Examples

```
matlab -r "BC 'Adam' 100 2 40 [1111, 2222, 3333, 4444, 5555, 6666, 7777, 8888, 9999, 10000]"
```

## Evaluation signals

- Every cell in the dataset receives exactly one consensus cluster label (no missing or duplicate assignments)
- Consensus labels are evaluable by rand_index.m and computeNMI.m, producing finite ARI and NMI values
- Median ARI and NMI across the seed set (e.g. 1111–10000) are reported and should match or exceed single-seed baselines
- The number of basic clusterings executed equals the ensemble size K=100, with each k drawn within the specified kmin/kmax range
- The reduced graph G_R has only kc cluster nodes with affinity matrix E_R = B^T * tau~_Z^{-1} * B, consistent with the transfer cut formulation

## Limitations

- Requires MATLAB in addition to Python; the pipeline spans SDDA.py (Python) and BC.m/BGEC.m (MATLAB)
- Ensemble parameters (K, kmin, kmax, seeds) must be tuned; the README exposes 'm' (ensemble size), 'bcsLowK', and 'bcsUpK' as user-settable with defaults
- Consensus quality depends on the latent representation from the dual denoising autoencoder; the article notes obtaining consensus partitions from feature representations remains under-explored

## Evidence

- [methods] it first generates K=100 K-means basic clusterings (with each k randomly chosen between kmin=2 and kmax=60) on the latent representation Z: [evidence span withheld — anchor status: not_found]
- [methods] The consensus partition is obtained by solving the generalized eigen-problem, which is transferred via the transfer cut approach: [evidence span withheld — anchor status: not_found]
- [readme] matlab -r "BC 'Adam' 100 2 40 [1111, 2222, 3333, 4444, 5555, 6666, 7777, 8888, 9999, 10000]": "matlab -r "BC 'Adam' 100 2 40 [1111, 2222, 3333, 4444, 5555, 6666, 7777, 8888, 9999, 10000]""
- [abstract] a bipartite graph ensemble clustering algorithm is designed to exploit the relationships between cells and the learned latent embedded space by means of a graph-based consensus function: "a bipartite graph ensemble clustering algorithm is designed to exploit the relationships between cells and the learned latent embedded space by means of a graph-based consensus function"
- [supplementary] BGEC.m       | Bipartite Graph Ensemble Clustering: "BGEC.m       | Bipartite Graph Ensemble Clustering"
