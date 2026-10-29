---
name: cross-modality-benchmark-table-generation
description: Use when each single-cell modality (e.g., PBMC RNA/ATAC, BM RNA/ADT)
  must be projected to a low-dimensional latent by multiple methods and compared in
  one table of KL0.01/KL0.1/KL1 and Silhouette metrics.
when_to_use_negative:
- Input is raw, unprocessed single-cell data — the benchmark script requires pre-processed
  data already reduced to 50 components (PCA for RNA, LSI for ATAC) as provided in
  the Data folder
- The goal is cross-modality alignment into a common space rather than per-modality
  projection benchmarking — use run_scTopoGAN instead of the TopoAE_benchmark.py evaluation
- Only one projection method is of interest and no method comparison table is required
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3474
tools:
- name: Topological Autoencoder
  role: Primary projection method under benchmark; obtains the 8-dimensional low-dimensional
    manifold of each modality via get_TopoAE_Embeddings
  repo: https://github.com/AkashCiel/scTopoGAN
- name: UMAP
  role: Non-AE baseline projection method included in the comparison table
- name: scTopoGAN
  role: Repository framework whose TopoAE_benchmark.py script performs the benchmark
    of topoAE against regular Autoencoder, Variational Autoencoder and UMAP
  repo: https://github.com/AkashCiel/scTopoGAN
- name: Python
  role: Implementation language of the TopoAE_benchmark.py benchmark script
  repo: https://github.com/AkashCiel/scTopoGAN
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioadv/vbad171
    title: 'scTopoGAN: unsupervised manifold alignment of single-cell data'
  source_content_hash: sha256:167769e990925e2a79e5c4f0a1811faa490c3bfd326c00e5bd147784bfc5eec4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/AkashCiel/scTopoGAN
  tools:
  - Python
  - Topological Autoencoder
  - UMAP
  - scTopoGAN
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# cross-modality-benchmark-table-generation

## Summary

Generates a benchmark table comparing manifold projection methods (topoAE, regular Autoencoder, Variational Autoencoder, UMAP) on single-cell modalities projected to 8-dimensional latents, scoring density preservation with KL divergence at regularization weights 0.01/0.1/1 and cell-class separation with Silhouette scores.

## When to use

Use when each single-cell modality (e.g., PBMC RNA/ATAC, BM RNA/ADT) must be projected to a low-dimensional latent by multiple methods and compared in one table of KL0.01/KL0.1/KL1 and Silhouette metrics.

## When NOT to use

- Input is raw, unprocessed single-cell data — the benchmark script requires pre-processed data already reduced to 50 components (PCA for RNA, LSI for ATAC) as provided in the Data folder
- The goal is cross-modality alignment into a common space rather than per-modality projection benchmarking — use run_scTopoGAN instead of the TopoAE_benchmark.py evaluation
- Only one projection method is of interest and no method comparison table is required

## Inputs

- Pre-processed PBMC RNA matrix reduced by PCA to 50 components (from the Data folder)
- Pre-processed PBMC ATAC matrix reduced by LSI to 50 components (from the Data folder)
- Pre-processed BM RNA and BM ADT matrices (from the Data folder)
- Cell class labels for Silhouette scoring

## Outputs

- topoae_benchmark_results.csv with rows = dataset x method and columns = KL0.01, KL0.1, KL1, Silhouette
- Trained 8-dimensional latent embeddings per modality saved as .npy files (topoAE, AE, VAE, UMAP)

## How to apply

Load the pre-processed data from the Data folder (PBMC RNA reduced by PCA to 50 components, PBMC ATAC reduced by LSI to 50 components, BM RNA/ADT matrices) as consumed by TopoAE_benchmark.py. Train topoAE, regular AE, and VAE separately on each modality with 8-dimensional latents using the architectures specified in Methods 2.7, and compute UMAP representations as the non-AE baseline. For every dataset-method pair, compute the KL divergence-based density-preservation scores at regularization weights 0.01, 0.1, and 1 (KL0.01, KL0.1, KL1) comparing high- versus low-dimensional pairwise-distance distributions, plus Silhouette scores of the low-dimensional embeddings. Write the full metrics table (rows = dataset x method, columns = KL0.01, KL0.1, KL1, Silhouette) to topoae_benchmark_results.csv and save each modality's trained 8-dimensional latent embeddings as .npy files.

## Related tools

- **Topological Autoencoder** (Primary projection method under benchmark; obtains the 8-dimensional low-dimensional manifold of each modality via get_TopoAE_Embeddings) — https://github.com/AkashCiel/scTopoGAN
- **UMAP** (Non-AE baseline projection method included in the comparison table)
- **scTopoGAN** (Repository framework whose TopoAE_benchmark.py script performs the benchmark of topoAE against regular Autoencoder, Variational Autoencoder and UMAP) — https://github.com/AkashCiel/scTopoGAN
- **Python** (Implementation language of the TopoAE_benchmark.py benchmark script) — https://github.com/AkashCiel/scTopoGAN

## Examples

```
python TopoAE_benchmark.py
```

## Evaluation signals

- topoae_benchmark_results.csv exists and contains KL0.01, KL0.1, KL1, and Silhouette fields for all four datasets (PBMC RNA, PBMC ATAC, BM RNA, BM ADT) and all four methods (topoAE, AE, VAE, UMAP)
- All Silhouette values in the table lie within [-1, 1]
- topoAE achieves the overall lowest KL0.01 value across datasets, except for PBMC ATAC where VAE performs better (matches the paper's reported pattern)
- UMAP obtains the highest Silhouette scores in the table
- Each modality's trained 8-dimensional latent embedding is saved as a .npy file

## Limitations

- topoAE's density-preservation advantage does not hold for all modalities: for PBMC ATAC the VAE achieved the better KL0.01 value
- UMAP attains the highest Silhouette scores, so the best method depends on whether density preservation (KL) or cell-class separation (Silhouette) is prioritized
- The benchmark requires pre-processed inputs (PCA-50 / LSI-50 reductions) supplied in the Data folder; it cannot consume raw counts directly

## Evidence

- [methods] TopoAE_benchmark.py compares the performance of the Topological Autoencoder with regular Autoencoder, Variational Autoencoder and UMAP: "```TopoAE_benchmark.py``` script compares the performance of the Topological Autoencoder with regular Autoencoder, Variational Autoencoder and UMAP"
- [readme] this script takes as input the pre-processed data provided in the Data folder: "Please note, this script takes as input the pre-processed data provided in the ```Data``` folder."
- [abstract] topoAE showed better performance in preserving the original structure of the data in low-dimensional representations: "the topoAE for individual modalities also showed better performance in preserving the original structure of the data in the low-dimensional representations"
- [readme] The Run_scTopoGAN.py script provides an example for running scTopoGAN on a new data.: "The ```Run_scTopoGAN.py``` script provides an example for running scTopoGAN on a new data."
