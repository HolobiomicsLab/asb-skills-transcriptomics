---
name: single-cell-latent-embedding-training
description: Use when per-modality low-dimensional representations must be produced
  and compared before manifold alignment, e.g. benchmarking topoAE against AE, VAE,
  and UMAP on PCA/LSI-reduced single-cell matrices.
when_to_use_negative:
- When the goal is cross-modality alignment of non-overlapping cells/features — use
  run_scTopoGAN (topology-guided GAN) instead of per-modality embedding benchmarking.
- When modalities already share overlapping features or cells, since conventional
  integration methods rather than topology-based embedding may suffice.
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_3170
tools:
- name: Topological Autoencoder
  role: obtains the low-dimensional manifold of each dataset separately via get_TopoAE_Embeddings;
    the benchmarked embedding method
  repo: https://github.com/AkashCiel/scTopoGAN
- name: scTopoGAN
  role: Python implementation containing scTopoGAN_Functions.py and the TopoAE_benchmark.py
    reproducibility script
  repo: https://github.com/AkashCiel/scTopoGAN
- name: UMAP
  role: non-AE baseline low-dimensional representation compared against topoAE, AE,
    and VAE
- name: Python
  role: implementation language for the TopoAE_benchmark.py embedding/benchmark workflow
  repo: https://github.com/AkashCiel/scTopoGAN
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioadv/vbad171
    title: 'scTopoGAN: unsupervised manifold alignment of single-cell data'
  source_content_hash: sha256:3f50b2c01ab089700a783ca489fee0080a963453319591a5e7e01d50d44eb823
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/AkashCiel/scTopoGAN
  tools:
  - Python
  - Topological Autoencoder
  - UMAP
  - scTopoGAN
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# single-cell-latent-embedding-training

## Summary

Train low-dimensional (8-dim) latent embeddings of single-cell modalities (PBMC RNA/ATAC, BM RNA/ADT) with topoAE, AE, VAE, and UMAP, then benchmark density preservation (KL0.01/KL0.1/KL1) and cell-class separation (Silhouette).

## When to use

Use when per-modality low-dimensional representations must be produced and compared before manifold alignment, e.g. benchmarking topoAE against AE, VAE, and UMAP on PCA/LSI-reduced single-cell matrices.

## When NOT to use

- When the goal is cross-modality alignment of non-overlapping cells/features — use run_scTopoGAN (topology-guided GAN) instead of per-modality embedding benchmarking.
- When modalities already share overlapping features or cells, since conventional integration methods rather than topology-based embedding may suffice.

## Inputs

- PBMC RNA matrix pre-reduced by PCA to 50 components (Data folder)
- PBMC ATAC matrix pre-reduced by LSI to 50 components (Data folder)
- BM RNA and BM ADT matrices (Data folder)
- Cell-class labels for Silhouette scoring

## Outputs

- topoae_benchmark_results.csv with rows = dataset x method and columns = KL0.01, KL0.1, KL1, Silhouette
- Trained 8-dimensional latent embeddings per modality saved as .npy files (topoAE, AE, VAE, UMAP)

## How to apply

Load the pre-processed data from the Data folder (PBMC RNA reduced by PCA to 50 components, PBMC ATAC reduced by LSI to 50 components, BM RNA and ADT matrices) as consumed by TopoAE_benchmark.py. Train topoAE, a regular autoencoder, and a VAE on each modality separately with 8-dimensional latents using the architectures specified in Methods 2.7, and compute UMAP representations as the non-AE baseline. For every dataset x method, compute KL divergence-based density-preservation scores at regularization weights 0.01, 0.1, and 1 (comparing high- vs low-dimensional pairwise-distance distributions) plus Silhouette scores of the embeddings. Write the metrics table to topoae_benchmark_results.csv (rows = dataset x method, columns = KL0.01, KL0.1, KL1, Silhouette) and save each modality's trained 8-dimensional embeddings as .npy files. The rationale is that topoAE's topology loss better preserves original data structure — it achieved the lowest KL0.01 overall (except PBMC ATAC, where VAE was better), while UMAP obtained the highest Silhouette scores.

## Related tools

- **Topological Autoencoder** (obtains the low-dimensional manifold of each dataset separately via get_TopoAE_Embeddings; the benchmarked embedding method) — https://github.com/AkashCiel/scTopoGAN
- **scTopoGAN** (Python implementation containing scTopoGAN_Functions.py and the TopoAE_benchmark.py reproducibility script) — https://github.com/AkashCiel/scTopoGAN
- **UMAP** (non-AE baseline low-dimensional representation compared against topoAE, AE, and VAE)
- **Python** (implementation language for the TopoAE_benchmark.py embedding/benchmark workflow) — https://github.com/AkashCiel/scTopoGAN

## Examples

```
python TopoAE_benchmark.py  # consumes pre-processed Data folder; writes topoae_benchmark_results.csv and .npy 8-dim embeddings
```

## Evaluation signals

- topoae_benchmark_results.csv exists and contains KL0.01, KL0.1, KL1, and Silhouette fields for all four datasets (PBMC RNA, PBMC ATAC, BM RNA, BM ADT) and all four methods (topoAE, VAE, AE, UMAP)
- All Silhouette values lie within [-1, 1]
- Each modality's trained 8-dimensional latent embeddings are saved as .npy files
- Expect topoAE to achieve the overall lowest KL0.01 across datasets, with VAE better for PBMC ATAC and UMAP achieving the highest Silhouette scores (Table 1 reproduction check)
- Input files match the pre-processed Data folder format: 50-component PCA for PBMC RNA and 50-component LSI for PBMC ATAC

## Limitations

- topoAE is not uniformly best: VAE preserved density better for PBMC ATAC, and UMAP always achieved the highest Silhouette scores, so metric choice (KL vs Silhouette) changes the winner
- The benchmark script requires the pre-processed data already present in the Data folder; it does not perform PCA/LSI preprocessing itself
- A late point in the benchmark the scripts only produce low-dimensional representations per modality — cross-modality alignment requires the separate run_scTopoGAN step

## Evidence

- [readme] topoAE benchmark: "The ```TopoAE_benchmark.py``` script compares the performance of the Topological Autoencoder with regular Autoencoder, Variational Autoencoder and UMAP."
- [readme] input data: "this script takes as input the pre-processed data provided in the ```Data``` folder."
- [methods] topoAE embeddings: "```get_TopoAE_Embeddings``` which obtains the low-dimensional manifold of each dataset separately using Topological Autoencoder"
- [abstract] topoAE rationale: "the topoAE for individual modalities also showed better performance in preserving the original structure of the data in the low-dimensional representations"
