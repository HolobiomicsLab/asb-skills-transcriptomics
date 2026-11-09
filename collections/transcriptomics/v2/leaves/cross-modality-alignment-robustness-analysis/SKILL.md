---
name: cross-modality-alignment-robustness-analysis
description: Use when you need to quantify how robust a cross-modality alignment (e.g.
  scATAC-to-scRNA via scTopoGAN) is to cell-number imbalance, by generating downsamples
  from 90% to 10% of cells and measuring alignment quality per fraction.
when_to_use_negative:
- When the goal is to benchmark scTopoGAN's baseline alignment against other methods
  (UnionCom, MMD-MA) on full datasets rather than test downsampling robustness — use
  the UnionCom.py / manifold_align_mmd_pytorch.py scripts instead.
- When both modalities have comparable cell counts and no modality-imbalance scenario
  needs to be evaluated — the robustness curves add no information.
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0102
tools:
- name: scTopoGAN
  role: Topology-guided GAN used via run_scTopoGAN to align each downsampled ATAC
    manifold to the fixed RNA manifold; its alignment quality is the measured quantity
  repo: https://github.com/AkashCiel/scTopoGAN
- name: Topological Autoencoder
  role: Computes the low-dimensional manifold of each modality separately via get_TopoAE_Embeddings
    before downsampling and alignment; RNA embedding is kept fixed as target
  repo: https://github.com/AkashCiel/scTopoGAN
- name: Python
  role: Implementation language; scripts and scTopoGAN_Functions.py provide get_TopoAE_Embeddings
    and run_scTopoGAN used in the workflow
  repo: https://github.com/AkashCiel/scTopoGAN
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioadv/vbad171
    title: 'scTopoGAN: unsupervised manifold alignment of single-cell data'
  source_content_hash: sha256:ad9e5f3533b66ff26c673b2125f67ebb7254054e91b24c1b1c73b5e4cda0871c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/AkashCiel/scTopoGAN
  tools:
  - Python
  - Topological Autoencoder
  - scTopoGAN
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# cross-modality-alignment-robustness-analysis

## Summary

Stress-tests an unsupervised single-cell manifold alignment method (scTopoGAN) by downsampling one modality (scATAC-seq) relative to a fixed target modality (scRNA-seq), in stratified and cell-type-specific regimes, and tracking the Celltype matching score as cells are removed.

## When to use

Use when you need to quantify how robust a cross-modality alignment (e.g. scATAC-to-scRNA via scTopoGAN) is to cell-number imbalance, by generating downsamples from 90% to 10% of cells and measuring alignment quality per fraction.

## When NOT to use

- When the goal is to benchmark scTopoGAN's baseline alignment against other methods (UnionCom, MMD-MA) on full datasets rather than test downsampling robustness — use the UnionCom.py / manifold_align_mmd_pytorch.py scripts instead.
- When both modalities have comparable cell counts and no modality-imbalance scenario needs to be evaluated — the robustness curves add no information.

## Inputs

- Pre-processed scRNA-seq and scATAC-seq matrices (provided in the package's Data folder, e.g. Full PBMC data)
- Cell-type labels for ATAC cells (to define stratified vs cell-type-specific downsampling strata, e.g. Monocytes, CD4 T cells, CD8 T cells)

## Outputs

- celltype_matching_scores.csv containing averaged Celltype matching score curves for every removal fraction (90%-10%) in both the stratified and cell-type-specific regimes (tabular form of Supplementary Fig. S2)

## How to apply

First compute the low-dimensional manifold of each modality separately with the Topological Autoencoder via get_TopoAE_Embeddings, keeping the RNA embedding fixed as the target. Generate ATAC downsamples by repeated random cell removal from 90% down to 10% of cells — either stratified across all cell types, or separately for specific cell types (e.g. Monocytes, CD4 T cells, CD8 T cells) in the cell-type-specific regime. For every downsample, align the source ATAC manifold to the fixed RNA manifold with run_scTopoGAN (the topology-guided GAN). Compute the Celltype matching score per alignment run and average across replicates per removal fraction and per cell type, then write the averaged score curves for both regimes to celltype_matching_scores.csv. Interpret robustness from the curves: scTopoGAN's alignment is robust under stratified downsampling until only ~30% of the original ATAC cells remain, after which the average Celltype matching score drops, and score variability increases at >=60% cell-type-specific removal.

## Related tools

- **scTopoGAN** (Topology-guided GAN used via run_scTopoGAN to align each downsampled ATAC manifold to the fixed RNA manifold; its alignment quality is the measured quantity) — https://github.com/AkashCiel/scTopoGAN
- **Topological Autoencoder** (Computes the low-dimensional manifold of each modality separately via get_TopoAE_Embeddings before downsampling and alignment; RNA embedding is kept fixed as target) — https://github.com/AkashCiel/scTopoGAN
- **Python** (Implementation language; scripts and scTopoGAN_Functions.py provide get_TopoAE_Embeddings and run_scTopoGAN used in the workflow) — https://github.com/AkashCiel/scTopoGAN

## Examples

```
from scTopoGAN_Functions import get_TopoAE_Embeddings, run_scTopoGAN; atac_emb = get_TopoAE_Embeddings(atac_data); aligned = run_scTopoGAN(source=atac_emb, target=rna_emb_fixed)
```

## Evaluation signals

- celltype_matching_scores.csv contains score curves for all fractions (90% to 10%) of both the stratified and cell-type-specific regimes
- Stratified-regime curve shows robust (stable) average Celltype matching scores until ~30% of original ATAC cells remain, then a drop — matching Supplementary Fig. S2
- Cell-type-specific regime shows increased score variability at >=60% removal for Monocytes, CD4 T cells, and CD8 T cells
- Scores are averaged across replicates per removal fraction and per cell type, not single-run values
- RNA embedding remains fixed as the target manifold across all alignment runs; only the ATAC source varies

## Limitations

- Robustness holds only until ~30% of original ATAC cells remain under stratified downsampling; below that the average Celltype matching score drops
- At >=60% cell-type-specific removal, alignment scores become more variable, so single-replicate results at high removal fractions are unreliable
- No changelog found in the scTopoGAN repository, so version drift in the pipeline cannot be tracked from the repo itself

## Evidence

- [methods] get_TopoAE_Embeddings which obtains the low-dimensional manifold of each dataset separately using Topological Autoencoder: "```get_TopoAE_Embeddings``` which obtains the low-dimensional manifold of each dataset separately using Topological Autoencoder"
- [methods] run_scTopoGAN aligns source to target manifold: "```run_scTopoGAN``` which aligns the source manifold to the target manifold using the topology-guided GAN model"
- [abstract] topoAE obtains latent representations of each modality separately: "We use topological autoencoders (topoAE) to obtain latent representations of each modality separately."
- [readme] README describes the two main functions and example script: "The main two functions are 1) ```get_TopoAE_Embeddings``` which obtains the low-dimensional manifold of each dataset separately using Topological Autoencoder, and 2) ```run_scTopoGAN``` which aligns"
- [methods] pre-processed data provided in Data folder as input: "this script takes as input the pre-processed data provided in the ```Data``` folder"
