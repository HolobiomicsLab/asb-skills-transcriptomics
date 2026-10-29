---
name: single-cell-manifold-alignment
description: Use when two single-modality single-cell datasets (e.g. ATAC source,
  RNA target) share no overlapping cells or features and must be aligned into a common
  latent space for label-matching or integration scoring.
when_to_use_negative:
- The datasets share overlapping features or cells — current integration methods that
  rely on such overlap may be more appropriate, and scTopoGAN's unsupervised design
  targets the non-overlapping regime
- Comparing manifold-projection methods rather than aligning two modalities — use
  TopoAE_benchmark.py (topoAE vs regular Autoencoder, Variational Autoencoder, UMAP)
  instead of the GAN alignment
- The ATAC modality retains fewer than ~30% of original cells in a stratified downsample
  — Celltype matching scores drop in this regime
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0102
tools:
- name: scTopoGAN
  role: Topology-guided GAN (run_scTopoGAN in scTopoGAN_Functions.py) that aligns
    the source manifold to the target manifold
  repo: https://github.com/AkashCiel/scTopoGAN
- name: Topological Autoencoder
  role: Obtains the low-dimensional manifold of each dataset separately via get_TopoAE_Embeddings
    before GAN alignment
  repo: https://github.com/AkashCiel/scTopoGAN
- name: UnionCom
  role: Benchmark baseline applied via UnionCom.py for comparison of alignment performance
  repo: https://github.com/AkashCiel/scTopoGAN
- name: MMD-MA
  role: Benchmark baseline applied via manifold_align_mmd_pytorch.py for comparison
    of alignment performance
  repo: https://github.com/AkashCiel/scTopoGAN
- name: Python
  role: Implementation language for all scTopoGAN functions and benchmark scripts
  repo: https://github.com/AkashCiel/scTopoGAN
provenance:
  source_task_ids:
  - task_002
  - task_004
  - task_007
  - task_003
  - task_006
  source_papers:
  - doi: 10.1093/bioadv/vbad171
    title: 'scTopoGAN: unsupervised manifold alignment of single-cell data'
  source_content_hash: sha256:42dcd6450a6c46c255cc0fd166ef118c159d0867d9c09e79708aeb4e3719c1ce
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/AkashCiel/scTopoGAN
  tools:
  - MMD-MA
  - Python
  - SCOTv2
  - Topological Autoencoder
  - UnionCom
  - scTopoGAN
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# single-cell-manifold-alignment

## Summary

Unsupervised alignment of single-cell modality manifolds (e.g. scRNA-seq and scATAC-seq) using scTopoGAN: topoAE-derived low-dimensional manifolds per modality, then a topology-guided GAN that maps the source manifold into the target space without requiring overlapping cells or features.

## When to use

Use when two single-modality single-cell datasets (e.g. ATAC source, RNA target) share no overlapping cells or features and must be aligned into a common latent space for label-matching or integration scoring.

## When NOT to use

- The datasets share overlapping features or cells — current integration methods that rely on such overlap may be more appropriate, and scTopoGAN's unsupervised design targets the non-overlapping regime
- Comparing manifold-projection methods rather than aligning two modalities — use TopoAE_benchmark.py (topoAE vs regular Autoencoder, Variational Autoencoder, UMAP) instead of the GAN alignment
- The ATAC modality retains fewer than ~30% of original cells in a stratified downsample — Celltype matching scores drop in this regime

## Inputs

- Pre-processed scRNA-seq (target) and scATAC-seq (source) datasets from the package's Data folder (e.g. Full PBMC, Partial PBMC, BM, scGEM)
- Per-modality low-dimensional manifolds computed with Topological Autoencoder (get_TopoAE_Embeddings)
- Cell-type and subcell-type labels for computing matching scores

## Outputs

- Aligned joint embeddings (source manifold projected into target space, saveable as .npy files)
- Celltype matching, Subcelltype matching, and batch LISI scores per method-dataset pair (e.g. Table-2-equivalent CSV)
- Topological loss and matching-score trajectories across GAN checkpoint epochs (e.g. gan_stability_correlations.csv)

## How to apply

