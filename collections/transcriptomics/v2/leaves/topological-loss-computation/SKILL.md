---
name: topological-loss-computation
description: 'Use when training or evaluating scTopoGAN: you need to quantify how
  well the topology of the source ATAC manifold is preserved after projection into
  the RNA target space, at training checkpoints (epochs 500–1000).'
when_to_use_negative:
- When the two modalities share overlapping cells or features — the article notes
  current integration methods that rely on overlapping features or cells, and scTopoGAN/topological
  loss is designed for non-overlapping single-modality datasets
- When evaluating the quality of per-modality latent representations themselves (use
  TopoAE_benchmark.py comparing topoAE with Autoencoder, VAE, or UMAP, not the GAN
  alignment loss)
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: scTopoGAN
  role: Topology-guided GAN whose training objective and diagnostic metric is the
    topological loss between source manifold and projected output
  repo: https://github.com/AkashCiel/scTopoGAN
- name: Topological Autoencoder
  role: Produces the low-dimensional source/target manifolds (get_TopoAE_Embeddings)
    between which the topological loss is computed
  repo: https://github.com/AkashCiel/scTopoGAN
- name: Python
  role: Implementation language; run_scTopoGAN and get_TopoAE_Embeddings are exposed
    in scTopoGAN_Functions.py
  repo: https://github.com/AkashCiel/scTopoGAN
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioadv/vbad171
    title: 'scTopoGAN: unsupervised manifold alignment of single-cell data'
  source_content_hash: sha256:657aa694d469c185015a315a1b9a680c6874a3eb5d2fde2e99d2f307cbbbd90a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/AkashCiel/scTopoGAN
  tools:
  - Python
  - Topological Autoencoder
  - scTopoGAN
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# topological-loss-computation

## Summary

Computes the topological loss (Eq. 2 in scTopoGAN) between the source-modality topoAE manifold and its GAN-projected output in target space, serving both as the topology-preserving training objective and as an alignment-quality diagnostic. In scTopoGAN this loss was negatively correlated with label-matching performance (Pearson r ≈ −0.73 for Celltype, −0.67 for Subcelltype across 40 GAN initializations).

## When to use

Use when training or evaluating scTopoGAN: you need to quantify how well the topology of the source ATAC manifold is preserved after projection into the RNA target space, at training checkpoints (epochs 500–1000).

## When NOT to use

- When the two modalities share overlapping cells or features — the article notes current integration methods that rely on overlapping features or cells, and scTopoGAN/topological loss is designed for non-overlapping single-modality datasets
- When evaluating the quality of per-modality latent representations themselves (use TopoAE_benchmark.py comparing topoAE with Autoencoder, VAE, or UMAP, not the GAN alignment loss)

## Inputs

- Pre-processed PBMC ATAC topoAE latent manifold (source)
- Pre-processed PBMC RNA topoAE latent manifold (target)
- Topology-guided GAN model from run_scTopoGAN at checkpoint epochs (500–1000)
- Celltype and Subcelltype labels on the aligned manifolds (for validation correlations)

## Outputs

- Topological loss values (Eq. 2) per checkpoint epoch per GAN run
- Pearson correlations between topological-loss and Celltype/Subcelltype-matching trajectories, saved to gan_stability_correlations.csv

## How to apply

First obtain the per-modality low-dimensional manifolds with get_TopoAE_Embeddings (Topological Autoencoder), e.g. the pre-processed PBMC ATAC (source) and RNA (target) latent spaces. During run_scTopoGAN training (1000 epochs), compute the topological loss between the input source manifold and the projected output at checkpoints from epoch 500 onward, every 100 epochs (500, 600, 700, 800, 900, 1000). At the same checkpoints, compute Celltype and Subcelltype label-matching scores between the aligned source and target manifolds. To validate the loss as a quality signal, correlate the loss trajectory with the matching-score trajectories per run (Pearson); across 40 different random GAN initializations on the Full PBMC dataset, expect correlations near −0.73 (Celltype) and −0.67 (Subcelltype), since low topological loss tends to accompany better manifold alignment.

## Related tools

- **scTopoGAN** (Topology-guided GAN whose training objective and diagnostic metric is the topological loss between source manifold and projected output) — https://github.com/AkashCiel/scTopoGAN
- **Topological Autoencoder** (Produces the low-dimensional source/target manifolds (get_TopoAE_Embeddings) between which the topological loss is computed) — https://github.com/AkashCiel/scTopoGAN
- **Python** (Implementation language; run_scTopoGAN and get_TopoAE_Embeddings are exposed in scTopoGAN_Functions.py) — https://github.com/AkashCiel/scTopoGAN

## Examples

```
from scTopoGAN_Functions import get_TopoAE_Embeddings, run_scTopoGAN; Zs, Zt = get_TopoAE_Embeddings(atac, rna); Zs_aligned = run_scTopoGAN(Zs, Zt)
```

## Evaluation signals

- Across 40 GAN runs on Full PBMC, Pearson correlation between topological loss and Celltype matching trajectories should fall near −0.73
- Pearson correlation between topological loss and Subcelltype matching trajectories should fall near −0.67
- Loss values are recorded at epochs 500, 600, 700, 800, 900, 1000 (every 100 epochs from epoch 500) for each of the 1000-epoch runs
- Correlation outputs are written to gan_stability_correlations.csv and reproduce the expected negative relationship
- Source manifold must be the topoAE embedding from get_TopoAE_Embeddings, not raw or AE/VAE/UMAP projections

## Limitations

- The negative loss–matching relationship is an empirical observation from 40 identically-architected GANs on the Full PBMC dataset; it is a stability/diagnostic signal, not a guarantee for other datasets or architectures
- The repository provides Run_scTopoGAN.py as an example for new data, but no changelog is present, so version drift in the loss implementation cannot be tracked
- Topological loss measures source-manifold preservation after projection; it does not by itself verify correct celltype correspondence without the label-matching validation step

## Evidence

- [methods] run_scTopoGAN which aligns the source manifold to the target manifold using the topology-guided GAN model: "2) ```run_scTopoGAN``` which aligns the source manifold to the target manifold using the topology-guided GAN model."
- [abstract] topoAE latent representations per modality: "We use topological autoencoders (topoAE) to obtain latent representations of each modality separately."
- [abstract] topology-guided GAN aligns latents into common space: "A topology-guided Generative Adversarial Network then aligns these latent representations into a common space."
- [readme] README on main functions: "The main two functions are 1) ```get_TopoAE_Embeddings``` which obtains the low-dimensional manifold of each dataset separately using Topological Autoencoder"
