---
name: dimensionality-reduction-ablation-comparison
description: Use when a single-cell deep clustering pipeline has two modules (a latent
  embedding and an ensemble/consensus clustering step) and the goal is to test whether
  each module independently improves clustering on labeled scRNA-seq datasets.
when_to_use_negative:
- The goal is benchmarking scBGEDA against other state-of-the-art clustering methods
  rather than testing the contribution of its own components — that is a different
  experiment design.
- The scRNA-seq datasets lack ground-truth cell labels, since NMI (computeNMI.m) and
  ARI (rand_index.m) cannot be computed.
- The pipeline being ablated has only one module, so there is no latent-embedding
  vs consensus-clustering split to perturb.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scBGEDA
  role: Source pipeline whose dual denoising autoencoder and bipartite graph ensemble
    clustering modules are ablated
  repo: https://github.com/wangyh082/scBGEDA
- name: SDDA.py
  role: Produces the latent embeddings whose alternatives (PCA/FA/UMAP) are compared
    in the ablation
  repo: https://github.com/wangyh082/scBGEDA
- name: BGEC.m
  role: Bipartite graph ensemble clustering run on each alternative 25-dim reduction
    as the ablated/ablation third-module setting
  repo: https://github.com/wangyh082/scBGEDA
- name: computeNMI.m
  role: Computes NMI values after clustering for each ablation setting
  repo: https://github.com/wangyh082/scBGEDA
- name: rand_index.m
  role: Computes ARI values after clustering for each ablation setting
  repo: https://github.com/wangyh082/scBGEDA
- name: Scanpy
  role: Provides Leiden and Louvain community detection used to cluster the latent
    space as the alternative-clustering ablation settings
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad075
    title: 'scBGEDA: deep single-cell clustering analysis via a dual denoising autoencoder
      with bipartite graph ensemble clustering'
  source_content_hash: sha256:343e54c8b808e96e2a0523eac22a4726c4f9789a26a6f0e8dd23ee873ed5473a
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
  - rand_index.m
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

# dimensionality-reduction-ablation-comparison

## Summary

Component-ablation protocol for scBGEDA-style deep single-cell clustering: replace the dual denoising autoencoder's latent space (25-dim PCA/FA/UMAP) or the bipartite graph ensemble clustering (Leiden/Louvain) and quantify each component's contribution via NMI/ARI/ASW/cLISI against ground-truth cell labels.

## When to use

Use when a single-cell deep clustering pipeline has two modules (a latent embedding and an ensemble/consensus clustering step) and the goal is to test whether each module independently improves clustering on labeled scRNA-seq datasets.

## When NOT to use

- The goal is benchmarking scBGEDA against other state-of-the-art clustering methods rather than testing the contribution of its own components — that is a different experiment design.
- The scRNA-seq datasets lack ground-truth cell labels, since NMI (computeNMI.m) and ARI (rand_index.m) cannot be computed.
- The pipeline being ablated has only one module, so there is no latent-embedding vs consensus-clustering split to perturb.

## Inputs

