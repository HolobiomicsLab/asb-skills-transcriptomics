---
name: kl-density-preservation-scoring
description: Use when comparing dimensionality-reduction or manifold-projection methods
  on single-cell modalities and you need quantitative metrics of how well low-dimensional
  embeddings preserve high-dimensional pairwise-distance distributions.
when_to_use_negative:
- The goal is manifold alignment between two modalities into a shared space — use
  scTopoGAN's run_scTopoGAN instead of per-modality projection scoring.
- You only have a single clustering/embedding with no corresponding high-dimensional
  input from which to compute pairwise-distance distributions, so KL density-preservation
  scores cannot be computed.
- You need to compare manifold alignment methods (scTopoGAN vs UnionCom vs MMD-MA)
  rather than per-modality projection methods.
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: Topological Autoencoder
  role: Generates the topoAE 8-dimensional latent embeddings per modality that are
    scored for density preservation against AE, VAE, and UMAP baselines; get_TopoAE_Embeddings
    obtains the low-dimensional manifold of each dataset separately.
  repo: github.com/AkashCiel/scTopoGAN
- name: UMAP
  role: Serves as the non-Autoencoder baseline projection method whose low-dimensional
    representations are scored alongside topoAE, AE, and VAE.
  repo: github.com/AkashCiel/scTopoGAN
- name: Python
  role: Implementation language; the TopoAE_benchmark.py script computes the comparison
    and the KL/Silhouette metrics.
  repo: github.com/AkashCiel/scTopoGAN
- name: scTopoGAN
  role: The parent method whose per-modality topoAE embedding step (get_TopoAE_Embeddings)
    produces the latents evaluated by this benchmark.
  repo: github.com/AkashCiel/scTopoGAN
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioadv/vbad171
    title: 'scTopoGAN: unsupervised manifold alignment of single-cell data'
  source_content_hash: sha256:aa6e98115578793dcc0b1a14bb730d542d86eae0366d08cec47063be40ce978a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - Topological Autoencoder
  - UMAP
  - scTopoGAN
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# kl-density-preservation-scoring

## Summary

Benchmarks how well manifold projection methods (topoAE, VAE, AE, UMAP) preserve the structure of single-cell data in low-dimensional embeddings, using KL-divergence-based density scores at regularization weights 0.01, 0.1, and 1 (KL0.01, KL0.1, KL1) plus Silhouette scores.

## When to use

Use when comparing dimensionality-reduction or manifold-projection methods on single-cell modalities and you need quantitative metrics of how well low-dimensional embeddings preserve high-dimensional pairwise-distance distributions.

## When NOT to use

- The goal is manifold alignment between two modalities into a shared space — use scTopoGAN's run_scTopoGAN instead of per-modality projection scoring.
- You only have a single clustering/embedding with no corresponding high-dimensional input from which to compute pairwise-distance distributions, so KL density-preservation scores cannot be computed.
- You need to compare manifold alignment methods (scTopoGAN vs UnionCom vs MMD-MA) rather than per-modality projection methods.

## Inputs

- Pre-processed PBMC RNA matrix reduced by PCA to 50 components (from the Data folder)
- Pre-processed PBMC ATAC matrix reduced by LSI to 50 components (from the Data folder)
- Pre-processed BM RNA and BM ADT matrices (from the Data folder)
- Low-dimensional embeddings (8-dimensional latents) produced by topoAE, regular Autoencoder, Variational Autoencoder, and UMAP for each modality

## Outputs

- topoae_benchmark_results.csv containing a metrics table with rows = dataset x method and columns = KL0.01, KL0.1, KL1, Silhouette
- Per-modality trained 8-dimensional latent embeddings saved as .npy files

## How to apply

Load the pre-processed single-cell data from the Data folder as consumed by TopoAE_benchmark.py: PBMC RNA reduced by PCA to 50 components, PBMC ATAC reduced by LSI to 50 components, and BM RNA/ADT matrices. For each modality, generate 8-dimensional latent embeddings with each candidate method (topoAE, regular Autoencoder, Variational Autoencoder, UMAP). For every dataset x method, compute the KL divergence-based density-preservation scores at regularization weights 0.01, 0.1, and 1, comparing high-dimensional vs low-dimensional pairwise-distance distributions, and compute Silhouette scores of the low-dimensional embeddings. Write the full metrics table (rows = dataset x method, columns = KL0.01, KL0.1, KL1, Silhouette) to topoae_benchmark_results.csv and save each modality's trained 8-dimensional latent embeddings as .npy files. Interpretation guidance from the paper: lower KL values indicate better density preservation (topoAE achieved the overall lowest KL0.01 across datasets, except PBMC ATAC where VAE performed better), while UMAP obtained the highest Silhouette scores.

## Related tools

- **Topological Autoencoder** (Generates the topoAE 8-dimensional latent embeddings per modality that are scored for density preservation against AE, VAE, and UMAP baselines; get_TopoAE_Embeddings obtains the low-dimensional manifold of each dataset separately.) — github.com/AkashCiel/scTopoGAN
- **UMAP** (Serves as the non-Autoencoder baseline projection method whose low-dimensional representations are scored alongside topoAE, AE, and VAE.) — github.com/AkashCiel/scTopoGAN
- **Python** (Implementation language; the TopoAE_benchmark.py script computes the comparison and the KL/Silhouette metrics.) — github.com/AkashCiel/scTopoGAN
- **scTopoGAN** (The parent method whose per-modality topoAE embedding step (get_TopoAE_Embeddings) produces the latents evaluated by this benchmark.) — github.com/AkashCiel/scTopoGAN

## Examples

```
python TopoAE_benchmark.py  # consumes pre-processed data from the Data folder, writes topoae_benchmark_results.csv and .npy latent embeddings
```

## Evaluation signals

- topoae_benchmark_results.csv exists and contains KL0.01, KL0.1, KL1, and Silhouette fields for all four datasets (PBMC RNA, PBMC ATAC, BM RNA, BM ADT) and all four methods (topoAE, VAE, AE, UMAP)
- All Silhouette values lie within [-1, 1]
- KL scores are computed at all three regularization weights (0.01, 0.1, 1) comparing high-dimensional vs low-dimensional pairwise-distance distributions
- Each modality's trained 8-dimensional latent embedding is saved as a .npy file
- Expected pattern from the paper: topoAE achieves the overall lowest KL0.01 value across datasets, except for PBMC ATAC where VAE performs better; UMAP obtains the highest Silhouette scores

## Limitations

- The benchmark evaluates projections to a fixed 8-dimensional latent space using the architectures specified in Methods 2.7; results may differ for other latent dimensionalities or architectures.
- Density preservation (KL) and cell-class separation (Silhouette) can disagree — topoAE won on KL but UMAP won on Silhouette — so a single metric should not be treated as the overall winner criterion.
- No changelog found in the repository, limiting version traceability of the benchmarking code.

## Evidence

- [methods] Benchmark Topological Autoencoder against regular Autoencoder, Variational Autoencoder and UMAP: "The ```TopoAE_benchmark.py``` script compares the performance of the Topological Autoencoder with regular Autoencoder, Variational Autoencoder and UMAP"
- [abstract] topoAE better preserves the original data structure in low-dimensional representations: "the topoAE for individual modalities also showed better performance in preserving the original structure of the data in the low-dimensional representations when compared to other manifold projection"
- [readme] Benchmark script consumes pre-processed data from Data folder: "Please note, this script takes as input the pre-processed data provided in the ```Data``` folder."
- [abstract] topoAE latent representations per modality: "We use topological autoencoders (topoAE) to obtain latent representations of each modality separately."