First compute a separate low-dimensional manifold for each modality with the Topological Autoencoder via get_TopoAE_Embeddings, using the pre-processed data in the Data folder; topoAE preserves the original data structure better than a regular Autoencoder, Variational Autoencoder, or UMAP. Then run run_scTopoGAN to align the source (ATAC) manifold to the target (RNA) manifold with the topology-guided GAN; the full benchmark protocol selects from 20 candidate GAN models by topological loss and refines the best model for 1000 epochs, with topological loss and label-matching scores computable at checkpoints from epoch 500 onward every 100 epochs. The GAN's model-selection loop exploits the observation that topological loss between source and projected output is negatively correlated with alignment quality (Pearson r ≈ −0.73 for Celltype and −0.67 for Subcelltype matching over 40 random initializations). Judge alignment by Celltype and Subcelltype label-matching scores and batch LISI (optimal value 2); scTopoGAN's alignment is robust to stratified ATAC downsampling until only ~30% of cells remain, with increased score variability at ≥60% cell-type-specific removal.

## Related tools

- **scTopoGAN** (Topology-guided GAN (run_scTopoGAN in scTopoGAN_Functions.py) that aligns the source manifold to the target manifold) — https://github.com/AkashCiel/scTopoGAN
- **Topological Autoencoder** (Obtains the low-dimensional manifold of each dataset separately via get_TopoAE_Embeddings before GAN alignment) — https://github.com/AkashCiel/scTopoGAN
- **UnionCom** (Benchmark baseline applied via UnionCom.py for comparison of alignment performance) — https://github.com/AkashCiel/scTopoGAN
- **MMD-MA** (Benchmark baseline applied via manifold_align_mmd_pytorch.py for comparison of alignment performance) — https://github.com/AkashCiel/scTopoGAN
- **Python** (Implementation language for all scTopoGAN functions and benchmark scripts) — https://github.com/AkashCiel/scTopoGAN

## Examples

```
python Run_scTopoGAN.py  # loads pre-processed Data-folder inputs, calls get_TopoAE_Embeddings per modality, then run_scTopoGAN to align source ATAC manifold to target RNA manifold
```

## Evaluation signals

- Batch LISI on the aligned joint embedding approaches the optimal value of 2, and scTopoGAN's batch LISI should exceed UnionCom, MMD-MA, and SCOT on Full PBMC, Partial PBMC, and BM
- Pearson correlation between topological loss trajectory and Celltype/Subcelltype matching trajectories across 40 GAN initializations is negative, near r = −0.73 and −0.67 respectively
- Celltype matching score across 40 differently initialized GANs averages 37.1 ± 16.5% despite similar losses — a single run falling far outside this range flags instability
- Stratified ATAC downsampling to ≥30% of cells preserves average Celltype matching; scores should only drop below that threshold, and cell-type-specific removal ≥60% should increase score variability (Supplementary Fig. S2 trend)
- Alignment output embeddings for each method-dataset pair can be saved as .npy files and metric CSVs must contain one Celltype matching, Subcelltype matching, and batch LISI entry per method-dataset combination

## Limitations

- Alignment quality varies widely across differently initialized GANs — despite similar losses, Celltype matching over 40 GANs was 37.1 ± 16.5%, motivating a topology-regularized single-GAN variant that removes the multi-GAN selection loop
- Robustness to ATAC downsampling is limited: stratified Celltype matching drops when only ~30% of original ATAC cells remain, and variability increases at ≥60% cell-type-specific removal
- Peak memory is scTopoGAN's advantage but it is not the fastest method — SCOT has the lowest runtime across Full PBMC, Partial PBMC, and BM datasets

## Evidence

- [abstract] topoAE latent workflow: "We use topological autoencoders (topoAE) to obtain latent representations of each modality separately. A topology-guided Generative Adversarial Network then aligns these latent representations into a"
- [abstract] non-overlapping scope: "Current integration methods rely on overlapping features or cells to link datasets measuring different modalities, limiting their application to experiments where different molecular layers are"
- [readme] core functions: "The main two functions are 1) ```get_TopoAE_Embeddings``` which obtains the low-dimensional manifold of each dataset separately using Topological Autoencoder, and 2) ```run_scTopoGAN```"
- [methods] benchmark scripts: "The ```UnionCom.py``` and ```manifold_align_mmd_pytorch.py``` scripts are used to apply UnionCom and MMD-MA, respectively. Both methods were used to benchmark the performance of scTopoGAN."
- [methods] topoAE vs alternatives: "The ```TopoAE_benchmark.py``` script compares the performance of the Topological Autoencoder with regular Autoencoder, Variational Autoencoder and UMAP"