- Trained latent embeddings from SDDA.py (single-cell dual denoising autoencoder, e.g. --dims "256, 32" on 2000 highly variable genes)
- Raw/processed scRNA-seq datasets and ground-truth cell labels from the figshare deposit (https://figshare.com/articles/software/scBGEDA/19657911)
- MATLAB bipartite graph ensemble clustering code (BC.m / BGEC.m)
- Random seed list used in the original runs (1111, 2222, 3333, 4444, 5555, 6666, 7777, 8888, 9999, 10000)

## Outputs

- Per-setting clustering metrics (NMI, ARI, ASW, cLISI) for Leiden, Louvain, PCA-, FA-, and UMAP-based BGEC variants and the full scBGEDA pipeline
- Aggregated comparison table (ablation_metrics.csv) mirroring Fig 6 and Supplementary Tables S23–S24
- Median ARI and NMI values as reported by the BC.m output

## How to apply

Load the trained single-cell dual denoising autoencoder latent embeddings (produced by SDDA.py, e.g. for dataset 'Adam' with --dims "256, 32" and 2000 highly variable genes) plus the scRNA-seq data from the figshare deposit. Build four ablation settings: (1) cluster the latent space directly with Leiden community detection, (2) cluster it with Louvain (both via scanpy), (3) generate 25-dimensional PCA, factor-analysis (FA), and UMAP reductions and run the bipartite graph ensemble clustering module (BC.m calling BGEC.m in MATLAB) on each, and (4) keep the full pipeline as the upper reference. Score every setting against ground-truth cell labels using NMI (computeNMI.m) and ARI (rand_index.m) plus ASW and cLISI, reusing the original random seeds 1111–10000 so runs are comparable. Aggregate the settings' metrics into one table (ablation_metrics.csv) mirroring Fig 6 and Supplementary Tables S23–S24, and interpret co-performance as evidence that the autoencoder and the ensemble clustering act synergistically rather than redundantly.

## Related tools

- **scBGEDA** (Source pipeline whose dual denoising autoencoder and bipartite graph ensemble clustering modules are ablated) — https://github.com/wangyh082/scBGEDA
- **SDDA.py** (Produces the latent embeddings whose alternatives (PCA/FA/UMAP) are compared in the ablation) — https://github.com/wangyh082/scBGEDA
- **BGEC.m** (Bipartite graph ensemble clustering run on each alternative 25-dim reduction as the ablated/ablation third-module setting) — https://github.com/wangyh082/scBGEDA
- **computeNMI.m** (Computes NMI values after clustering for each ablation setting) — https://github.com/wangyh082/scBGEDA
- **rand_index.m** (Computes ARI values after clustering for each ablation setting) — https://github.com/wangyh082/scBGEDA
- **Scanpy** (Provides Leiden and Louvain community detection used to cluster the latent space as the alternative-clustering ablation settings)

## Examples

```
matlab -r "BC 'Adam' 100 2 40 [1111, 2222, 3333, 4444, 5555, 6666, 7777, 8888, 9999, 10000]"
```

## Evaluation signals

- Reproduced NMI/ARI for each ablation setting match the values in Supplementary Tables S23–S24 (output_matches_reference), with the full scBGEDA pipeline as the upper reference.
- All four ablation settings are computed on the same datasets with the same seed list (1111–10000) as the original BC.m invocation.
- BC.m output reports median ARI and NMI values, matching the README's stated output explanation.
- Each alternative reduction fed to BGEC.m is the specified 25 dimensions (PCA, FA, UMAP), not the full latent dimensionality (32 in the Adam example).
- Both alternative clusterings (Leiden and Louvain) and all three alternative reductions (PCA, FA, UMAP) appear in the final ablation_metrics.csv.

## Limitations

- The synergistic use of the single-cell dual denoising autoencoder and the bipartite graph ensemble clustering often but not always enhances clustering performance — some ablation settings may match or exceed the full pipeline on individual datasets.
- The README/abstract only name fifteen scRNA-seq datasets from different sequencing platforms without enumerating them; the 20-dataset set and per-dataset metric values must be taken from the supplementary tables.
- The ensemble-clustering ablation requires MATLAB (BC.m/BGEC.m) in addition to the Python environment (tensorflow 2.6.0, keras 2.6.0, scanpy 1.7.2), so partial environments cannot complete all settings.

## Evidence

- [abstract] a bipartite graph ensemble clustering algorithm is designed to exploit the relationships between cells and the learned latent embedded space by means of a graph-based consensus function: "a bipartite graph ensemble clustering algorithm is designed to exploit the relationships between cells and the learned latent embedded space by means of a graph-based consensus function"
- [supplementary] rand_index.m | Computing ARI values after clustering: "rand_index.m | Computing ARI values after clustering"
- [readme] The final output reports the clustering performance and the median values of ARI and NMI is provided, respectively.: "The final output reports the clustering performance and the median values of ARI and NMI is provided, respectively."
- [supplementary] The real single-cell RNA-seq datasets and source codes for our analyses are freely available at (https://figshare.com/articles/software/scBGEDA/19657911).: "The real single-cell RNA-seq datasets and source codes for our analyses are freely available at (https://figshare.com/articles/software/scBGEDA/19657911)."
