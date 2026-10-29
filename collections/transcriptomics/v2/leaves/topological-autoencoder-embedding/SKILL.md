---
name: topological-autoencoder-embedding
description: Use when single-cell RNA/ATAC/ADT modalities must be reduced to a structure-preserving
  latent space (e.g., PCA-reduced PBMC RNA, LSI-reduced PBMC ATAC, BM RNA/ADT matrices)
  before manifold alignment or benchmarking projection methods.
when_to_use_negative:
- The data already has a latent embedding and only modality alignment is needed —
  use run_scTopoGAN directly on existing manifolds
- You need cell-type labels or supervised structure in the projection — topoAE is
  an unsupervised, per-modality embedding method
- Modalities are already low-dimensional in a shared space — computing a new topoAE
  embedding discards it
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Topological Autoencoder
  role: Core method invoked via get_TopoAE_Embeddings to obtain the low-dimensional
    manifold of each dataset separately
  repo: https://github.com/AkashCiel/scTopoGAN
- name: scTopoGAN
  role: Package providing scTopoGAN_Functions.py (get_TopoAE_Embeddings) and TopoAE_benchmark.py
    for the projection benchmark
  repo: https://github.com/AkashCiel/scTopoGAN
- name: UMAP
  role: Non-autoencoder baseline compared against topoAE in TopoAE_benchmark.py
- name: Python
  role: Implementation language of the repository scripts (TopoAE_benchmark.py, scTopoGAN_Functions.py)
  repo: https://github.com/AkashCiel/scTopoGAN
provenance:
  source_task_ids:
  - task_004
  - task_003
  - task_001
  source_papers:
  - doi: 10.1093/bioadv/vbad171
    title: 'scTopoGAN: unsupervised manifold alignment of single-cell data'
  source_content_hash: sha256:a2c162e51adb03e211ea06c194d5d110cdb25c88cb3bc336499d4bd9fd5fb474
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/AkashCiel/scTopoGAN
  tools:
  - MMD-MA
  - Python
  - SCOTv2
  - Topological Autoencoder
  - UMAP
  - UnionCom
  - scTopoGAN
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# topological-autoencoder-embedding

## Summary

Computes low-dimensional manifold embeddings (8-dimensional latents) of single-cell modalities using a Topological Autoencoder (topoAE), the first stage of the scTopoGAN pipeline before topology-guided GAN alignment. topoAE better preserves original data structure in low-dimensional representations than regular AE, VAE, and UMAP.

## When to use

Use when single-cell RNA/ATAC/ADT modalities must be reduced to a structure-preserving latent space (e.g., PCA-reduced PBMC RNA, LSI-reduced PBMC ATAC, BM RNA/ADT matrices) before manifold alignment or benchmarking projection methods.

## When NOT to use

- The data already has a latent embedding and only modality alignment is needed — use run_scTopoGAN directly on existing manifolds
- You need cell-type labels or supervised structure in the projection — topoAE is an unsupervised, per-modality embedding method
- Modalities are already low-dimensional in a shared space — computing a new topoAE embedding discards it

## Inputs

- Pre-processed scRNA-seq data reduced by PCA to 50 components
- Pre-processed scATAC-seq data reduced by LSI to 50 components
- Pre-processed BM RNA/ADT matrices
- Data files provided in the scTopoGAN repository's Data folder

## Outputs

- 8-dimensional latent embeddings per modality (savable as .npy files)
- Benchmark metrics table (topoae_benchmark_results.csv) with KL0.01, KL0.1, KL1, and Silhouette columns per dataset x method
- Low-dimensional manifolds consumed as input to run_scTopoGAN

## How to apply

Load pre-processed single-cell data from the Data folder (PBMC RNA reduced by PCA to 50 components, PBMC ATAC reduced by LSI to 50 components, BM RNA/ADT matrices). For each modality separately, train the topological autoencoder with an 8-dimensional latent space using the architectures specified in Methods 2.7, via the get_TopoAE_Embeddings function in scTopoGAN_Functions.py. Keep the RNA embedding fixed as the alignment target when this feeds run_scTopoGAN. To validate projection quality, benchmark against regular Autoencoder, VAE, and UMAP using the TopoAE_benchmark.py script, computing KL divergence-based density-preservation scores (KL0.01, KL0.1, KL1) and Silhouette scores. The rationale is that topology preservation retains the original data structure in the low-dimensional representation, which downstream topology-guided GAN alignment depends on.

## Related tools

- **Topological Autoencoder** (Core method invoked via get_TopoAE_Embeddings to obtain the low-dimensional manifold of each dataset separately) — https://github.com/AkashCiel/scTopoGAN
- **scTopoGAN** (Package providing scTopoGAN_Functions.py (get_TopoAE_Embeddings) and TopoAE_benchmark.py for the projection benchmark) — https://github.com/AkashCiel/scTopoGAN
- **UMAP** (Non-autoencoder baseline compared against topoAE in TopoAE_benchmark.py)
- **Python** (Implementation language of the repository scripts (TopoAE_benchmark.py, scTopoGAN_Functions.py)) — https://github.com/AkashCiel/scTopoGAN

## Examples

```
python TopoAE_benchmark.py  # trains topoAE/AE/VAE and computes UMAP on pre-processed data in the Data folder
```

## Evaluation signals

- topoae_benchmark_results.csv exists and contains KL0.01, KL0.1, KL1, and Silhouette fields for all four datasets and all four methods (topoAE, VAE, AE, UMAP)
- All Silhouette values lie within [-1, 1]
- topoAE achieves the lowest KL0.01 across datasets (expected exception: PBMC ATAC, where VAE performs better), matching Table 1
- Latent embeddings are 8-dimensional per modality and savable as .npy files
- Downstream run_scTopoGAN alignment quality (Celltype matching, batch LISI) depends on the topoAE embeddings remaining fixed for the target modality

## Limitations

- For PBMC ATAC, VAE preserved density estimates better than topoAE (higher KL0.01 for topoAE), so topoAE is not uniformly best across modalities
- UMAP obtained the highest Silhouette scores, so Silhouette alone does not justify choosing topoAE
- Hyperparameters follow Methods 2.7 architectures; deviating from them may change the reported benchmark behavior

## Evidence

- [abstract] topoAE obtains latent representations of each modality separately: "We use topological autoencoders (topoAE) to obtain latent representations of each modality separately."
- [methods] get_TopoAE_Embeddings obtains the low-dimensional manifold of each dataset: "```get_TopoAE_Embeddings``` which obtains the low-dimensional manifold of each dataset separately using Topological Autoencoder"
- [readme] TopoAE_benchmark.py compares topoAE with AE, VAE and UMAP on pre-processed Data-folder inputs: "The ```TopoAE_benchmark.py``` script compares the performance of the Topological Autoencoder with regular Autoencoder, Variational Autoencoder and UMAP."
- [abstract] topoAE showed better structure preservation than other manifold projection methods: "the topoAE for individual modalities also showed better performance in preserving the original structure of the data in the low-dimensional representations when compared to other manifold projection"
