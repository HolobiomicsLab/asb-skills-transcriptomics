---
name: generative-adversarial-network-training
description: Use when aligning two non-overlapping single-cell modalities (e.g. Full
  PBMC source/target manifolds) whose topoAE embeddings must be mapped into a common
  space, especially when GAN-to-GAN variability across initializations is too high.
when_to_use_negative:
- Datasets have overlapping features or cells that standard integration methods can
  exploit directly — the article notes current integration methods rely on overlap,
  but this GAN approach is designed for non-overlapping modalities; if overlap exists,
  simpler methods may suffice
- The task is only low-dimensional embedding of a single modality, not cross-modality
  manifold alignment — use the TopoAE step alone (e.g. TopoAE_benchmark.py) instead
  of the GAN
edam_operation: http://edamontology.org/operation_0334
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: scTopoGAN
  role: Topology-guided GAN that aligns the source manifold to the target manifold;
    its training objective is augmented with the topological loss as regularization
  repo: https://github.com/AkashCiel/scTopoGAN
- name: Topological Autoencoder
  role: Obtains the low-dimensional manifold of each dataset separately via get_TopoAE_Embeddings
    before GAN training; its topological loss is reused as the regularization term
  repo: https://github.com/AkashCiel/scTopoGAN
- name: Python
  role: Implementation language; scTopoGAN_Functions.py contains get_TopoAE_Embeddings
    and run_scTopoGAN
  repo: https://github.com/AkashCiel/scTopoGAN
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioadv/vbad171
    title: 'scTopoGAN: unsupervised manifold alignment of single-cell data'
  source_content_hash: sha256:1485e2e9c22c0910db93d89327bd14f1c5bc0b2813401ff3c60e27c3a8811e25
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/AkashCiel/scTopoGAN
  tools:
  - Python
  - Topological Autoencoder
  - scTopoGAN
  edam_operation: http://edamontology.org/operation_0334
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# generative-adversarial-network-training

## Summary

Trains the topology-guided GAN of scTopoGAN to align source and target single-cell manifolds, and extends it by augmenting the GAN's overall loss with the topological loss as a regularization term so the unstable multi-GAN model-selection loop can be removed.

## When to use

Use when aligning two non-overlapping single-cell modalities (e.g. Full PBMC source/target manifolds) whose topoAE embeddings must be mapped into a common space, especially when GAN-to-GAN variability across initializations is too high.

## When NOT to use

- Datasets have overlapping features or cells that standard integration methods can exploit directly — the article notes current integration methods rely on overlap, but this GAN approach is designed for non-overlapping modalities; if overlap exists, simpler methods may suffice
- The task is only low-dimensional embedding of a single modality, not cross-modality manifold alignment — use the TopoAE step alone (e.g. TopoAE_benchmark.py) instead of the GAN

## Inputs

- Pre-processed source and target single-cell modality matrices (e.g. Full PBMC dataset from the Data folder)
- TopoAE low-dimensional latent embeddings of each modality (from get_TopoAE_Embeddings)

## Outputs

- Aligned source-to-target manifold in a common latent space
- Celltype and subcelltype label-matching scores and batch LISI metrics per method
- Comparison CSV table of baseline scTopoGAN vs topology-regularized single-GAN metrics

## How to apply

First compute low-dimensional latent representations of each modality separately with the Topological Autoencoder via get_TopoAE_Embeddings, using the pre-processed data in the Data folder (e.g. Full PBMC). Then train the topology-guided GAN baseline with run_scTopoGAN, which aligns the source manifold to the target manifold using its model-selection loop over multiple GANs; note the baseline's variability — across 40 differently initialized GANs, celltype matching scores averaged 37.1 ± 16.5% despite similar losses. To improve stability, implement a single-GAN variant by adding the topological loss as a regularization term in the GAN's overall training objective, eliminating the multi-GAN selection loop. Finally, evaluate both variants on label-matching scores (celltype and subcelltype) and batch LISI, and write the baseline-versus-regularized comparison to a CSV table.

## Related tools

- **scTopoGAN** (Topology-guided GAN that aligns the source manifold to the target manifold; its training objective is augmented with the topological loss as regularization) — https://github.com/AkashCiel/scTopoGAN
- **Topological Autoencoder** (Obtains the low-dimensional manifold of each dataset separately via get_TopoAE_Embeddings before GAN training; its topological loss is reused as the regularization term) — https://github.com/AkashCiel/scTopoGAN
- **Python** (Implementation language; scTopoGAN_Functions.py contains get_TopoAE_Embeddings and run_scTopoGAN) — https://github.com/AkashCiel/scTopoGAN

## Examples

```
python Run_scTopoGAN.py
```

## Evaluation signals

- Celltype label-matching score of the topology-regularized single GAN should match or exceed the baseline multi-GAN variability of 37.1 ± 16.5% (mean ± SD over 40 GANs)
- Subcelltype label-matching score computed on the Full PBMC alignment should not degrade relative to the multi-GAN baseline
- Batch LISI on the integrated embedding should be comparable to or better than the baseline selected-GAN result
- Similar total GAN losses across differently initialized runs should correspond to similar alignment quality in the regularized variant (unlike the baseline, where similar losses hid wide score variance)
- Comparison CSV contains metric rows for both baseline and topology-regularized GAN on the same Full PBMC evaluation

## Limitations

- In the baseline workflow, GANs with similar losses can yield very different alignment quality (37.1 ± 16.5% celltype matching over 40 GANs), so single-run results may not be representative without the regularized loss
- No changelog is available in the repository, so version drift cannot be tracked
- The regularized single-GAN variant is an extension to be validated against the multi-GAN baseline; it is not part of the published scTopoGAN method itself

## Evidence

- [results] Alignment quality varies widely across differently initialized GANs: [evidence span withheld — anchor status: not_found]
- [methods] topoAE embeddings: "```get_TopoAE_Embeddings``` which obtains the low-dimensional manifold of each dataset separately using Topological Autoencoder"
- [methods] run_scTopoGAN alignment: "```run_scTopoGAN``` which aligns the source manifold to the target manifold using the topology-guided GAN model"
- [abstract] GAN aligns latent representations: "A topology-guided Generative Adversarial Network then aligns these latent representations into a common space."
- [readme] Run_scTopoGAN example: "The ```Run_scTopoGAN.py``` script provides an example for running scTopoGAN on a new data."
