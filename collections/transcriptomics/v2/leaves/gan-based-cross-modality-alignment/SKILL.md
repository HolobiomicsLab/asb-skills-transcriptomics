---
name: gan-based-cross-modality-alignment
description: Use when integrating two single-modality single-cell datasets (e.g. scRNA-seq
  and scATAC-seq) profiled on different cells with no shared features, and an unsupervised
  joint embedding is needed.
when_to_use_negative:
- The modalities share overlapping features or cells, so standard integration methods
  that rely on overlaps can be applied instead
- The task is only to obtain a low-dimensional representation of a single modality
  (use the topoAE/UMAP benchmark step rather than the GAN alignment)
- Inputs are raw unprocessed count matrices; the scripts expect pre-processed data
  from the Data folder
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scTopoGAN
  role: 'Core method: topology-guided GAN that aligns the topoAE latent manifolds
    of source and target modalities into a common space (run_scTopoGAN)'
  repo: https://github.com/AkashCiel/scTopoGAN
- name: Topological Autoencoder (topoAE)
  role: Obtains the low-dimensional manifold of each dataset separately before GAN
    alignment (get_TopoAE_Embeddings)
  repo: https://github.com/AkashCiel/scTopoGAN
- name: UnionCom
  role: Unsupervised alignment baseline used to benchmark scTopoGAN (UnionCom.py)
  repo: https://github.com/AkashCiel/scTopoGAN
- name: MMD-MA
  role: Unsupervised manifold alignment baseline used to benchmark scTopoGAN (manifold_align_mmd_pytorch.py)
  repo: https://github.com/AkashCiel/scTopoGAN
- name: SCOTv2
  role: Alignment baseline applied with the hyperparameter settings from Methods 2.7
    in the benchmark comparison
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioadv/vbad171
    title: 'scTopoGAN: unsupervised manifold alignment of single-cell data'
  source_content_hash: sha256:e641ad0049e36d598af3d340c6418e70b1ff1a27d88727af8f57568a4edff2d3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/AkashCiel/scTopoGAN
  tools:
  - MMD-MA
  - SCOTv2
  - Topological Autoencoder (topoAE)
  - UnionCom
  - scTopoGAN
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# gan-based-cross-modality-alignment

## Summary

Align single-cell datasets from different molecular modalities that share no overlapping cells or features by first embedding each modality with a topological autoencoder (topoAE) and then aligning the latent manifolds with a topology-guided GAN (scTopoGAN).

## When to use

Use when integrating two single-modality single-cell datasets (e.g. scRNA-seq and scATAC-seq) profiled on different cells with no shared features, and an unsupervised joint embedding is needed.

## When NOT to use

- The modalities share overlapping features or cells, so standard integration methods that rely on overlaps can be applied instead
- The task is only to obtain a low-dimensional representation of a single modality (use the topoAE/UMAP benchmark step rather than the GAN alignment)
- Inputs are raw unprocessed count matrices; the scripts expect pre-processed data from the Data folder

## Inputs

- Pre-processed single-cell modality matrices (source and target, no overlapping cells or features) from the Data folder, e.g. Full PBMC, Partial PBMC, BM, scGEM datasets

## Outputs

- Aligned joint embedding of the source and target modalities in a common low-dimensional space (savable as .npy files)
- Integration metrics per method-dataset pair: Celltype matching, Subcelltype matching, and batch LISI scores

## How to apply

Load the pre-processed modalities (e.g. Full PBMC, Partial PBMC, BM, scGEM) from the Data folder and shuffle inputs per the benchmark protocol. Compute the low-dimensional manifold of each modality separately with Topological Autoencoder via get_TopoAE_Embeddings, since topoAE preserves the original data structure better than regular Autoencoder, Variational Autoencoder, or UMAP. Call run_scTopoGAN to align the source manifold to the target manifold with the topology-guided GAN, selecting from 20 candidate GAN models by topological loss and refining the best model for 1000 epochs. Benchmark against UnionCom (UnionCom.py), MMD-MA (manifold_align_mmd_pytorch.py), and SCOTv2, then evaluate Celltype matching, Subcelltype matching, and batch LISI (optimal value 2) on the aligned embeddings.

## Related tools

- **scTopoGAN** (Core method: topology-guided GAN that aligns the topoAE latent manifolds of source and target modalities into a common space (run_scTopoGAN)) — https://github.com/AkashCiel/scTopoGAN
- **Topological Autoencoder (topoAE)** (Obtains the low-dimensional manifold of each dataset separately before GAN alignment (get_TopoAE_Embeddings)) — https://github.com/AkashCiel/scTopoGAN
- **UnionCom** (Unsupervised alignment baseline used to benchmark scTopoGAN (UnionCom.py)) — https://github.com/AkashCiel/scTopoGAN
- **MMD-MA** (Unsupervised manifold alignment baseline used to benchmark scTopoGAN (manifold_align_mmd_pytorch.py)) — https://github.com/AkashCiel/scTopoGAN
- **SCOTv2** (Alignment baseline applied with the hyperparameter settings from Methods 2.7 in the benchmark comparison)

## Examples

```
python Run_scTopoGAN.py  # example script for running scTopoGAN on new data, after get_TopoAE_Embeddings on each modality
```

## Evaluation signals

- batch LISI on the aligned joint embedding approaches the optimal value of 2, and exceeds UnionCom, MMD-MA, and SCOT baselines on Full PBMC, Partial PBMC, and BM datasets
- Celltype matching and Subcelltype matching scores on the aligned embeddings reproduce the Table 2 values reported in the scTopoGAN preprint
- Result CSV contains exactly one Celltype matching, Subcelltype matching, and batch LISI entry per method-dataset combination
- topoAE embeddings preserve the original data structure better than regular Autoencoder, Variational Autoencoder, and UMAP in the TopoAE_benchmark comparison

## Limitations

- Designed for complete unsupervised settings; current integration methods that rely on overlapping features or cells are outside this skill's scope and vice versa
- No changelog found in the repository, so version drift between the preprint and current code is not documented
- GAN model selection requires evaluating 20 candidate models by topological loss plus 1000 refinement epochs, which is compute-intensive

## Evidence

- [abstract] topoAE latent manifold alignment: "We use topological autoencoders (topoAE) to obtain latent representations of each modality separately."
- [abstract] topology-guided GAN alignment: "A topology-guided Generative Adversarial Network then aligns these latent representations into a common space."
- [abstract] unsupervised setting with non-overlapping cells/features: "We present scTopoGAN, a method for unsupervised manifold alignment of single-cell datasets with non-overlapping cells or features."
- [readme] two core functions: "```run_scTopoGAN``` which aligns the source manifold to the target manifold using the topology-guided GAN model."
- [readme] baseline benchmark scripts: "The ```UnionCom.py``` and ```manifold_align_mmd_pytorch.py``` scripts are used to apply UnionCom and MMD-MA, respectively."
- [results] outperforms state-of-the-art: "We show that scTopoGAN outperforms state-of-the-art manifold alignment methods in complete unsupervised settings."
