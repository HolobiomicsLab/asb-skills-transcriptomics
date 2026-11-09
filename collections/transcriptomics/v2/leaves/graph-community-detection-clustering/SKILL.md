---
name: graph-community-detection-clustering
description: Use when you have a trained SDDA latent embedding of scRNA-seq data (e.g.
  256-32 dims, 2000 highly variable genes) and need cell-type clusters via graph-based
  consensus or community detection, or when ablating the bipartite graph ensemble
  clustering component of scBGEDA.
when_to_use_negative:
- Input is raw scRNA-seq counts rather than a trained latent embedding — preprocess.py
  and SDDA.py must run first to obtain the feature representations the graph operates
  on.
- A single canonical clustering with no ensemble consensus is required and the full
  scBGEDA pipeline (not an ablation of the bipartite graph module) is the target.
- Cell labels are unknown and no ground truth exists — NMI/ARI evaluation via computeNMI.m/rand_index.m
  cannot be performed.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scBGEDA
  role: provides the bipartite graph ensemble clustering module (BC.m / BGEC.m) that
    builds a graph between cells and the learned latent embedded space to derive a
    graph-based consensus clustering
  repo: https://github.com/wangyh082/scBGEDA
- name: Scanpy
  role: supplies Leiden and Louvain community detection used to cluster the learned
    latent embedded space as ablation alternatives
- name: MATLAB
  role: runtime for BC.m and BGEC.m; invoked with ensemble size and cluster-number
    bounds (e.g. m=100, K from 2 to 40) and the SDDA random seeds
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad075
    title: 'scBGEDA: deep single-cell clustering analysis via a dual denoising autoencoder
      with bipartite graph ensemble clustering'
  source_content_hash: sha256:bd126caf12f6f59c7682eb89167affc44d86dcd91525cfc323d666e8700f8811
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wangyh082/scBGEDA
  tools:
  - BC.m
  - BGEC.m
  - MATLAB
  - Python
  - SDDA.py
  - Scanpy
  - computeNMI.m
  - scBGEDA
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5333333333333333
n_claims: 15
---

# graph-community-detection-clustering

## Summary

Clusters single-cell latent embeddings by community detection (Leiden/Louvain in scanpy) and by the bipartite graph ensemble clustering algorithm (BC.m/BGEC.m), which builds a graph between cells and the learned latent embedded space and derives a graph-based consensus partition, evaluated with NMI and ARI.

## When to use

Use when you have a trained SDDA latent embedding of scRNA-seq data (e.g. 256-32 dims, 2000 highly variable genes) and need cell-type clusters via graph-based consensus or community detection, or when ablating the bipartite graph ensemble clustering component of scBGEDA.

## When NOT to use

- Input is raw scRNA-seq counts rather than a trained latent embedding — preprocess.py and SDDA.py must run first to obtain the feature representations the graph operates on.
- A single canonical clustering with no ensemble consensus is required and the full scBGEDA pipeline (not an ablation of the bipartite graph module) is the target.
- Cell labels are unknown and no ground truth exists — NMI/ARI evaluation via computeNMI.m/rand_index.m cannot be performed.

## Inputs

- SDDA.py latent embeddings (e.g. 32-dim dual denoising autoencoder output) for datasets such as 'Adam' from the figshare deposit
- corresponding scRNA-seq expression matrix and ground-truth cell labels
- MATLAB BC.m/BGEC.m module and its parameter set (dataset name, m, bcsLowK, bcsUpK, random seeds)

## Outputs

- cell cluster assignments (consensus partition from BGEC.m or Leiden/Louvain communities)
- NMI and ARI values computed via computeNMI.m and rand_index.m
- clustering performance report with median ARI and NMI (BC.m output)

## How to apply

Load the latent embeddings produced by SDDA.py (trained with e.g. --dims "256, 32", --highly_genes 2000, seeds 1111-10000) and the corresponding scRNA-seq data from the figshare deposit. For the graph-ensemble path, run the MATLAB module BC.m, which invokes BGEC.m to exploit relationships between cells and the learned latent embedded space via a graph-based consensus function; parameters are the dataset name, ensemble size m (e.g. 100), and the minimum (e.g. 2) and maximum (e.g. 40) numbers of clusters for the basic clusterings, using the same random seeds as SDDA.py. For the community-detection ablation path, cluster the same latent space with Leiden and Louvain community detection in scanpy (version 1.7.2 in the reference environment). Score every resulting partition against ground-truth cell labels with computeNMI.m (NMI) and rand_index.m (ARI); BC.m itself reports the clustering performance and the median values of ARI and NMI.

## Related tools

- **scBGEDA** (provides the bipartite graph ensemble clustering module (BC.m / BGEC.m) that builds a graph between cells and the learned latent embedded space to derive a graph-based consensus clustering) — https://github.com/wangyh082/scBGEDA
- **Scanpy** (supplies Leiden and Louvain community detection used to cluster the learned latent embedded space as ablation alternatives)
- **MATLAB** (runtime for BC.m and BGEC.m; invoked with ensemble size and cluster-number bounds (e.g. m=100, K from 2 to 40) and the SDDA random seeds)

## Examples

```
matlab -r "BC 'Adam' 100 2 40 [1111, 2222, 3333, 4444, 5555, 6666, 7777, 8888, 9999, 10000]"
```

## Evaluation signals

- Reproduced NMI (computeNMI.m) and ARI (rand_index.m) for each ablation setting match Supplementary Tables S23-S24, with the full scBGEDA pipeline as the upper reference
- BC.m reports median ARI and NMI over the seed set 1111, 2222, ..., 10000; medians should be stable across the ensemble size m=100 and K range 2-40
- Ablation comparison table (ablation_metrics.csv) contains all four settings (Leiden, Louvain, PCA/FA/UMAP + BC) plus full-pipeline reference metrics
- Consensus partition cluster count falls within the specified bcsLowK-bcsUpK bounds (e.g. 2-40)

## Limitations

- The synergy of the dual denoising autoencoder and the bipartite graph ensemble clustering often but not always enhances clustering performance; each component's contribution varies by dataset, so gains over community-detection-only clustering are not guaranteed.
- The bipartite graph module requires MATLAB and a specific environment (scanpy 1.7.2, jgraph 0.2.1, tensorflow 2.6.0, python 3.6); version mismatches may break reproduction.
- Clustering quality depends on the trained latent embedding; poor SDDA training (e.g. wrong --highly_genes, --noise_sd, or seed set) degrades the graph-based consensus.

## Evidence

- [readme] readme: "a bipartite graph ensemble clustering algorithm is designed to exploit the relationships between cells and the learned latent embedded space by means of a graph-based consensus function"
- [other] other: [evidence span withheld — anchor status: not_found]
