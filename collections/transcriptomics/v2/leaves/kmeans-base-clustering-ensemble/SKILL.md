---
name: kmeans-base-clustering-ensemble
description: Use when you need to create diverse base partitions on an embedded scRNA-seq
  representation (e.g. latent matrix from SDDA.py) before building a bipartite graph
  for consensus clustering of cell types.
when_to_use_negative:
- Input is raw gene expression counts rather than a learned latent representation
  — scBGEDA generates basic clusterings on the compressed low-dimensional space from
  the dual denoising autoencoder, not on raw scRNA-seq data.
- You need a single fixed-k clustering with no ensemble aggregation — this skill's
  purpose is to create many partitions of varying k for a graph-based consensus function,
  not a final partition itself.
- The downstream goal is direct clustering of cells without consensus assembly — this
  skill feeds BC.m/BGEC.m and the transfer cut step; running K-means alone is the
  wrong granularity.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: BC.m
  role: Main function of the third module of scBGEDA; runs the ensemble of K-means
    basic clusterings on the latent representation and drives the bipartite graph
    ensemble clustering
  repo: https://github.com/wangyh082/scBGEDA
- name: BGEC.m
  role: Implements Bipartite Graph Ensemble Clustering; consumes the basic clusterings
    as cell-to-cluster membership matrix B to build the bipartite graph G
  repo: https://github.com/wangyh082/scBGEDA
- name: MATLAB
  role: Execution environment in which BC.m and BGEC.m are invoked via 'matlab -r'
    with the dataset name, ensemble size, k bounds and seeds
- name: SDDA.py
  role: Produces the latent embedding Z on which the K-means basic clusterings are
    computed
  repo: https://github.com/wangyh082/scBGEDA
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btad075
    title: 'scBGEDA: deep single-cell clustering analysis via a dual denoising autoencoder
      with bipartite graph ensemble clustering'
  source_content_hash: sha256:af16f22820b966db3db9a48e868e8f5d2bf631905d69915fa45362c8e518cb07
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wangyh082/scBGEDA
  tools:
  - BC.m
  - BGEC.m
  - MATLAB
  - SDDA.py
  - scBGEDA
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# kmeans-base-clustering-ensemble

## Summary

Generates a large ensemble of K-means basic clusterings (K=100 base partitions, each with a random k drawn between kmin and kmax) on the latent representation Z learned by the single-cell dual denoising autoencoder, to serve as the diversity-producing front end of bipartite graph ensemble clustering.

## When to use

Use when you need to create diverse base partitions on an embedded scRNA-seq representation (e.g. latent matrix from SDDA.py) before building a bipartite graph for consensus clustering of cell types.

## When NOT to use

- Input is raw gene expression counts rather than a learned latent representation — scBGEDA generates basic clusterings on the compressed low-dimensional space from the dual denoising autoencoder, not on raw scRNA-seq data.
- You need a single fixed-k clustering with no ensemble aggregation — this skill's purpose is to create many partitions of varying k for a graph-based consensus function, not a final partition itself.
- The downstream goal is direct clustering of cells without consensus assembly — this skill feeds BC.m/BGEC.m and the transfer cut step; running K-means alone is the wrong granularity.

## Inputs

- Latent representation matrix Z produced by the single-cell dual denoising autoencoder (output of SDDA.py)
- Dataset name (e.g. 'Adam') pointing to the preprocessed scRNA-seq data
- Random seed list (e.g. 1111, 2222, ..., 10000) shared with the SDDA stage

## Outputs

- K=100 K-means basic clusterings of the cells, each with a randomly chosen k in [bcsLowK, bcsUpK]
- Cross-affinity matrix B of cell-to-cluster memberships (bij=1 if cell zi belongs to cluster Cj, else 0) consumed by BGEC.m

## How to apply

