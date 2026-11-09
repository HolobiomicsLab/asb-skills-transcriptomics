---
name: topology-guided-gan-training
description: Use when two single-cell modalities must be aligned into a common latent
  space without overlapping cells or features, using topology-guided GAN training
  on separately computed topoAE manifolds.
when_to_use_negative:
- The modalities share overlapping cells or features — current integration methods
  that rely on overlap are applicable and this fully unsupervised alignment is unnecessary
- You need only a low-dimensional projection of a single dataset, not cross-modality
  alignment — use get_TopoAE_Embeddings / TopoAE_benchmark.py instead
- Input is raw counts rather than pre-processed data in the Data folder format, which
  the scTopoGAN scripts are not set up to consume directly
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scTopoGAN
  role: Topology-guided GAN that aligns the source topoAE manifold to the target manifold;
    run via run_scTopoGAN / Run_scTopoGAN.py
  repo: github.com/AkashCiel/scTopoGAN
- name: Topological Autoencoder
  role: Computes the low-dimensional manifold (topoAE embeddings) of each dataset
    separately as GAN input via get_TopoAE_Embeddings
  repo: github.com/AkashCiel/scTopoGAN
- name: MMD-MA
  role: Benchmarking method applied via manifold_align_mmd_pytorch.py to compare alignment
    performance against scTopoGAN
  repo: github.com/AkashCiel/scTopoGAN
- name: UnionCom
  role: Benchmarking method applied via UnionCom.py to compare alignment performance
    against scTopoGAN
  repo: github.com/AkashCiel/scTopoGAN
- name: Python
  role: Implementation language; scTopoGAN_Functions.py contains all necessary functions
  repo: github.com/AkashCiel/scTopoGAN
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioadv/vbad171
    title: 'scTopoGAN: unsupervised manifold alignment of single-cell data'
  source_content_hash: sha256:6129be493f1c01b1e7cdfd785532412481f6d3158b98b01fc0a4e3dfa3d8151e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MMD-MA
  - Python
  - Topological Autoencoder
  - UnionCom
  - scTopoGAN
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 15
---

# topology-guided-gan-training

## Summary

Train scTopoGAN, a topology-guided Generative Adversarial Network that aligns a source single-cell manifold (e.g., ATAC topoAE embeddings) onto a target manifold (e.g., RNA topoAE embeddings) in a fully unsupervised setting with non-overlapping cells or features.

## When to use

Use when two single-cell modalities must be aligned into a common latent space without overlapping cells or features, using topology-guided GAN training on separately computed topoAE manifolds.

## When NOT to use

- The modalities share overlapping cells or features — current integration methods that rely on overlap are applicable and this fully unsupervised alignment is unnecessary
- You need only a low-dimensional projection of a single dataset, not cross-modality alignment — use get_TopoAE_Embeddings / TopoAE_benchmark.py instead
- Input is raw counts rather than pre-processed data in the Data folder format, which the scTopoGAN scripts are not set up to consume directly

## Inputs

- Pre-processed source modality single-cell dataset (e.g., PBMC ATAC) as provided in the repository's Data folder
- Pre-processed target modality single-cell dataset (e.g., PBMC RNA) from the Data folder
- topoAE latent manifold of each modality computed via get_TopoAE_Embeddings
- Random initialization seed(s) for GAN training

## Outputs

- Aligned source manifold projected into the target (common) space
- Per-epoch topological loss values (Eq. 2) at checkpoint epochs (e.g., 500, 600, 700, 800, 900, 1000)
- Celltype and Subcelltype label-matching scores between aligned source and target manifolds
- Cross-run summary statistics such as Pearson correlations saved to gan_stability_correlations.csv

## How to apply

First compute a low-dimensional manifold for each modality separately with get_TopoAE_Embeddings (Topological Autoencoder), since topoAE preserves the original data structure better than regular Autoencoder, Variational Autoencoder, or UMAP. Then call run_scTopoGAN to align the source manifold to the target manifold with the topology-guided GAN; e.g., in the PBMC Full dataset task, train for 1000 epochs and inspect checkpoints from epoch 500 every 100 epochs. The GAN's topology preservation loss (Eq. 2) penalizes distortions of the source topology after projection into target space, so lower topological loss should coincide with better alignment. Assess alignment via Celltype and Subcelltype label-matching scores between the aligned source and target manifolds; across 40 random GAN initializations on Full PBMC, topological loss correlates with these scores at Pearson r ≈ −0.73 and −0.67. Benchmark results against UnionCom and MMD-MA using the provided UnionCom.py and manifold_align_mmd_pytorch.py scripts.

## Related tools

- **scTopoGAN** (Topology-guided GAN that aligns the source topoAE manifold to the target manifold; run via run_scTopoGAN / Run_scTopoGAN.py) — github.com/AkashCiel/scTopoGAN
- **Topological Autoencoder** (Computes the low-dimensional manifold (topoAE embeddings) of each dataset separately as GAN input via get_TopoAE_Embeddings) — github.com/AkashCiel/scTopoGAN
- **MMD-MA** (Benchmarking method applied via manifold_align_mmd_pytorch.py to compare alignment performance against scTopoGAN) — github.com/AkashCiel/scTopoGAN
- **UnionCom** (Benchmarking method applied via UnionCom.py to compare alignment performance against scTopoGAN) — github.com/AkashCiel/scTopoGAN
- **Python** (Implementation language; scTopoGAN_Functions.py contains all necessary functions) — github.com/AkashCiel/scTopoGAN

## Examples

```
python Run_scTopoGAN.py
```

## Evaluation signals

- Across 40 random GAN initializations on Full PBMC, Pearson correlation between topological loss trajectory and Celltype matching should be ≈ −0.73 (and ≈ −0.67 for Subcelltype), confirming lower loss accompanies better alignment
- Topological loss (Eq. 2) measured between the source manifold and its projected output should decrease from epoch 500 to 1000 as label-matching scores rise
- Alignment performance should compare favorably to UnionCom and MMD-MA benchmarks, since scTopoGAN outperforms state-of-the-art manifold alignment methods in complete unsupervised settings
- Input topoAE embeddings should preserve the original data structure better than regular Autoencoder, Variational Autoencoder, or UMAP alternatives
- Alignment must be achieved without overlapping cells or features between the two modalities

## Limitations

- The repository provides no changelog, so version drift in functions like run_scTopoGAN cannot be tracked from the repo alone
- Scripts expect pre-processed data already provided in the Data folder; raw inputs require prior preprocessing to match this format
- GAN results vary with random initialization — stability must be assessed across multiple runs (e.g., 40) rather than from a single run

## Evidence

- [abstract] We present scTopoGAN, a method for unsupervised manifold alignment of single-cell datasets with non-overlapping cells or features.: "We present scTopoGAN, a method for unsupervised manifold alignment of single-cell datasets with non-overlapping cells or features."
- [abstract] A topology-guided Generative Adversarial Network then aligns these latent representations into a common space.: "A topology-guided Generative Adversarial Network then aligns these latent representations into a common space."
- [methods] ```run_scTopoGAN``` which aligns the source manifold to the target manifold using the topology-guided GAN model.: "```run_scTopoGAN``` which aligns the source manifold to the target manifold using the topology-guided GAN model."
- [readme] The ```Run_scTopoGAN.py``` script provides an example for running scTopoGAN on a new data.: "The ```Run_scTopoGAN.py``` script provides an example for running scTopoGAN on a new data."