Load the latent embedding matrix produced by the single-cell dual denoising autoencoder (SDDA.py) as the input representation of the cells. Run K-means K=100 times via the MATLAB function BC.m, invoking it as 'BC <dataname> 100 <kmin> <kmax> [seeds]'; each basic clustering's k is randomly chosen between bcsLowK (minimum number of clusters, e.g. 2) and bcsUpK (maximum, e.g. 40 in the README example, up to 60 in the described ensemble), with random seeds matching those used in SDDA.py to ensure fairness. The deliberately varied k values produce base partitions of different granularity, which is what gives the ensemble the diversity exploited downstream. Each base partition yields a cell-to-cluster membership that is recorded in the cross-affinity matrix B (bij=1 if cell zi belongs to cluster Cj, else 0), which BGEC.m consumes when constructing the bipartite graph G={Z, phi, B} over cells and basic clusters.

## Related tools

- **BC.m** (Main function of the third module of scBGEDA; runs the ensemble of K-means basic clusterings on the latent representation and drives the bipartite graph ensemble clustering) — https://github.com/wangyh082/scBGEDA
- **BGEC.m** (Implements Bipartite Graph Ensemble Clustering; consumes the basic clusterings as cell-to-cluster membership matrix B to build the bipartite graph G) — https://github.com/wangyh082/scBGEDA
- **MATLAB** (Execution environment in which BC.m and BGEC.m are invoked via 'matlab -r' with the dataset name, ensemble size, k bounds and seeds)
- **SDDA.py** (Produces the latent embedding Z on which the K-means basic clusterings are computed) — https://github.com/wangyh082/scBGEDA

## Examples

```
matlab -r "BC 'Adam' 100 2 40 [1111, 2222, 3333, 4444, 5555, 6666, 7777, 8888, 9999, 10000]"
```

## Evaluation signals

- Exactly K=100 base partitions are generated (ensemble size parameter m=100 in the BC invocation).
- Each basic clustering's k falls within [bcsLowK, bcsUpK] (e.g. 2 to 40 in the README example; up to 60 as described in the ensemble), with multiple distinct k values across the ensemble.
- Every cell in the dataset is assigned to exactly one cluster in each basic clustering, so the resulting cross-affinity matrix B is a binary matrix with one membership per cell per base partition.
- The random seed list passed to BC matches the seeds used in SDDA.py (e.g. 1111, 2222, 3333, 4444, 5555, 6666, 7777, 8888, 9999, 10000) to keep the ensemble consistent with the SDDA stage.
- The final consensus labels downstream must assign exactly one cluster label to every cell and be evaluable by rand_index.m and computeNMI.m for ARI/NMI scoring against ground truth.

## Limitations

- The ensemble parameters (m, bcsLowK, bcsUpK) must be chosen by the user; inappropriate bounds (e.g. bcsUpK too small) would reduce base-partition diversity.
- Requires MATLAB for the BC.m/BGEC.m stage, which is separate from the Python (tensorflow 2.6.0 / keras 2.6.0) environment used for SDDA.py.
- Random seeds must be coordinated between the SDDA.py and BC stages; mismatched seeds break the fairness of the pipeline.

## Evidence

- [other] K=100 K-means basic clusterings (with each k randomly chosen between kmin=2 and kmax=60) on the latent representation Z: [evidence span withheld — anchor status: not_found]
- [readme] matlab -r "BC 'Adam' 100 2 40 [1111, 2222, 3333, 4444, 5555, 6666, 7777, 8888, 9999, 10000]": "matlab -r "BC 'Adam' 100 2 40 [1111, 2222, 3333, 4444, 5555, 6666, 7777, 8888, 9999, 10000]""
- [abstract] the bipartite graph ensemble clustering algorithm is designed to exploit the relationships between cells and the learned latent embedded space by means of a graph-based consensus function: "a bipartite graph ensemble clustering algorithm is designed to exploit the relationships between cells and the learned latent embedded space by means of a graph-based consensus function"
- [supplementary] BC.m | Main function of the third module of scBGEDA: "BC.m         | Main function of the third module of scBGEDA"
- [readme] random seed is used to select the cells randomly to ensure the fairness: "--random_seed RANDOM_SEED The random seeds which are used to select the cells randomly to ensure the fairness"
